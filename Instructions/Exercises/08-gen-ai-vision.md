---
lab:
  title: 開發具備視覺功能的聊天應用程式
  description: 瞭解如何使用 Azure AI Foundry 來建置支援影像輸入的產生式 AI 應用程式。
---

# 開發具備視覺功能的聊天應用程式

在此練習中，您會使用 *Phi-4-multimodal-instruct* 產生式 AI 模型，來產生對包含影像之提示的回應。 您將開發一個應用程式，透過使用 Azure AI Foundry 和 Azure AI 模型推斷服務，為雜貨店中的新鮮產品提供 AI 協助。

本練習大約需要 **30** 分鐘的時間。

## 建立 Azure AI Foundry 專案

讓我們從建立 Azure AI Foundry 專案開始。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](../media/ai-foundry-home.png)

2. 在首頁中，選取 **+ 建立專案**。
3. 請在** [建立專案**精靈] 中，輸入專案有效名稱。如果建議使用現有的中樞，請先選擇建立新的中樞選項。 然後審查 Azure 資源，將會自動建立，以便支援中樞和專案。
4. 選取**自訂**，然後為您的中樞指定下列設定：
    - **中樞名稱**：*請提供有效的中樞名稱*
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **區域**：選取下列任何區域\*：
        - 美國東部
        - 美國東部 2
        - 美國中北部
        - 美國中南部
        - 瑞典中部
        - 美國西部
        - 美國西部 3
    - **連接 Azure AI 服務或 Azure OpenAI**：*[建立新的 AI 服務資源]*
    - **連接 Azure AI 搜尋服務**：跳過連接

    > \* 撰寫本文時，我們在此練習中將使用的 Microsoft *Phi-4-multimodal-instruct* 模型已在下列區域推出。 您可以在 [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)中查看特定模型的最新區域可用性。 如果練習稍後達到區域配額限制，您可能需要在不同的區域中建立另一個資源。

5. 選取**下一步**並檢閱您的設定。 然後選取**建立**並等待該流程完成。
6. 建立專案後，關閉顯示的所有提示並檢閱 Azure AI Foundry 入口網站中的專案頁面，該頁面應類似於下圖：

    ![Azure AI Foundry 入口網站中 Azure AI 專案詳細資料的螢幕螢幕擷取畫面。](../media/ai-foundry-project.png)

## 部署多模式模型

現在您已準備好部署可支援影像型輸入的多模式模型。 您可以選擇數個模型，包括 OpenAI *gpt-4o* 模型。 在此練習中，我們將使用 *Phi-4-multimodal-instruct* 模型，以支援包含影像的提示。

