---
lab:
  title: 讀取圖片中的文字
  module: Module 11 - Reading Text in Images and Documents
---

# 讀取圖片中的文字

光學字元辨識 (OCR) 是電腦視覺的子集，處理在圖片及文件中讀取文字。 **Azure AI 視覺**服務提供讀取文字的 API，您將在此練習中探索該 API。

## 複製本課程的存放庫

如果您尚未複製本課程的程式碼存放庫，請務必先複製：

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。 如果出現提示訊息：「在資料夾中偵測到 Azure 函式專案」**，您可以放心關閉該訊息。

## 佈建 Azure AI 服務資源

如果您的訂用帳戶中還沒有 **Azure AI 服務**資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 在頂端搜尋列中搜尋 *Azure AI 服務*、選取 [Azure AI 服務]****，並使用下列設定建立 Azure AI 服務多服務帳戶資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*從美國東部、法國中部、南韓中部、北歐、東南亞、西歐、美國西部或東亞中選擇\**
    - **名稱**：輸入唯一名稱**
    - **定價層**:標準 S0

    \*Azure AI Vision 4.0 功能目前僅適用於這些區域。

3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署的詳細資料。
5. 部署資源後，請移至該資源並檢視其 [金鑰和端點] **** 頁面。 在下一個程序中，您將需要此頁面中的端點和其中一個金鑰。

## 準備使用 Azure AI 視覺 SDK

在此練習中，您將完成已部分實作、使用 Azure AI 視覺 SDK 來讀取文字的用戶端應用程式。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，請執行您的慣用語言適用的動作。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **Labfiles\05-ocr** 資料夾，然後根據您的語言偏好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 [read-text] **** 資料夾，然後開啟整合式終端。 然後，針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 視覺 SDK 套件：

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **注意**：如果出現安裝開發套件延伸模組的提示，您可以放心關閉訊息。

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. 檢視 **read-text** 資料夾的內容，並注意其中包含組態設定的檔案：

    - **C#**：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其所包含的組態值，以反映 Azure AI 服務資源的**端點**和驗證**金鑰**。 儲存您的變更。


## 使用 Azure AI 視覺 SDK 從影像中讀取文字

**Azure AI 視覺 SDK** 的功能之一，是從影像中讀取文字。 在此練習中，您將完成已部分實作、使用 Azure AI 視覺 SDK 從影像中讀取文字的用戶端應用程式。

1. **read-text** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：read-text.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解底下新增下列語言特有程式碼，以匯入您使用 Azure AI 視覺 SDK 所需的命名空間：

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. 在用戶端應用程式的程式碼檔案中，**Main** 函式中已提供可載入組態設定的程式碼。 然後，尋找**驗證 Azure AI 視覺用戶端**註解。 然後，在此註解底下新增下列語言特有程式碼，以建立及驗證 Azure AI 視覺用戶端物件：

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```
    
    **Python**
    
    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

3. 在您剛在 **Main** 函式中新增的程式碼底下，您會看到程式碼指定了影像檔的路徑，然後將影像路徑傳遞至 **GetTextRead** 函式。 此函式尚未完全實作。

4. 將一些程式碼新增至 **GetTextRead** 函式的主體。 尋找**使用分析影像函式讀取影像中的文字**註解。 然後，在此註解底下新增下列語言特定程式碼，指出在呼叫 `Analyze` 函式時會指定視覺功能：

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

5. 在您剛在 **GetTextRead** 函式內新增的程式碼 中，於**傳回在影像中偵測到的文字**註解底下新增下列程式碼 (此程式碼會將影像文字列印至主控台，並產生將影像的文字醒目提示的影像 **text.jpg**)：

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

6. 在 **read-text/images** 資料夾中選取 **Lincoln.jpg**，以檢視程式碼所處理的檔案。

7. 在應用程式的程式碼檔案中的 **Main** 函數中，檢查使用者選取功能表選項 **1** 時所執行的程式碼。 此程式碼會呼叫 **GetTextRead** 函式，並傳遞 *Lincoln.jpg* 影像檔的路徑。

8. 儲存您的變更並返回 **read-text** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. 出現提示時，輸入 **1** 並觀察輸出，其為從圖片中擷取的文字。

10. 在 **read-text** 資料夾中選取 **text.jpg** 影像，並注意到每*行*文字週圍都有多邊形。

11. 返回 Visual Studio Code 中的程式碼檔案，並尋找**傳回每一行週圍的位置週框方塊**註解。 然後，在此註解底下新增下列程式碼：

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");  
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

12. 儲存您的變更並返回 **read-text** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. 在出現提示時輸入 **1**，並觀察輸出，這應該是影像中的每一行文字及其各自在影像中的位置。


14. 返回 Visual Studio Code 中的程式碼檔案，並尋找**傳回在影像中偵測到的每個字，以及每個字週圍的位置週框方塊和每個字的信賴等級**註解。 然後，在此註解底下新增下列程式碼：

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

15. 儲存您的變更並返回 **read-text** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. 在出現提示時輸入 **1**，並觀察輸出，這應該是影像中的每個文字及其各自在影像中的位置。 請注意，也會傳回每個字的信賴等級。

17. 在 **read-text** 資料夾中選取 **text.jpg** 影像，並注意到每個*字*週圍都有多邊形。

## 使用 Azure AI 視覺 SDK 從影像中讀取手寫文字

在上一個練習中，您從影像中讀取了定義完善的文字，但有時您可能也想要從手寫筆記或文件中讀取文字。 好消息是，**Azure AI 視覺 SDK** 也可以使用您讀取妥善定義的文字時所使用的相同程式碼，來讀取手寫文字。 我們將使用上一個練習中的相同程式碼，但這次將使用不同的影像。

1. 在 **read-text/images** 資料夾中選取 **Note.jpg**，以檢視程式碼所處理的檔案。

2. 在應用程式的程式碼檔案中，在 **Main** 函式中，檢查使用者選取功能表選項 **2** 時所執行的程式碼。 此程式碼會呼叫 **GetTextRead** 函式，並傳遞 *Note.jpg* 影像檔的路徑。

3. 在 **read-text** 資料夾的整合式終端，輸入下列命令以執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. 在出現提示時輸入 **2**，並觀察輸出，這是從筆記影像中擷取的文字。

5. 在 **read-text** 資料夾中選取 **text.jpg** 影像，並注意到筆記的每個*字*週圍都有多邊形。

## 清除資源

如果您不將此實驗室中建立的 Azure 資源用於其他訓練模組，可以將其刪除，以避免產生進一步的費用。 方法如下：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

2. 在頂端搜尋列中，搜尋 *Azure AI 服務多服務帳戶*，並選取您在此實驗室中建立的 Azure AI 服務多服務帳戶資源

3. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

如需使用 **Azure AI 視覺**服務來讀取文字的詳細資訊，請參閱 [Azure AI 視覺文件](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)。
