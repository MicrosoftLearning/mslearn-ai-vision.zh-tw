---
lab:
  title: 讀取影像中的文字
  description: 使用 Azure AI 視覺影像分析服務中的光學字元辨識 (OCR) 來找出和擷取影像中的文字。
---

# 讀取影像中的文字

光學字元辨識 (OCR) 是電腦視覺的子集，處理在圖片及文件中讀取文字。 **Azure AI 視覺**影像分析服務提供讀取文字的 API，您將在此練習中探索該 API。

> **注意**：本練習是以發行前 SDK 軟體為基礎，可能會有所變更。 如有必要，我們會使用特定版本的套件，這可能無法反映最新的可用版本。 您可能會遇到一些非預期行為、警告或錯誤。

雖然本練習是以 Azure 視覺分析 Python SDK 為基礎，您仍可使用多種特定語言 SDK 來開發 AI 聊天，包括：

* [適用於 JavaScript 的 Azure AI 視覺分析](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [適用於 Microsoft .NET 的 Azure AI 視覺分析](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [適用於 Java 的 Azure AI 視覺分析](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

本練習大約需要 **30** 分鐘的時間。

## 佈建 Azure AI 視覺資源

若您的訂閱中還沒有 Azure AI 視覺資源，則必須加以佈建。

> **注意**：在本練習中，您將使用獨立的**電腦視覺**資源。 您也可以直接 (或在 *Azure AI Foundry *專案中) 在 *Azure AI 服務*多服務資源中使用 Azure AI 視覺服務。

1. 在 `https://portal.azure.com` 開啟 [Azure 入口網站](https://portal.azure.com)，並使用您的 Azure 認證登入。 關閉任何顯示的歡迎訊息或秘訣。
1. 選取 [建立資源]****。
1. 在搜尋列中，搜尋 `Computer Vision`，選取 [電腦視覺]****，並使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **區域**：*從下列區域中選擇：**美國東部**、**美國西部**、**法國中部**、**南韓中部**、**北歐**、**東南亞**、**西歐**、或**東亞**\**
    - **名稱**：*電腦視覺資源的有效名稱*
    - **定價層**：免費 F0

    \*Azure AI 視覺 4.0 完整功能目前僅適用於這些區域。

1. 選取必要的核取方塊並建立資源。
1. 等候部署完成，然後檢視部署的詳細資料。
1. 部署資源後，請前往該資源，然後在瀏覽窗格中的 [資源管理]**** 節點下，檢視其 [金鑰和端點]**** 頁面。 在下一個程序中，您需要此頁面中的端點和其中一個金鑰。

## 使用 Azure AI 視覺 SDK 開發文字擷取應用程式

在本練習中，您將完成已部分實作、使用 Azure AI 視覺 SDK 從影像中擷取文字的用戶端應用程式。

### 準備應用程式組態

1. 使用 Azure 入口網站頂部搜尋列右側的 **[\>_]** 按鈕，以在 Azure 入口網站中建立一個新的 Cloud Shell，並選取訂閱中沒有儲存體的 ***PowerShell*** 環境。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

    > **注意**：若入口網站要求您選取儲存體來保存檔案，請選擇 [不需要儲存體帳戶]****，然後選取您正在使用的訂閱，然後按 [套用]****。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 調整 Cloud Shell 窗格的大小，以便您可以看見電腦視覺資源的 [金鑰和端點]**** 頁面。

    > **秘訣**：您可以拖曳上方框線來調整窗格的大小。 您也可以使用最小化和最大化按鈕，在 Cloud Shell 和主要入口網站介面之間切換。

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **秘訣**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，使用下列命令瀏覽至應用程式程式碼檔案：

    ```
   cd mslearn-ai-vision/Labfiles/ocr/python/read-text
   ls -a -l
    ```

    資料夾包含應用程式的應用程式設定和程式碼檔案。 資料夾也包含 **/images** 子資料夾，其中包含一些供應用程式分析的影像檔。

1. 執行下列命令安裝 Azure AI 視覺 SDK 套件和其他必要套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. 輸入下列命令，編輯應用程式的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中，更新其包含的設定值，以反映**端點**和電腦視覺資源的驗證**金鑰** (從 Azure 入口網站中的 [金鑰和端點]**** 頁面複製)。
1. 取代預留位置後，使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

### 新增程式碼以從影像讀取文字

1. 在 Cloud Shell 命令列中，輸入下列命令以開啟用戶端應用程式的程式碼檔案：

    ```
   code read-text.py
    ```

    > **秘訣**：建議您最大化 Cloud Shell 窗格，並在命令列主控台與程式碼編輯器之間移動分割列，以便更輕鬆地查看程式碼。

1. 在程式碼檔案中，尋找 **Import namespaces** 註解，並在註解下方新增下列程式碼，以匯入您使用 Azure AI 視覺 SDK 所需的命名空間：

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. 在 **Main** 函式中，已提供可載入組態設定並決定待分析檔案的程式碼。 然後尋找 **Authenticate Azure AI Vision client** 註解，並新增下列特定語言程式碼，以建立和驗證 Azure AI 視覺影像分析用戶端物件：

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. 在 **Main** 函式中，在您剛才新增的程式碼下，尋找 **Read text in image** 註解，並新增下列程式碼，以使用影像分析用戶端讀取影像中的文字：

    ```python
   # Read text in image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print (f"\nReading text in {image_file}")

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ])
    ```

1. 尋找 **Print the text** 註解並新增下列程式碼 (包括最終註解) 以列印找到的文字行，並呼叫函式以在影像中為這些文字行加上標註 (使用每行文字傳回的 **bounding_polygon**)：

    ```python
   # Print the text
   if result.read is not None:
        print("\nText:")
    
        for line in result.read.blocks[0].lines:
            print(f" {line.text}")        
        # Annotate the text in the image
        annotate_lines(image_file, result.read)

        # Find individual words in each line
        
    ```

1. 儲存您的變更 (*CTRL+S*)，但請維持開啟程式碼編輯器，以防您需要修正任何拼字錯誤。

1. 調整窗格大小，以便看到更多主控台，然後輸入下列命令以執行程式：

    ```
   python read-text.py images/Lincoln.jpg
    ```

1. 程式讀取指定影像檔 (*images/Lincoln.jpg*) 中的文字，如下所示：

    ![亞伯拉罕·林肯雕像的相片。](../media/Lincoln.jpg)

1. 在 **read-text** 資料夾中，已建立 **lines.jpg** 影像。 使用 (特定 Azure Cloud Shell) **download** 命令以下載：

    ```
   download lines.jpg
    ```

    下載命令就會在瀏覽器右下方建立快顯視窗連結，您可以選取連結，即可下載並開啟檔案。 影像看起來應該會像這樣：

    ![醒目提示文字的影像。](../media/text.jpg)

1. 再次執行程式，這次指定參數 *images/Business-card.jpg* 以從下列影像中擷取文字：

    ![掃描的名片影像。](../media/Business-card.jpg)

    ```
   python read-text.py images/Business-card.jpg
    ```

1. 下載並檢視產生的 **lines.jpg** 檔案：

    ```
   download lines.jpg
    ```

1. 再執行程式一次，這次指定參數 *images/Note.jpg* 以從此影像中擷取文字：

    ![手寫購物清單的相片。](../media/Note.jpg)

    ```
   python read-text.py images/Note.jpg
    ```

1. 下載並檢視產生的 **lines.jpg** 檔案：

    ```
   download lines.jpg
    ```

### 新增程式碼以傳回個別單字的位置

1. 調整窗格大小，以便您可以看到更多程式碼檔案。 尋找 **Find individual words in each line** 註解，然後新增下列程式碼 (請務必維持正確的縮排層級)：

    ```python
   # Find individual words in each line
   print ("\nIndividual words:")
   for line in result.read.blocks[0].lines:
        for word in line.words:
            print(f"  {word.text} (Confidence: {word.confidence:.2f}%)")
   # Annotate the words in the image
   annotate_words(image_file, result.read)
    ```

1. 儲存變更 (*Ctrl+S*)。 然後，在命令列窗格中，重新執行程式，以從 *images/Lincoln.jpg* 中擷取文字。
1. 觀察輸出，其中應包括影像中的每個單一單字，以及與其預測相關聯的信賴度。
1. 在 **read-text** 資料夾中，已建立 **words.jpg** 影像。 使用 (特定 Azure Cloud Shell) **download** 命令來下載並檢視：

    ```
   download words.jpg
    ```

1. 重新執行 *images/Business-card.jpg* 和 *images/Note.jpg* 的程式，檢視為每個影像產生的 **words.jpg** 檔案。

## 清除資源

若您已完成探索 Azure AI 視覺，您應該刪除在本練習中建立的資源，以避免產生不必要的 Azure 成本：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

1. 在頂端搜尋列中，搜尋 [電腦視覺]**，然後選取您在此實驗室中建立的電腦視覺資源。

1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。
