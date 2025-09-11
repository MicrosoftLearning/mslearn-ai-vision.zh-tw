---
lab:
  title: 偵測影像中的物件
  description: 使用 Azure AI 自訂視覺服務訓練物件偵測模型。
---

# 偵測影像中的物件

**Azure AI 自訂視覺**服務可讓您建立以自己的影像定型的電腦視覺模型。 您可用來定型*影像分類*和*物件偵測*模型；然後可從應用程式進行發佈和取用。

在此練習中，您將使用自訂視覺服務來定型「物件偵測」** 模型，以偵測並找出影像中的三種水果 (蘋果、香蕉和柳橙)。

雖然本練習是以 Azure 自訂視覺 Python SDK 為基礎，您仍可使用多種特定語言 SDK 來開發 AI 聊天，包括：

* [適用於 JavaScript 的 Azure 自訂視覺 (訓練)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training)
* [適用於 JavaScript 的 Azure 自訂視覺 (預測)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction)
* [適用於 Microsoft .NET 的 Azure 自訂視覺 (訓練)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
* [適用於 .NET 的 Azure 自訂視覺 Microsoft (預測)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)
* [適用於 Java 的 Azure 自訂視覺 (訓練)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-training/1.1.0-preview.2/jar)
* [適用於 Java 的 Azure 自訂視覺 (預測)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-prediction/1.1.0-preview.2/jar)

此練習大約需要 **45 分鐘**。

## 建立自訂視覺資源

在定型模型之前，您需要 Azure 資源進行*定型*和*預測*。 您可以為每項工作建立**自訂視覺**資源，也可以建立單一資源並同時用於兩者。 在本練習中，您將建立用於訓練和預測的**自訂視覺**資源。

1. 在 `https://portal.azure.com` 開啟 [Azure 入口網站](https://portal.azure.com)，並使用您的 Azure 認證登入。 關閉任何顯示的歡迎訊息或秘訣。
1. 選取 [建立資源]****。
1. 在搜尋列中，搜尋 `Custom Vision`，選取 [自訂視覺]****，並使用下列設定建立資源：
    - **建立選項**：兩者
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **區域**：*選擇任何可用的區域*
    - **名稱**：*自訂視覺資源的有效名稱*
    - **定型定價層**：F0
    - **預測定價層**：F0

1. 建立資源並等候部署完成，然後檢視部署詳細資料。 您會發現有兩個自訂視覺資源已佈建，一個用於訓練，另一個用於預測。

    > **注意**：每個資源都有自己的*端點*和*金鑰*，可用來管理來自程式碼的存取。 若要定型影像分類模型，您的程式碼必須使用*定型*資源 (搭配其端點和金鑰)；而若要使用定型的模型來預測影像類別，您的程式碼必須使用*預測*資源 (搭配其端點和金鑰)。

1. 部署資源之後，請前往資源群組檢視。 您應該會看到兩個自訂視覺資源，其中一個有著尾碼 ***-Prediction***。

## 在自訂視覺入口網站中建立自訂視覺專案

若要訓練物件偵測模型，您必須根據訓練資源來建立自訂視覺專案。 為此，您將使用自訂視覺入口網站。

1. 開啟新的瀏覽器索引標籤 (將 Azure 入口網站索引標籤保持開啟，您會在稍後返回)。
1. 在瀏覽器索引標籤中，開啟位於 `https://customvision.ai` 的[自訂視覺入口網站](https://customvision.ai)。 若出現提示，請使用您的 Azure 認證登入，並同意服務條款。
1. 建立包含下列設定的新專案：
    - **名稱**：`Detect Fruit`
    - **描述**：`Object detection for fruit.`
    - **資源**：*您的自訂視覺資源*
    - **專案類型**：物件偵測
    - **網域**：一般
1. 等候專案建立並在瀏覽器中開啟。

## 上傳並標記影像

現在您已經有了物件偵測專案，您可以上傳和標記影像以訓練模型。

### 在自訂視覺入口網站中上傳和標記影像

自訂視覺入口網站包含視覺效果工具，可用於上傳包含多個物件類型的影像和標記區域。

1. 在新的瀏覽器索引標籤中，從 `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/object-detection/training-images.zip` 下載[訓練影像](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/object-detection/training-images.zip)，並擷取 zip 資料夾以檢視其內容。 此資料夾包含水果的影像。
1. 在自訂視覺入口網站，您的物件偵測專案中，選取 [新增影像] **** 並上傳已擷取資料夾中的影像。
1. 影像上傳完成後，選取第一個加以開啟。
1. 將滑鼠游標暫留在影像中的任一物件上，直到如下影像顯示自動偵測的區域。 然後選取物件，如有必要，請調整區域大小以將其環繞。

    ![螢幕擷取畫面顯示物件的預設區域。](../media/object-region.jpg)

    或者，您可以簡單地在物件周圍拖曳以建立區域。

1. 當區域環繞物件時，新增具有適當物件類型 (「蘋果」**、「香蕉」** 或「柳橙」**) 的新標籤，如下所示：

    ![螢幕擷取畫面顯示影像中已標記的物件。](../media/object-tag.jpg)

1. 選取並標記影像中的每個其他物件，調整區域大小並視需要新增標籤。

    ![螢幕擷取畫面顯示影像中兩個已標記物件。](../media/object-tags.jpg)

1. 使用右側的 **>** 連結移至下一個影像，並標記其物件。 然後，只需繼續處理整個影像集合並標記每個蘋果、香蕉與柳橙即可。

1. 當您完成最後影像標記時，請先關閉 [ **影像細節** 資料] 編輯器。 請在 [**訓練影像]** 頁面上的 [標記 **] 下方**，選取 **[** 已標記]，即可查看所有已標記的影像：

![專案中已標記影像的螢幕擷取畫面。](../media/tagged-images.jpg)

### 使用自訂視覺 SDK 上傳影像

您可以使用自訂視覺入口網站中的 UI 標記影像，但許多 AI 開發小組會使用其他工具，以便產生檔案，其中包含影像中的標記、物件區域等相關資訊。 在這類案例中，您可以使用自訂視覺定型 API 將已標記的影像上傳至專案。

1. 按一下自訂視覺入口網站中 [定型影像]**** 頁面右上方的 [設定]** (&#9881;) 圖示，以檢視專案設定。
1. 在左側的 [一般]**** 之下，記下可唯一識別此專案的 [專案識別碼]****。
1. 在右側的 [資源]**** 下方，您會發現**金鑰**和**端點**已顯示。 這些是*定型*資源的詳細資料 (您也可以在 Azure 入口網站中檢視資源以取得這項資訊)。
1. 返回包含 Azure 入口網站的瀏覽器索引標籤 (將自訂視覺入口網站索引標籤保持開啟，您會在稍後返回)。
1. 使用 Azure 入口網站頂部搜尋列右側的 **[\>_]** 按鈕，以在 Azure 入口網站中建立一個新的 Cloud Shell，並選取訂閱中沒有儲存體的 ***PowerShell*** 環境。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

    > **注意**：若入口網站要求您選取儲存體來保存檔案，請選擇 [不需要儲存體帳戶]****，然後選取您正在使用的訂閱，然後按 [套用]****。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 調整 Cloud Shell 窗格大小以查看更多內容。

    > **秘訣**：您可以拖曳上方框線來調整窗格的大小。 您也可以使用最小化和最大化按鈕，在 Cloud Shell 和主要入口網站介面之間切換。

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，使用下列命令瀏覽至應用程式程式碼檔案：

    ```
   cd mslearn-ai-vision/Labfiles/object-detection/python/train-detector
   ls -a -l
    ```

    資料夾包含應用程式的應用程式設定和程式碼檔案。 資料夾也包含 **tagged-images.json** 檔案，其中包含多個影像中物件的週框方塊座標，以及包含影像的 **/images** 子資料夾。

1. 執行下列命令，安裝 Azure AI 自訂視覺 SDK 套件以進行訓練，並安裝任何其他必要套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. 輸入下列命令，編輯應用程式的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中，更新其中包含的設定值，以反映自訂視覺*訓練*資源的**端點**和驗證**金鑰**，以及您先前建立的自訂視覺專案的**專案識別碼**。
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。
1. 在 Cloud Shell 命令列中，輸入下列命令開啟 **tagged-images.json** 檔案，以查看 **/images **子資料夾中影像檔的標記資訊：

    ```
   code tagged-images.json
    ```
    
     JSON 會定義一份影像清單，每個影像都包含一或多個已標記的區域。 每個已標記的區域都包含標記名稱，以及含有已標記物件的週框方塊的頂端和左座標及寬度和高度維度。

    > **注意**：此檔案中的座標和維度表示影像上的相對點。 例如，「高度」** 值為 0.7 表示影像高度 70% 的方塊。 有些標記工具會產生其他格式的檔案，其中座標和維度值代表像素、英吋或其他測量單位。

1. 不儲存任何變更並關閉 JSON 檔案 (*CTRL_Q*)。

1. 在 Cloud Shell 命令列中，輸入下列命令以開啟用戶端應用程式的程式碼檔案：

    ```
   code add-tagged-images.py
    ```

1. 您會發現程式碼檔案中的下列詳細資料：
    - 已匯入 Azure AI 自訂視覺 SDK 的命名空間。
    - **Main** 函式會擷取組態設定，並使用金鑰和端點來建立已驗證的 **CustomVisionTrainingClient**，然後搭配專案識別碼使用以建立專案的 **Project** 參考。
    - **Upload_Images**函式會從 JSON 檔案擷取已標記的區域資訊，並用以建立具有區域的影像批次，然後再上傳至專案。

1. 關閉程式碼編輯器 (*CTRL+Q*)，然後輸入下列命令以執行程式：

    ```
   python add-tagged-images.py
    ```

1. 等候程式結束。
1. 切換回包含自訂視覺入口網站的瀏覽器索引標籤 (將 Azure 入口網站 Cloud Shell 索引標籤保持開啟)，並檢視專案的 [訓練影像]**** 頁面 (必要時請重新整理瀏覽器)。
1. 確認已將一些已標記的新影像新增至專案。

## 訓練並測試模型

現在您已經在專案中標記影像，可開始訓練模型。

1. 在自訂視覺專案中，按一下 [訓練]**** (&#9881;<sub>&#9881;</sub>) 以使用已標記的影像來訓練物件偵測模型。 選取 [Quick Training] (快速訓練)**** 選項。
1. 等待訓練完成 (可能需要十分鐘左右的時間)。

    > **秘訣**：Azure Cloud Shell 有 20 分鐘的非使用狀態逾時，超過這段時間便會放棄該工作階段。 在等待訓練完成時，請偶爾返回 Cloud Shell 並輸入一個如 `ls` 的命令，以維持工作階段活躍。

1. 在自訂視覺入口網站中，當訓練完成後，請檢閱*精確度*、*重新叫用*和 *mAP* 效能計量，這些計量會測量物件偵測模型的預測精確度，且各項計量應該都很高。
1. 在頁面右上方，按一下 [快速測試]****，然後在 [影像 URL]**** 方塊中，輸入 `https://aka.ms/test-fruit` 並按一下 [快速測試影像]** (&#10132;) 按鈕。
1. 檢視產生的預測。

    ![測試物件偵測的螢幕擷取畫面](../media/test-object-detection.png)

1. 關閉 [快速測試]**** 視窗。

## 在用戶端應用程式中使用物件偵測器

現在，您已準備好發佈已訓練的模型，並在用戶端應用程式中使用。

### 發佈物件偵測模型

1. 在自訂視覺入口網站的 [效能]**** 頁面上，按一下 [&#128504; 發佈]**** 以發佈具有下列設定的已定型模型：
    - **模型名稱**：`fruit-detector`
    - **預測資源**：*您先前建立的**預測**資源，其結尾是 "-Prediction" (<u>不是</u>定型資源)*。
1. 在 [專案設定] **** 頁面的左上方，按一下*專案資源庫* (&#128065;) 圖示以返回自訂視覺入口網站首頁，現在您的專案已在其中列出。
1. 在自訂視覺入口網站首頁的右上方，按一下設定 ** (&#9881;) 圖示以檢視自訂視覺服務的設定。 然後，在 [資源]**** 之下，尋找以 "-Prediction" 結尾的*預測*資源 (<u>不是</u>定型資源) 來判斷其**金鑰**和**端點**值 (您也可在Azure 入口網站中檢視資源以取得這項資訊)。

## 從用戶端應用程式使用影像分類器

既然您已發佈影像分類模型，即可從用戶端應用程式使用。 同樣地，您可以選擇使用 **C#** 或 **Python**。

1. 返回包含 Azure 入口網站和 Cloud Shell 的瀏覽器索引標籤。
1. 在 Cloud Shell 中，執行下列命令以切換至用戶端應用程式的資料夾，並檢視其包含的檔案：

    ```
   cd ../test-detector
   ls -a -l
    ```

    資料夾包含應用程式的應用程式設定和程式碼檔案。 資料夾也包含下列將用於測試模型的 **produce.jpg** 影像檔。

    ![一些水果的影像。](../media/produce.jpg)

1. 執行下列命令，安裝 Azure AI 自訂視覺 SDK 套件以進行預測，並安裝任何其他必要套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. 輸入下列命令，編輯應用程式的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 更新設定值，以反映自訂視覺*<u>預測</u>* 資源的**端點**和**金鑰**、物件偵測專案的**專案識別碼**，以及已發佈模型的名稱 (應為 *fruit-detector*)。 儲存變更 (*CTRL+S*)，然後關閉程式碼編輯器 (*CTRL+Q*)。

1. 在 Cloud Shell 命令列中，輸入下列命令以開啟用戶端應用程式的程式碼檔案：

    ```
   code test-detector.py
    ```

1. 檢閱程式碼，並注意下列詳細資料：
    - 已匯入 Azure AI 自訂視覺 SDK 的命名空間。
    - **Main** 函式會擷取組態設定，並使用金鑰和端點來建立已驗證的 **CustomVisionPredictionClient**。
    - 預測用戶端物件用於取得 **produce.jpg** 影像的物件偵測預測，並在要求中指定專案識別碼和模型名稱。 預測的標記區域接著會繪製在影像上，結果會儲存為 **output.jpg**。
1. 關閉程式碼編輯器，然後輸入下列命令以執行程式：

    ```
   python test-detector.py
    ```

1. 檢閱程式輸出，其中列出了影像中偵測到的每個物件。
1. 您會發現已產生一個名為 **output.jpg** 的影像檔。 使用 (特定 Azure Cloud Shell) **download** 命令以下載：

    ```
   download output.jpg
    ```

    下載命令就會在瀏覽器右下方建立快顯視窗連結，您可以選取連結，即可下載並開啟檔案。 影像看起來應該會像這樣：

    ![醒目提示偵測到物件的影像。](../media/object-detection-output.jpg )

## 清除資源

如果您不將本實驗中所建立的 Azure 資源用於其他訓練模組，則可以將其刪除以避免產生進一步的費用。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，然後在頂端搜尋列中搜尋您在此實驗中所建立的資源。

1. 在資源頁面上，選取 [刪除]**** 並依照指示刪除該資源。 或者，您也可以刪除整個資源群組以同時清理所有資源。
   
## 其他相關資訊

如需使用自訂視覺服務進行物件偵測的詳細資訊，請參閱[自訂視覺文件](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/)。