1. 在工具列頂部右上方的 Azure AI Foundry 專案頁面中，使用 **預覽功能** (**amp;#9215;**) 圖示，以確保 **部署模型至 Azure AI 模型推斷服務** 功能已啟用。 這項功能可確保模型部署可供 Azure AI 推斷服務使用，您將可在應用程式程式碼中使用這項功能。
2. 在專案左側窗格中的 [我的資產]**** 區段中，選取 [模型 + 端點]**** 頁面。
3. 在 [模型 + 端點]**** 頁面中，於 [模型部署]**** 索引標籤的 [+ 部署模型]**** 功能表中，選取 [部署基本模型]****。
4. 在清單中搜尋 **Phi-4-multimodal-instruct** 模型，然後選取並確認。
5. 如果出現提示，請同意授權合約，然後透過在部署詳細資料中選取 [自訂]**** 來使用以下設定部署模型：
    - **部署名稱**：*模型部署的有效名稱*
    - **部署類型**：全球標準
    - **部署詳細資料**： *使用預設設定*
6. 等待部署配置狀態為 [完成]****。

## 在遊樂場中測試模型。

現在您已部署多模式模型，您可以在聊天遊樂場中使用以影像為基礎的提示進行測試。

1. 在左側瀏覽窗格中，選取 **[遊樂場]** 頁面，然後開啟 **[聊天** 遊樂場]。
1. 1. 在新的瀏覽器索引標籤中，從 `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg`下載 [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg)，並將其儲存至本機檔案系統上的資料夾。
1. 在聊天遊樂場頁面上的 **[設定]** 窗格中，確定已選取 **Phi-4-multimodal-instruct** 模型部署。
1. 在主要聊天工作階段面板的聊天輸入方塊底下，使用 [附加] 按鈕 (**&#128206;**) 上傳 *mango.jpeg* 影像檔，然後新增文字 `What desserts could I make with this fruit?`並提交提示。

    ![螢幕擷取畫面顯示聊天遊樂場，其中已選取影像和提示。](../media/chat-playground-image.png)

1. 檢閱回應，希望為您可以使用芒果製作甜點提供相關指導。

## 建立用戶端應用程式

現在您已部署模型，您可以在用戶端應用程式中使用部署。

> **提示**：您可以選擇使用 Python 或 Microsoft C# 開發解決方案* (即將推出)*。 請遵循您所選語言的相應章節中的說明進行操作。

### 準備應用程式組態

1. 在 Azure AI Foundry 入口網站中，檢視專案的**概觀**頁面。
2. 在 [專案詳細資料]**** 區域中，記下 [專案連接字串]****。 您將使用此連接字串連線到用戶端應用程式中的專案。
3. 開啟一個新的瀏覽器索引標籤（保持 Azure AI Foundry 入口網站在現有索引標籤中開啟）。 然後在新索引標籤中，瀏覽到 `https://portal.azure.com` 的 [Azure 入口網站](https://portal.azure.com)；如果出現提示，請使用您的 Azure 認證登入。

    關閉任何歡迎通知，以查看 Azure 入口網站 首頁。

1. 使用頁面頂部搜尋欄右側的 **[\>_]** 按鈕在 Azure 入口網站中建立一個新的 Cloud Shell，並選擇 ***PowerShell 環境*** (訂用帳戶中沒有儲存體)。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。 您可以調整或最大化此窗格的大小，以便更輕鬆地使用。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

5. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：


    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

7. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    **Python**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/python
    ```

    **C#**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/c-sharp
    ```

8. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

9. 輸入以下命令，編輯已提供的設定檔：

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    程式碼編輯器中會開啟檔案。

10. 在程式碼檔案中，將 **your_project_connection_string** 預留位置替換為您的專案的連接字串（從 Azure AI Foundry 入口網站中的專案 **概觀** 頁面複製），並將 **your_model_deployment** 預留位置替換為您指派給 Phi-4-multimodal-instruct 模型部署的名稱。
11. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或 **按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或 **按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

### 撰寫程式碼以連線至您的專案，並與模型聊天

> **提示**：新增程式碼時，請確保保持正確的縮排。

1. 輸入以下命令，編輯已提供的\程式碼檔案：

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

2. 在程式碼檔案中，記下檔案頂端新增的現有語句，以匯入必要的 SDK 命名空間。 然後，在註解 **[新增參考]** 下，新增以下程式碼來參考您先前安裝之程式庫中的命名空間：

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        ImageContentItem,
        ImageUrl,
   )
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. 在 **main** 函式中，在註解 [取得組態設定]**** 下，請注意程式碼會載入您在設定檔中定義的專案連接字串和模型部署名稱值。
4. 在註解 **[初始化專案用戶端]** 下，新增以下程式碼，使用您目前登入的 Azure 認證連接到您的 Azure AI Foundry 專案：

    **Python**

    ```python
   # Initialize the project client
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. 在註解 **[取得聊天用戶端]** 下，新增下列程式碼以建立用戶端物件來與模型聊天：

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

### 撰寫程式碼以提交 URL 型影像提示

1. 請注意，程式碼包含迴圈，可讓使用者輸入提示，直到他們輸入 "quit" 為止。 然後在迴圈區段中，尋找註解 **[取得影像輸入的回應]**，新增下列程式代碼以提交包含下列影像的提示：

    ![芒果的照片。](../media/orange.jpeg)

    **Python**

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imageUrl = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(new Uri(imageUrl))
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. **使用 CTRL+S** 命令將變更儲存至程式碼檔案 - 但尚未關閉。

3. 在 Cloud Shell 命令列窗格中，輸入下列命令來執行應用程式：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. 當系統提示時，輸入下列提示：

    ```
   Suggest some recipes that include this fruit
    ```

5. 檢閱回應。 然後輸入 `quit`以結束程式。

### 修改程式碼以上傳本機圖像檔

1. 在應用程式程式代碼的程式代碼編輯器中，於 [迴圈] 區段中，尋找您先前在註解 [取得影像輸入的回應]**** 下新增的程式碼。 然後修改程式碼，如下所示，以上傳此本機圖像檔：

    ![火龍果的照片。](../media/mystery-fruit.jpeg)

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

    # Read and encode the image file
    with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

    # Include the image file data in the prompt
    data_url = f"data:{mime_type};base64,{base64_encoded_data}"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);
    
   // Include the image file data in the prompt
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(bytes: binaryImage, mimeType: mimeType) 
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. **使用 CTRL+S** 命令，將您的變更儲存至程式代碼檔案。 您也可以視需要關閉程式代碼編輯器 （**CTRL+Q**）。

3. 在 Cloud Shell 命令列窗格中，輸入下列命令來執行應用程式：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. 當系統提示時，輸入下列提示：

    ```
   What is this fruit? What recipes could I use it in?
    ```

5. 檢閱回應。 然後輸入 `quit`以結束程式。

    > **注意**：在這個簡單的應用程式中，我們還沒有實現保留交談記錄的邏輯；因此模型會將每個提示視為一個新要求，而沒有前一個提示的上下文。

## 進一步探索：(如果時間允許)

您已瞭解如何使用 Azure AI 推斷 SDK 和多模式模型來實作可回應影像型提示的產生式 AI 應用程式。 如果您有時間，以下是一些進一步探索的想法。

### 使用不同的多模式模型

您已使用 *Phi-4-multimodal-instruct* 模型來產生影像型提示的回應。 現在讓我們嘗試 OpenAI *gpt-4o* 模型。

1. 在 Azure AI Foundry 中，將 **gpt-4o** 模型部署至 Azure AI 模型推斷端點（您可能需要在不同的區域中建立新的資源）。
1. 更新您應用程式的程式碼組態檔（*.env* 適用於 Python，*appsettings.json* 適用於 C#），以指定您的 gpt-4o 模型的名稱。
1. 如先前一樣執行應用程式，使用相同的提示（您可以還原為使用URL型影像的程序代碼，如有需要）。

### 使用 OpenAI API

您在此練習中使用的程式代碼是以 Azure AI 推斷 SDK 為基礎，其適用於部署至 Azure AI 模型推斷端點的任何模型。 使用 OpenAI 模型時，您也可以使用 OpenAI SDK。

下列指示假設您已完成此練習和上述額外工作，以部署及測試 **gpt-4o** 模型。

1. 安裝 (或更新) 應用程式所需的套件：

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai
    ```
    
    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

1. 更新程式代碼檔案中的命名空間（移除 *azure.ai-inference* 參考）：

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   import openai
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. 修改程式碼以取得聊天用戶端：

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient chatClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

1. 修改程式碼以根據本機影像檔案取得回應

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = chat_client.chat.completions.create(
        model=model_deployment,
        messages=[
            { "role": "system", "content": system_message },
            { "role": "user", "content": [  
                { 
                    "type": "text", 
                    "text": prompt 
                },
                { 
                    "type": "image_url",
                    "image_url": {
                        "url": data_url
                    }
                }
            ] } 
        ]
   )
   completion = response.choices[0].message.content
   print(completion)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);

   // Include the image file data in the prompt
   List<ChatMessage> messages =
   [
        new SystemChatMessage(system_message),
        new UserChatMessage(
            ChatMessageContentPart.CreateTextPart(prompt),
            ChatMessageContentPart.CreateImagePart(binaryImage, mimeType)),
   ];

   ChatCompletion completion = chatClient.CompleteChat(messages);
   Console.WriteLine(completion.Content[0].Text);
    ```

1. 儲存變更並執行應用程式，以與您先前使用的相同提示進行測試。

## 清理

如果您已完成 Azure AI 語音探索，您應該刪除在本練習中建立的資源，以避免產生不必要的 Azure 成本。。

1. 返回包含 Azure 入口網站的瀏覽器索引標籤 (或在新的瀏覽器索引標籤中重新開啟位於`https://portal.azure.com`的 [Azure 入口網站](https://portal.azure.com))，並檢視您在其中部署本練習所用資源的資源群組內容。
1. 在工具列上，選取 [刪除資源群組]****。
1. 輸入資源群組名稱並確認您想要將其刪除。
