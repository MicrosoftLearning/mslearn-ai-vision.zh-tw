---
lab:
  title: 使用 Azure AI 視覺自訂模型對影像進行分類
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# 使用 Azure AI 視覺自訂模型對影像進行分類

Azure AI 視覺可讓您訓練自訂模型來對具有您所指定標籤的物件進行分類和偵測。 在本實驗中，我們將建置一個自訂影像分類模型來對水果影像進行分類。

## 佈建電腦視覺資源

若您的訂閱中還沒有**電腦視覺**資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
1. 選取 **[建立資源]**。
1. 在搜尋列中，搜尋 [電腦視覺]**、選取[電腦視覺]****，並使用下列設定建立資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*從美國東部、美國西部、法國中部、南韓中部、北歐、東南亞、西歐或東亞中選擇\**
    - **名稱**：輸入唯一名稱**
    - **定價層**：免費 F0

    \*Azure AI 視覺 4.0 完整功能目前僅適用於這些區域。

1. 選取必要的核取方塊並建立資源。
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

我們也需要一個儲存體帳戶來儲存訓練影像。

1. 在 Azure 入口網站中，搜尋並選取 [儲存體帳戶]****，然後使用下列設定建立一個新的儲存體帳戶：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇您建立自訂視覺資源所使用的相同資源群組*
    - **儲存體帳戶名稱**：customclassifySUFFIX 
        - *備註：將 `SUFFIX` 權杖取代為您的姓名縮寫或其他值，以確保資源名稱是全域唯一的。*
    - **區域**：*選擇您用於您的 Azure AI Service 資源的相同區域*
    - **主要服務**：Azure Blob 儲存體或 Azure Data Lake Storage Gen2
    - **主要工作負載**：其他
    - **效能**：標準
    - **備援**：[本地備援儲存體 (LRS)]

1. 部署資源之後，選取 [前往資源]****。
1. 在儲存體帳戶上啟用公用存取。 在左窗格中，瀏覽至 [設定]**** 群組中的 [組態]****，然後啟用 [允許 Blob 匿名存取]**。 選取**儲存**
1. 在左窗格的 [資料儲存體]**** 中，選取 [容器]** ** 並建立一個名為 `fruit` 的新容器，然後將 [匿名存取層級]**** 設定為 [容器 (容器和 Blob 的匿名讀取存取權)]**。

    > **注意**：如果停用 [匿名存取層級]****，請重新整理瀏覽器頁面。
   
## 複製本課程的存放庫

訓練模型的影像檔已在 GitHub 存放庫中提供。 您將複製存放庫，並使用 Azure 入口網站中的 Cloud Shell 將影像上傳至儲存體帳戶。 

> **秘訣**：若您最近已複製 **mslearn-ai-vision** 存放庫，您可以跳過複製工作。 否則，請遵循下列步驟將存放庫複製到您的開發環境。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. 複製存放庫之後，瀏覽至包含練習檔案的資料夾：  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. 執行命令 `code replace.ps1` 並檢閱程式碼。 您將看到它取代了我們在後續步驟中使用的 JSON 檔案 (COCO 檔案) 中的預留位置的儲存體帳戶名稱。
1. 僅將檔案*第一行中*的預留位置取代為您的儲存體帳戶的名稱。
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或按右鍵 > [儲存]**** 來儲存變更，然後使用 **CTRL+Q** 命令或按右鍵 > [結束]**** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。
1. 使用下列命令來執行指令碼：

    ```powershell
    ./replace.ps1
    ```

1. 您可以檢閱 COCO 檔案以確保您的儲存體帳戶名稱存在。 執行 `code training-images/training_labels.json` 並檢視前幾個項目。 在 [absolute_url]** 欄位中，您應該會看到類似 *"https://myStorage.blob.core.windows.net/fruit/...* 的項目。如果您沒有看到預期的變更，請確定您只更新 PowerShell 指令碼中的第一個預留位置。
1. 關閉程式碼編輯器。
1. 執行下列命令，將 `<your-storage-account>` 取代為您的儲存體帳戶名稱，將 **training-images** 資料夾的內容上傳至您稍早建立的 `fruit` 容器。

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. 開啟 `fruit` 容器，並確認檔案已正確上傳。

## 建立自訂模型訓練專案

接下來，您將在 Vision Studio 中建立一個用於自訂影像分類的新訓練專案。

1. 在網頁瀏覽器中，瀏覽至 `https://portal.vision.cognitive.azure.com/` 並使用您在其中建立 Azure AI 資源的 Microsoft 帳戶登入。
1. 選取 [使用影像自訂模型]**** 磚 (如果未在預設檢視中顯示，則可以在 [影像分析]**** 索引標籤中找到)。
1. 選取您建立的 Azure AI 服務帳戶。
1. 在您的專案中，選取頂端的 [新增資料集]****。 使用下列設定來設定  ：
    - **資料集名稱**：training_images
    - **模型類型**：影像分類
    - **選取 Azure Blob 儲存體容器**：選取 [選取容器]****
        - **訂用帳戶**：您的 Azure 訂用帳戶**
        - **儲存體帳戶**：*您所建立的儲存體帳戶*
        - **Blob 容器**：fruit
    - 選取 [允許 Vision Studio 讀取和寫入您的 Blob 儲存體] 方塊
1. 選取 [training_images]**** 資料集。

此時，在專案建立過程中，您通常會選取 [建立 Azure ML 資料標記專案]**** 並標記您的影像 (這會產生 COCO 檔案)。 如果您有時間，我們鼓勵您嘗試此動作，但基於本實驗的目的，我們已經為您標記了影像並提供了產生的 COCO 檔案。

1. 選取 [新增 COCO 檔案]****
1. 在下拉式清單中，選取 [從 Blob 容器匯入 COCO 檔案]****
1. 由於您已經連接了名為 `fruit` 的容器，因此 Vision Studio 會在其中搜尋 COCO 檔案。 從下拉式清單中選取 [training_labels.json****，然後新增該 COCO 檔案。
1. 瀏覽至左側的 [自訂模型]**** 並選取 [訓練新的模型]****。 使用下列設定：
    - **模型名稱**：classifyfruit
    - **模型類型**：影像分類
    - **選擇訓練資料集**：training_images
    - 將其餘部分保留為預設值，然後選取 [訓練模型]****

訓練可能需要一些時間 (預設預算最多為一個小時)，但是對於這個小型資料集，它通常比這個要快得多。 每隔幾分鐘選取 [重新整理]**** 按鈕，直到作業的狀態為*成功*為止。 選取該模型。

您可以在這裏檢視訓練作業的效能。 檢閱訓練模型的精確度和正確性。

## 測試您的自訂模型

您的模型已經過訓練並準備好進行測試。

1. 在自訂模型的頁面頂端上，選取 [試用]****。
1. 從指定要使用的模型的下拉式清單中選取 [classifyfruit]**** 模型，然後瀏覽至 **02-image-classification\test-images** 資料夾。
1. 選取每個影像並檢視結果。 選取結果方塊中的 [JSON]**** 索引標籤來檢查完整的 JSON 回應。

<!-- Option coding example to run-->
## 清除資源

如果您不將本實驗中所建立的 Azure 資源用於其他訓練模組，則可以將其刪除以避免產生進一步的費用。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，然後在頂端搜尋列中搜尋您在此實驗中所建立的資源。

2. 在資源頁面上，選取 [刪除]**** 並依照指示刪除該資源。 或者，您也可以刪除整個資源群組以同時清理所有資源。
