---
lab:
  title: 使用 Azure AI 視覺自訂模型對影像進行分類
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# 使用 Azure AI 視覺自訂模型對影像進行分類

Azure AI 視覺可讓您訓練自訂模型來對具有您所指定標籤的物件進行分類和偵測。 在本實驗中，我們將建置一個自訂影像分類模型來對水果影像進行分類。

## 複製本課程的存放庫

如果您尚未將 **Azure AI 視覺**程式碼存放庫複製到您正在進行本實驗的環境，請遵循下列步驟來執行此動作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。 如果出現提示訊息：「在資料夾中偵測到 Azure 函式專案」**，您可以放心地關閉該訊息。

## 佈建 Azure 資源

如果您的訂用帳戶中還沒有 **Azure AI 服務**資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 在頂端搜尋列中，搜尋 *Azure AI 服務*、選取 [Azure AI 服務]****，並使用下列設定建立 Azure AI 服務多服務帳戶資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*從美國東部、西歐、美國西部 2 中做選擇\**
    - **名稱**：輸入唯一名稱**
    - **定價層**:標準 S0

    \*Azure AI 視覺 4.0 自訂模型標籤目前僅適用於這些區域。

3. 選取必要的核取方塊並建立資源。
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

我們也需要一個儲存體帳戶來儲存訓練影像。

1. 在 Azure 入口網站中，搜尋並選取 [儲存體帳戶]****，然後使用下列設定建立一個新的儲存體帳戶：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇您在 * 中建立 Azure AI 服務資源的相同資源群組
    - **儲存體帳戶名稱**：customclassifySUFFIX 
        - *備註：將 `SUFFIX` 標記取代為您的姓名縮寫或其他值，以確保資源名稱是全域唯一的。*
    - **區域**：*選擇您用於您的 Azure AI Service 資源的相同區域*
    - **主要服務**：Azure Blob 儲存體或 Azure Data Lake Storage Gen2
    - **主要工作負載**：其他
    - **效能**：標準
    - **備援**：[本地備援儲存體 (LRS)]
1. 建立您的儲存體帳戶時，請移至 Visual Studio Code，然後展開 **Labfiles/02-image-classification** 資料夾。
1. 在該資料夾中，選取 **replace.ps1** 並檢閱程式碼。 您將看到它取代了我們在後續步驟中使用的 JSON 檔案 (COCO 檔案) 中的預留位置的儲存體帳戶名稱。 僅將檔案*第一行中*的預留位置取代為您的儲存體帳戶的名稱。 儲存檔案。
1. 以滑鼠右鍵按一下 **02-image-classification** 資料夾，然後開啟整合式終端。 執行下列命令。

    ```powershell
    ./replace.ps1
    ```

1. 您可以檢閱 COCO 檔案以確保您的儲存體帳戶名稱存在。 選取 [training-images/training_labels.json]**** 並檢視前幾個項目。 在 [absolute_url]** 欄位中，您應該會看到類似 *"https://myStorage.blob.core.windows.net/fruit/...* 的項目。如果您沒有看到預期的變更，請確定您只更新 PowerShell 指令碼中的第一個預留位置。
1. 關閉 JSON 和 PowerShell 檔案，然後返回到您的瀏覽器視窗。
1. 您的儲存體帳戶應該已完成。 移至您的儲存體帳戶。
1. 在儲存體帳戶上啟用公用存取。 在左窗格中，瀏覽至 [設定]**** 群組中的 [組態]****，然後啟用 [允許 Blob 匿名存取]**。 選取**儲存**
1. 在左窗格的 [資料儲存體]**** 中，選取 [容器]** ** 並建立一個名為 `fruit` 的新容器，然後將 [匿名存取層級]**** 設定為 [容器 (容器和 Blob 的匿名讀取存取權)]**。

    > **注意**：如果停用 [匿名存取層級]****，請重新整理瀏覽器頁面。

1. 瀏覽至 `fruit`，選取 [上傳]****，然後將 **Labfiles/02-image-classification/training-images** 中的影像 (以及一個 JSON 檔案) 上傳至該容器。

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
