---
lab:
  title: 使用 AI 產生影像
  description: 在 Azure AI Foundry 中使用 OpenAI a DALL-E 模型來產生影像。
---

# 使用 AI 產生影像

在本練習中，您將使用 OpenAI DALL-E 生成式 AI 模型來產生影像。 您也可以使用 OpenAI Python SDK 來建立簡單的應用程式，以根據您的提示產生影像。

> **注意**：本練習是以發行前 SDK 軟體為基礎，可能會有所變更。 如有必要，我們會使用特定版本的套件，這可能無法反映最新的可用版本。 您可能會遇到一些非預期行為、警告或錯誤。

雖然本練習是以 OpenAI Python SDK 為基礎，您仍可使用多種特定語言 SDK 來開發 AI 聊天，包括：

* [適用於 Microsoft .NET 的 OpenAI](https://www.nuget.org/packages/OpenAI)
* [適用於 JavaScript 的 OpenAI 專案](https://www.npmjs.com/package/openai)

本練習大約需要 **30** 分鐘的時間。

## 開啟 Azure AI Foundry 入口網站

讓我們從登入 Azure AI Foundry 入口網站開始。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖（若 [說明]**** 窗格已開啟，請將其關閉）：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](./media/ai-foundry-home.png)

1. 檢閱首頁上的資訊。

## 選擇模型以開始專案

Azure AI *專案* 提供 AI 開發的共同作業工作區。 讓我們從選擇想要使用的模型開始，並建立一個專案以運用該模型。

> **注意**：AI Foundry 專案可以以 *Azure AI Foundry* 資源為基礎，提供存取 AI 模型 (包括 Azure OpenAI)、Azure AI 服務和其他資源，以開發 AI 代理程式和聊天解決方案。 或者，專案可以以 *AI 中心*資源為基礎，其中包括與 Azure 資源的連線，以取得安全儲存體、計算和特殊化工具。 Azure AI Foundry 型專案非常適合想要管理 AI 代理程式或聊天應用程式開發資源的開發人員。 AI 中心型專案更適合用於處理複雜 AI 解決方案的企業開發團隊。

1. 在首頁的 [探索模型和功能]**** 區段中搜尋 `dall-e-3` 模型。我們會在專案中使用這個模型。

1. 在搜尋結果中選取 **dall-e-3** 模型以查看其詳細資料，然後在該模型的頁面頂端選取 [使用此模型]****。

1. 當系統提示您建立專案時，請輸入您專案的有效名稱，然後展開 [進階選項]****。

1. 選取**自訂**，然後為您的中樞指定下列設定：
    - **Azure AI Foundry 資源**：*Azure AI Foundry 資源的有效名稱*
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **區域**：*選取任何 **建議的 AI Foundry***\*

    > \*部分 Azure AI 資源受區域模型配額限制。 在練習後期，若超過配額限制，您可能需要在不同區域建立另一個資源。

1. 選取 [建立]****，並等待您的專案建立完畢，這包括您選取的 dall-e-3 模型部署。

    > 注意：視您選取的模型而定，您可能會在專案建立過程中收到其他提示。 同意條款並完成部署。

1. 建立專案時，您的模型將顯示在 [模型 + 端點] **** 頁面中。

## 在遊樂場中測試模型。

在建立用戶端應用程式之前，讓我們在遊樂場中測試 DALL-E 模型。

1. 選取 [遊樂場]****，然後選取 [影像遊樂場]****。

1. 確定已選取您的 DALL-E 模型部署。 然後，在頁面底部附近的方塊中，輸入提示，例如 `Create an image of an robot eating spaghetti`，然後選取 [產生]****。

1. 在遊樂場中檢閱產生的影像：

    ![影像遊樂場的螢幕擷取畫面，其中包含產生的影像。](../media/images-playground.png)

1. 輸入後續提示，例如 `Show the robot in a restaurant` 並檢閱產生的影像。

1. 繼續使用新的提示進行測試，以精簡影像，直到您滿意為止。 

1. 選取 [檢視程式碼]**\</\>** 按鈕，並確定您位於 [Entra ID 驗證]**** 索引標籤上。然後記錄下列資訊以供稍後在練習中使用。 請注意，這些值是範例，請務必記錄部署中的資訊。

    * OpenAI 端點：*https://dall-e-aus-resource.cognitiveservices.azure.com/*
    * OpenAI API 版本：*2024-04-01-preview*
    * 部署名稱 (模型名稱)：*dall-e-3*

## 建立用戶端應用程式

該模型似乎在遊樂場中發揮作用。 現在您可以使用 OpenAI SDK 以在用戶端應用程式中使用。

### 準備應用程式組態

1. 開啟一個新的瀏覽器索引標籤（保持 Azure AI Foundry 入口網站在現有索引標籤中開啟）。 然後在新索引標籤中，瀏覽到 `https://portal.azure.com` 的 [Azure 入口網站](https://portal.azure.com)；如果出現提示，請使用您的 Azure 認證登入。

1. 使用頁面頂部搜尋欄右側的 **[\>_]** 按鈕在 Azure 入口網站中建立一個新的 Cloud Shell，並選擇 ***PowerShell*** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

    > **注意**：若入口網站要求您選取儲存體來保存檔案，請選擇 [不需要儲存體帳戶]****，然後選取您正在使用的訂閱，然後按 [套用]****。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-vision/Labfiles/dalle-client/python
    ```

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity openai requests
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 將 **your_endpoint**、**your_model_deployment** 和 **your_api_version** 佔位元取代為您從**影像遊樂場**所記錄的值。

1. 取代預留位置後，使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

### 撰寫程式碼以連線至您的專案，並與模型聊天

> **提示**：新增程式碼時，請確保保持正確的縮排。

1. 輸入以下命令，編輯已提供的\程式碼檔案：

    ```
   code dalle-client.py
    ```

1. 在程式碼檔案中，記下檔案頂端新增的現有語句，以匯入必要的 SDK 命名空間。 然後，在註解 [新增引用]**** 下，新增以下程式碼來引用之前安裝的庫中的命名空間：

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   from openai import AzureOpenAI
   import requests
    ```

1. 在 **main** 函式中，在 **Get configuration settings** 註解下方，您會發現程式碼會載入您在設定檔中定義的端點、API 版本和模型部署名稱值。

1. 在 **Initialize the client** 註解下方新增以下程式碼，並使用您目前登入的 Azure 認證連線到您的模型：

    ```python
   # Initialize the client
   token_provider = get_bearer_token_provider(
       DefaultAzureCredential(exclude_environment_credential=True,
           exclude_managed_identity_credential=True), 
       "https://cognitiveservices.azure.com/.default"
   )
    
   client = AzureOpenAI(
       api_version=api_version,
       azure_endpoint=endpoint,
       azure_ad_token_provider=token_provider
   )
    ```

1. 請注意，程式碼包含迴圈，可讓使用者輸入提示，直到他們輸入 "quit" 為止。 然後在執行迴圈區段的註解 [產生影像]**** 下，新增下列程式碼以提交提示，並從您的模型擷取所產生影像的 URL：

    **Python**

    ```python
   # Generate an image
   result = client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

   json_response = json.loads(result.model_dump_json())
   image_url = json_response["data"][0]["url"] 
    ```

1. 請注意，**main** 函式其餘部分的程式碼將影像 URL 和檔案名稱傳遞給提供的函式，該函式下載產生的影像並將其儲存為 .png 檔案。

1. 使用 **CTRL+S** 命令儲存對程式碼檔案的變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持雲端 shell 命令列開啟。

### 執行用戶端應用程式

1. 在 Cloud Shell 命令行窗格中，輸入下列命令以登錄 Azure。

    ```
   az login
    ```

    **<font color="red">即使 Cloud Shell 工作階段已經過驗證，您還是必須登錄 Azure。</font>**

    > **注意**：在大部分情況下，只要使用 *az 登入*即可。 不過，如果您在多個租用戶中擁有訂用帳戶，您可能需要使用 *--tenant* 參數指定租用戶。 如需詳細資料，請參閱[使用 Azure CLI 以互動方式登入 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。

1. 出現提示時，請遵循指示，在新的索引標籤中開啟登入頁面，然後輸入所提供的驗證碼和您的 Azure 認證。 接著，在命令行中完成登入流程。如果出現提示，請選取包含 Azure AI Foundry 中樞的訂用帳戶。

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來執行應用程式：

    ```
   python dalle-client.py
    ```

1. 出現提示時，請輸入影像的要求，例如 `Create an image of a robot eating pizza`。 片刻之後，應用程式應確認影像已儲存。

1. 嘗試更多提示。 完成後，輸入 `quit` 可結束程式。

    > **注意**：在這個簡單的應用程式中，我們還沒有實現保留交談記錄的邏輯；因此模型會將每個提示視為一個新要求，而沒有前一個提示的上下文。

1. 若想下載並檢視應用程式所產生的影像，請使用Cloud Shell **下載** 命令，也就是指定產生的.png 檔案：

    ```
   download ./images/image_1.png
    ```

    下載命令就會在瀏覽器右下方建立快顯視窗連結，您可以選取連結，即可下載並開啟檔案。

## 摘要

在本練習中，您使用 Azure AI Foundry 和 Azure OpenAI SDK 建立了一個用戶端應用程式，該應用程式使用 DALL-E 模型來產生影像。

## 清理

如果您已完成 DALL-E 探索，您應該刪除在本練習中建立的資源，以避免產生不必要的 Azure 成本。

1. 返回包含 Azure 入口網站的瀏覽器索引標籤 (或在新的瀏覽器索引標籤中重新開啟位於`https://portal.azure.com`的 [Azure 入口網站](https://portal.azure.com))，並檢視您在其中部署本練習所用資源的資源群組內容。
1. 在工具列上，選取 [刪除資源群組]****。
1. 輸入資源群組名稱並確認您想要將其刪除。
