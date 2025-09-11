---
lab:
  title: 讀取圖片中的文字
  module: Module 11 - Reading Text in Images and Documents
---

# 讀取圖片中的文字

光學字元辨識 (OCR) 是電腦視覺的子集，處理在圖片及文件中讀取文字。 **Azure AI 視覺**服務提供讀取文字的 API，您將在此練習中探索該 API。

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
1. 等候部署完成，然後檢視部署的詳細資料。
1. 部署資源後，請移至該資源並檢視其 [金鑰和端點] **** 頁面。 在下一個程序中，您需要此頁面中的端點和其中一個金鑰。

## 複製本課程的存放庫

您將可使用 Azure 入口網站中的 Cloud Shell，開發程式碼。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：若您最近已複製 **mslearn-ai-vision** 存放庫，您可以跳過這項工作。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## 準備使用 Azure AI 視覺 SDK

在此練習中，您將完成已部分實作、使用 Azure AI 視覺 SDK 來讀取文字的用戶端應用程式。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 瀏覽至包含您慣用介面語言的應用程式程式碼檔案的資料夾：  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
    ```

1. 針對您的使用語言執行適當的命令，以安裝 Azure AI 視覺 SDK 套件和必要相依性：

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. 使用 `ls` 命令，您可以檢視 **computer-vision** 資料夾的內容。 您會發現其中還包含組態設定的檔案：

    - **C#**：appsettings.json
    - **Python**：.env

1. 輸入以下命令，編輯已提供的設定檔：

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中，更新其所包含的設定值，以反映電腦視覺資源的**端點**和驗證**金鑰**。
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 使用 Azure AI 視覺 SDK 從影像中讀取文字

**Azure AI 視覺 SDK** 的功能之一，可從影像中讀取文字。 在此練習中，您將完成已部分實作、使用 Azure AI 視覺 SDK 從影像中讀取文字的用戶端應用程式。

1. **read-text** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：read-text.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解底下新增下列語言特有程式碼，以匯入您使用 Azure AI 視覺 SDK 所需的命名空間：

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

1. 在用戶端應用程式的程式碼檔案中，**Main** 函式中已提供可載入組態設定的程式碼。 然後，尋找**驗證 Azure AI 視覺用戶端**註解。 然後，在此註解底下新增下列語言特有程式碼，以建立及驗證 Azure AI 視覺用戶端物件：

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

1. 在您剛在 **Main** 函式中新增的程式碼底下，您會看到程式碼指定了影像檔的路徑，然後將影像路徑傳遞至 **GetTextRead** 函式。 此函式尚未完全實作。

1. 將一些程式碼新增至 **GetTextRead** 函式的主體。 尋找**使用分析影像函式讀取影像中的文字**註解。 然後，在此註解底下新增下列語言特定程式碼，指出在呼叫 `Analyze` 函式時會指定視覺功能：

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
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
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

1. 在您剛在 **GetTextRead** 函式內新增的程式碼 中，於**傳回在影像中偵測到的文字**註解底下新增下列程式碼 (此程式碼會將影像文字列印至主控台，並產生將影像的文字醒目提示的影像 **text.jpg**)：

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
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

1. 針對 C# 應用程式檔案，還需要協助程式函式來繪製多邊形。 在 **Helper method to draw a polygon given an array of SKPoints** 註解下方，新增下列程式碼：

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. 在 **Main** 函式中，檢查使用者選取功能表選項 **1** 時所執行的程式碼。 此程式碼會呼叫 **GetTextRead** 函式，並傳遞 *Lincoln.jpg* 影像檔的路徑。

1. 儲存變更並關閉程式碼編輯器。

1. 在 Cloud Shell 工具列中，選取 [上傳/下載檔案]****，然後選取 [下載]****。 在新的對話方塊中，輸入下列檔案路徑，然後選取 [下載]****：

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. 開啟 **Lincoln.jpg** 影像以檢視。

1. 檢視程式碼處理的影像之後，請輸入下列命令以執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 出現提示時，輸入 **1** 並觀察輸出，其為從圖片中擷取的文字。

1. 在 **read-text** 資料夾中，已建立 **text.jpg** 影像。 您可以使用檔案路徑 `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` 下載，並注意每個文字 *行* 周圍的多邊形。

1. 重新開啟程式碼檔案並尋找 **Return the position bounding box around each line** 註解。 然後，在此註解底下新增下列程式碼：

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

1. 儲存變更並關閉程式碼編輯器，然後輸入下列命令以執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 在出現提示時輸入 **1**，並觀察輸出，這應該是影像中的每一行文字及其各自在影像中的位置。

1. 再次開啟程式碼檔案並尋找 **Return each word detected in the image and the position bounding box around each word with the confidence level of each word** 註解。 然後，在此註解底下新增下列程式碼：

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
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

1. 儲存變更並關閉程式碼編輯器，然後輸入下列命令以執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 在出現提示時輸入 **1**，並觀察輸出，這應該是影像中的每個文字及其各自在影像中的位置。 請注意，也會傳回每個字的信賴等級。

1. 再次下載 **text.jpg** 影像，您會發現每個*單字*周圍有多邊形。

## 使用 Azure AI 視覺 SDK 從影像中讀取手寫文字

在上一個練習中，您從影像中讀取了定義完善的文字，但有時您可能也想要從手寫筆記或文件中讀取文字。 好消息是，**Azure AI 視覺 SDK** 也可以使用您讀取妥善定義的文字時所使用的相同程式碼，來讀取手寫文字。 我們將使用上一個練習中的相同程式碼，但這次將使用不同的影像。

1. 使用檔案路徑 `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` 下載 **Note.jpg**，以檢視程式碼處理的下一個影像。

1. 在應用程式的程式碼檔案中，在 **Main** 函式中，檢查使用者選取功能表選項 **2** 時所執行的程式碼。 此程式碼會呼叫 **GetTextRead** 函式，並傳遞 *Note.jpg* 影像檔的路徑。

1. 在終端機上，輸入下列命令以執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 在出現提示時輸入 **2**，並觀察輸出，這是從筆記影像中擷取的文字。

1. 再次下載 **text.jpg** 影像，您會發現筆記的每個*單字*周圍有多邊形。

## 清除資源

如果您不將此實驗室中建立的 Azure 資源用於其他訓練模組，可以將其刪除，以避免產生進一步的費用。 方法如下：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

1. 在頂端搜尋列中，搜尋 [電腦視覺]**，然後選取您在此實驗室中建立的電腦視覺資源。

1. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

如需使用 **Azure AI 視覺**服務來讀取文字的詳細資訊，請參閱 [Azure AI 視覺文件](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)。
