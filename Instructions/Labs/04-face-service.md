---
lab:
  title: 偵測並分析臉部
  module: Module 4 - Detecting and Analyze Faces
---

# 偵測並分析臉部

偵測並分析人臉的能力，是核心 AI 功能。 在此練習中，您將探索兩個可用來對影像中的臉部進行處理的 Azure AI 服務：**Azure AI 視覺**服務和**臉部**服務。

> **重要**：無須要求對受限的功能進行任何額外存取，即可完成此實驗室。

> **注意**：從 2022 年 6 月 21 日開始，傳回個人識別資訊的 Azure AI 服務功能僅限已獲授與[有限存取權](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客戶使用。 此外，無法再使用推斷情緒狀態的功能。 如需 Microsoft 所做之變更和原因的詳細資訊，請參閱[臉部辨識的負責任 AI 投資和保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (英文)。

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
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## 準備使用 Azure AI 視覺 SDK

在此練習中，您將完成部分實作的用戶端應用程式，其會使用 Azure AI 視覺 SDK 來分析影像。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 瀏覽至包含您慣用介面語言的應用程式程式碼檔案的資料夾：  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
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
1. 取代預留位置後，使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

## 檢視您將分析的影像

在此練習中，您將使用 Azure AI 視覺服務來分析人物的影像。

1. 在 Cloud Shell 工具列中，選取 [上傳/下載檔案]****，然後選取 [下載]****。 在新的對話方塊中，輸入下列檔案路徑，然後選取 [下載]****：

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. 開啟 **people.jpg** 影像加以檢視。

## 偵測影像中的人臉

您現在已準備好使用 SDK 來呼叫視覺服務，並偵測影像中的臉部。

1. 請注意，**computer-vision** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：detect-people.py

1. 輸入以下命令，即可開啟用戶端應用程式的程式碼檔案：

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. 在 **Import namespaces** 註解下方新增下列特定語言程式碼，以匯入您使用 Azure AI 視覺 SDK 所需的命名空間：

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
    
1. 在 **Main** 函式中，請注意已經提供可載入組態設定的程式碼。 然後，尋找**驗證 Azure AI 視覺用戶端**註解。 然後，在此註解底下新增下列語言特有程式碼，以建立及驗證 Azure AI 視覺用戶端物件：

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
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

1. 在您剛在 **Main** 函式中新增的程式碼底下，您會看到程式碼指定了影像檔的路徑，然後將影像路徑傳遞至名為 **AnalyzeImage** 的函式。 此函式尚未完全實作。

1. 在 **AnalyzeImage** 函式的註解 **Get result with specify features to be retrieved (PEOPLE)** 下方新增下列程式碼：

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

1. 在 **AnalyzeImage** 函式的註解 **Draw bounding box around detected people**下方新增下列程式碼：

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

1. 儲存變更並關閉程式碼編輯器，然後輸入下列命令以執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 觀察輸出，其應該指出偵測到的臉部數目。
7. 下載在與程式碼檔案相同的資料夾中產生的 **people.jpg** 檔案，以查看已標註的臉部。 在此情況下，您的程式碼已使用臉部的特性來標示方塊左上角的位置，而周框方塊座標會在每個臉部周圍繪製矩形。

如果您想要查看服務偵測到的所有人員信賴度分數，您可以取消註解 `Return the confidence of the person detected` 下的程式碼行，然後重新執行程序碼。

## 準備使用臉部 SDK

雖然 **Azure AI 視覺**服務提供基本臉部偵測 (以及許多其他影像分析功能)，但**臉部**服務提供更完整的臉部分析和辨識功能。

1. 執行命令 `cd ../face-api` 以瀏覽至 **face-api** 資料夾。 然後針對您的語言喜好設定執行適當的命令，以安裝臉部 SDK 套件：

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. 檢視 **face-api** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#**：appsettings.json
    - **Python**：.env

1. 開啟組態檔並更新其所包含的組態值，以反映 Azure AI 服務資源的**端點**和驗證**金鑰**。 儲存您的變更。

1. 請注意，**face-api** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：analyze-faces.py

1. 開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解底下新增下列語言特有程式碼，以匯入您使用視覺 SDK 所需的命名空間：

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. 在 **Main** 函式中，請注意已經提供可載入組態設定的程式碼。 然後尋找 **Authenticate Face client** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以建立及驗證 **FaceClient** 物件：

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

1. 在 **Main** 函式中您剛才新增的程式碼之下，請注意程式碼會顯示功能表，可讓您在程式碼中呼叫函式，以探索臉部服務的功能。 您將在本練習的其餘部分實作這些函式。

## 偵測並分析臉部

臉部服務的最基本功能之一就是偵測影像中的人臉，並判斷其特性，例如頭部姿勢、模糊、是否遮罩等等。

1. 在應用程式的程式碼檔案中的 **Main** 函數中，檢查使用者選取功能表選項 **1** 時所執行的程式碼。 此程式碼會呼叫 **DetectFaces** 函式，並將路徑傳遞至影像檔。
1. 在程式碼檔案中尋找 **DetectFaces** 函式，並在 **Specify facial features to be retrieved** 註解之下，新增下列程式碼：

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

1. 在 **DetectFaces** 函式中您剛才新增的程式碼之下，尋找 **Get faces** 註解並新增下列程式碼：

    **C#**

    ```C#
    // Get faces
    using (var imageData = File.OpenRead(imageFile))
    {    
        var response = await faceClient.DetectAsync(
            BinaryData.FromStream(imageData),
            FaceDetectionModel.Detection03,
            FaceRecognitionModel.Recognition04,
            returnFaceId: false,
            returnFaceAttributes: features);
        IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

        if (detected_faces.Count() > 0)
        {
            Console.WriteLine($"{detected_faces.Count()} faces detected.");

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

            int faceCount=0;

            // Draw and annotate each face
            foreach (var face in detected_faces)
            {
                faceCount++;
                Console.WriteLine($"\nFace number {faceCount}");
            
                // Get face properties
                Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
                Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
                Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
                Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
                Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

                // Draw and annotate face
                var r = face.FaceRectangle;

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
        }
    }
    ```

    **Python**

    ```Python
    # Get faces
    with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=False,
            return_face_attributes=features,
        )

        if len(detected_faces) > 0:
            print(len(detected_faces), 'faces detected.')

            # Prepare image for drawing
            fig = plt.figure(figsize=(8, 6))
            plt.axis('off')
            image = Image.open(image_file)
            draw = ImageDraw.Draw(image)
            color = 'lightgreen'
            face_count = 0

            # Draw and annotate each face
            for face in detected_faces:
    
                # Get face properties
                face_count += 1
                print('\nFace number {}'.format(face_count))

                print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
                print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
                print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
                print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
                print(' - Mask: {}'.format(face.face_attributes.mask.type))

                # Draw and annotate face
                r = face.face_rectangle
                bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
                draw = ImageDraw.Draw(image)
                draw.rectangle(bounding_box, outline=color, width=5)
                annotation = 'Face number {}'.format(face_count)
                plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

            # Save annotated image
            plt.imshow(image)
            outputfile = 'detected_faces.jpg'
            fig.savefig(outputfile)

            print('\nResults saved in', outputfile)
    ```

1. 檢查您新增至 **DetectFaces** 函式的程式碼。 它會分析影像檔案並偵測其包含的任何臉部，包括頭部姿勢、模糊及是否遮罩的屬性。 每個臉部的詳細資料都會顯示，包括指派給每個臉部的唯一臉部識別碼；而臉部的位置會使用週框方塊在影像上指出來。
1. 儲存變更並關閉程式碼編輯器，然後輸入下列命令以執行程式：

    **C#**

    ```
    dotnet run
    ```

    *C# 輸出可能會顯示未使用 **await** 運算子之非同步函式警告。您可以忽略這些警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

1. 出現提示時，請輸入 **1** 並觀察輸出，其中應該包含偵測到的每個臉部的識別碼和特性。
1. 下載在與程式碼檔案相同的資料夾中產生的 **detected_faces.jpg** 檔案，以查看標註的臉部。

## 清除資源

如果您不將本實驗中所建立的 Azure 資源用於其他訓練模組，則可以將其刪除以避免產生進一步的費用。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，然後在頂端搜尋列中搜尋您在此實驗中所建立的資源。

1. 在資源頁面上，選取 [刪除]**** 並依照指示刪除該資源。 或者，您也可以刪除整個資源群組以同時清理所有資源。

## 其他相關資訊

**臉部**服務中還有幾個額外功能可供使用，但需遵循[負責任 AI 標準](https://aka.ms/aah91ff)，並受有限存取權原則所限制。 這些功能包括識別、驗證及建立臉部辨識模型。 如需深入了解及申請存取權，請參閱 [Azure AI 服務的有限存取權](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)。

如需使用 **Azure AI 視覺**服務來偵測臉部的詳細資訊，請參閱 [Azure AI 視覺文件](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

若要深入了解**臉部**服務，請參閱[臉部文件](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity)。
