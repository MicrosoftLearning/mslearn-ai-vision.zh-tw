---
lab:
  title: 分析影片
  description: 使用 Azure AI Video Indexer 分析影片。
---

# 分析影片

現今大部分建立和取用的資料都是視訊格式。 **Azure AI Video Indexer** 是 AI 支援的服務，可用來編製影片的索引並從中擷取深入解析。

> **注意**：從 2022 年 6 月 21 日開始，傳回個人識別資訊的 Azure AI 服務功能僅限已獲授與[有限存取權](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客戶使用。 若未獲得有限存取權核准，則無法針對此實驗室使用 Video Indexer 辨識人物和名人。 如需 Microsoft 所做之變更和原因的詳細資訊，請參閱[臉部辨識的負責任 AI 投資和保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (英文)。

## 將影片上傳至 Video Indexer

首先，您必須登入 Video Indexer 入口網站並上傳影片。

1. 在您的瀏覽器中，在 `https://www.videoindexer.ai` 開啟 [Video Indexer 入口網站](https://www.videoindexer.ai)。
1. 如果您有現有 Video Indexer 帳戶，請登入。 否則，註冊免費帳戶，並使用您的 Microsoft 帳戶 (或任何其他有效的帳戶類型) 進行登入。 如果登入時遇到困難，請嘗試開啟私人瀏覽器工作階段。

    > 注意：若這是您第一次登入，可能會看到快顯表單，要求您確認如何使用該服務。 

1. 在新的索引標籤中，造訪 `https://aka.ms/responsible-ai-video` 來下載負責任 AI 影片。 儲存檔案。
1. 在 Video Indexer中，選取 [上傳]**** 選項。 然後選取 [瀏覽檔案]**** 選項、選取下載的影片，再按一下 [新增]****。 將 [檔案名稱]**** 欄位中的文字變更為 [負責任 AI]****。 選取 [檢閱 + 上傳]****、檢閱摘要概觀、選取核取方塊以確認是否符合 Microsoft 的臉部辨識原則，然後上傳檔案。
1. 上傳檔案之後，請稍候幾分鐘，Video Indexer 會自動為其編製索引。

> **注意**：在此練習中，我們會使用此影片來探索 Video Indexer 功能；但是當您完成練習後，您應該花一些時間完整觀看影片，因為其中包含實用資訊和指引，以便負責地開發已啟用 AI 的應用程式！ 

## 檢閱影片深入解析

索引編製程序會從影片中擷取深入解析，您可在入口網站中檢視。

1. 在 Video Indexer 入口網站中，影片編製索引後，請加以選取進行檢視。 您會看到窗格中顯示從影片擷取的深入解析，該窗格旁邊則有影片播放程式。

    > **注意**：由於保護個人身分識別的有限存取原則，因此您在編製影片索引時可能不會看到名稱。

    ![具有影片播放程式的 Video Indexer 和深入解析窗格](../media/video-indexer-insights.png)

1. 視訊播放時，選取 [時間軸]**** 索引標籤以檢視視訊音訊的文字記錄。

    ![具有影片播放程式的 Video Indexer 和顯示影片文字記錄的時間軸窗格。](../media/video-indexer-transcript.png)

1. 在入口網站的右上方，選取 [檢視]**** 符號 (看起來類似 &#128455;)，然後在深入解析清單中，除了 [文字記錄]**** 之外，選取 [OCR]**** 和 [說話者]****。

    ![已選取具有 [文字記錄]、[OCR] 和 [演講者] 的Video Indexer 檢視功能表](../media/video-indexer-view-menu.png)

1. 觀察 [時間軸]**** 窗格現在包含：
    - 音訊旁白的文字記錄。
    - 影片中可見的文字。
    - 影片中出現之說話者的指示。 有些知名人物會自動依名稱辨識，其他人物則是以數字表示 (例如「說話者 #1」**)。
1. 切換回到 [深入解析]**** 窗格，然後檢視該處顯示的深入解析。 其中包含：
    - 影片中出現的個別人物。
    - 影片中討論的主題。
    - 影片中出現之物件的標籤。
    - 具名實體，例如出現在影片中的人物和品牌。
    - 主要場景。
1. 在 [深入解析]**** 窗格顯示的情況下，再次選取 [檢視]**** 符號，然後在深入解析清單中，將 [關鍵字]**** 和 [情感]**** 新增至窗格。

    找到的深入解析可協助您判斷影片中的主要主題。 例如，此影片的**主題**顯示其顯然關於技術、社會責任和道德。

## 搜尋深入解析

您可以使用 Video Indexer 來搜尋影片以取得深入解析。

1. 在 [深入解析]**** 窗格的 [搜尋]**** 方塊，輸入「蜜蜂」**。 您可能需要在 [深入解析] 窗格中向下捲動，才能查看所有類型的深入解析結果。
1. 觀察已找到一個相符的「標籤」**，而其在影片中的位置於下方指出。
1. 選取指出蜜蜂存在其中的區段開頭，並在該時間點檢視影片 (您可能需要暫停影片並仔細選取 -蜜蜂只會短暫出現！)

    ![Bee 的 Video Indexer 搜尋結果](../media/video-indexer-search.png)

1. 清除 [搜尋]**** 方塊以顯示影片的所有深入解析。

## 使用 Video Indexer REST API

Video Indexer 提供 REST API，可供您上傳和管理您帳戶中的影片。

1. 在新的瀏覽器索引標籤中，在 `https://portal.azure.com` 開啟 [Azure 入口網站](https://portal.azure.com)，並使用您的 Azure 認證登入。 保留現有索引標籤，並維持 Video Indexer 入口網站開啟。
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

1. 複製存放庫之後，瀏覽至包含本練習應用程式程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-vision/Labfiles/video-indexer
    ```

### 取得 API 詳細資料

若要使用 Video Indexer API，您需要一些資訊來驗證要求：

1. 在 Video Indexer 入口網站中，展開左窗格並選取 [帳戶設定]**** 頁面。
1. 請注意此頁面上的 [帳戶識別碼]**** - 您稍後會用到。
1. 開啟新的瀏覽器索引標籤，前往位於 https://api-portal.videoindexer.ai 的 [Video Indexer 開發人員入口網站](https://api-portal.videoindexer.ai)，然後使用您的 Azure 認證登入。
1. 在 [設定檔]**** 頁面上，檢視與您的設定檔相關聯的 [訂用帳戶]****。
1. 在具有您訂用帳戶的頁面上，觀察每個訂用帳戶都已獲得兩個金鑰 (主要和次要)。 然後針對任何金鑰選取 [顯示]**** 加以查看。 您很快就需要此金鑰。

### 使用 REST API

現在您已擁有帳戶識別碼和 API 金鑰，即可使用 REST API 來處理帳戶中的影片。 在此程序中，您將使用 PowerShell 指令碼進行 REST 呼叫；但相同的準則適用於 HTTP 公用程式，例如 cURL 或 Postman，或任何能夠透過 HTTP 傳送和接收 JSON 的程式開發語言。

所有與 Video Indexer REST API 的互動都會遵循相同的模式：

- 標頭中具有 API 金鑰之 **AccessToken** 方法的初始要求用於取得存取權杖。
- 後續要求會在呼叫 REST 方法來處理影片時，使用存取權杖進行驗證。

1. 在 Cloud Shell 中，使用下列命令來開啟 PowerShell 指令碼：

    ```
   code get-videos.ps1
    ```
    
1. 在 PowerShell 指令碼中，將 **YOUR_ACCOUNT_ID** 和 **YOUR_API_KEY** 預留位置取代為您先前識別的帳戶識別碼和 API 金鑰值。
1. 觀察免費帳戶的*位置*是「試用版」。 如果您已建立不受限制的 Video Indexer 帳戶 (具有相關聯的 Azure 資源)，即可將此變更為 Azure 資源的佈建位置 (例如「eastus」)。
1. 檢閱指令碼中的程式碼，請注意叫用兩個 REST 方法：一個用以取得存取權杖，另一個用以列出您帳戶中的影片。
1. 儲存變更 (按 *CTRL+S*)、關閉程式碼編輯器 (按 *CTRL+Q*)，然後執行下列命令來執行指令碼：

    ```
   ./get-videos.ps1
    ```
    
1. 檢視來自 REST 服務的 JSON 回應，其中應該包含您先前編製索引之**負責任 AI** 影片的詳細資料。

## 使用 Video Indexer 小工具

Video Indexer 入口網站是管理影片索引專案的實用介面。 不過，您可能想要讓影片及其深入解析可供無法存取您的 Video Indexer 帳戶的人員使用。 Video Indexer 會提供一些小工具，讓您為此目的內嵌於網頁中。

1. 使用 `ls` 命令來檢視 **video-indexer** 資料夾的內容。 您會發現其包含 **analyze-video.html** 檔案。 這是您將新增 Video Indexer [播放程式]**** 和 [深入解析]**** 小工具的基本 HTML 頁面。
1. 輸入下列命令以編輯檔案：

    ```
   code analyze-video.html
    ```

    程式碼編輯器中會開啟檔案。
   
1. 請注意標頭中 **vb.widgets.mediator.js** 指令碼的參考 - 此指令碼可讓頁面上的多個 Video Indexer 小工具彼此互動。
1. 在 Video Indexer 入口網站中，返回 [媒體檔案]**** 頁面，然後開啟您的 [負責任 AI]**** 影片。
1. 在影片播放程式之下，選取 [&lt;/&gt; 內嵌]**** 可檢視 HTML iframe 程式碼以內嵌小工具。
1. 在 [共用和內嵌]**** 對話方塊中，選取 [播放程式]**** 小工具、將視訊大小設定為 560 x 315，然後將內嵌程式碼複製到剪貼簿。
1. 在 Azure 入口網站 Cloud Shell 上，在 **analyze-video.html** 檔案的程式碼編輯器中，將複製的程式碼貼在 **&lt;-- Player widget goes here -- &gt;** 註解下方。
1. 返回 Video Indexer 入口網站，在 [共用和內嵌]**** 對話方塊中，選取 [深入解析]**** 小工具，然後將內嵌程式碼複製到剪貼簿。 接著關閉 [共用和內嵌]**** 對話方塊、切換回到 Azure 入口網站，然後將所複製的程式碼貼在 **&lt;-- Insights widget goes here -- &gt;** 註解下方。
1. 編輯檔案後，在程式碼編輯器中使用 *CTRL+S* 儲存變更，然後使用 *CTRL+Q* 關閉程式碼編輯器，同時維持 Cloud Shell 命令列開啟。
1. 在 Cloud Shell 工具列中，輸入下列 (特定 Cloud Shell) 命令，以下載您編輯的 HTML 檔案：

    ```
    download analyze-video.html
    ```

    下載命令會在瀏覽器右下方建立快顯連結，您可以選取連結以下載並開啟檔案，其看起來應該如下：

    ![網頁中的 Video Indexer 小工具](../media/video-indexer-widgets.png)

1. 透過小工具進行試驗，使用 [深入解析]**** 小工具來搜尋深入解析，並跳到影片中的深入解析。

