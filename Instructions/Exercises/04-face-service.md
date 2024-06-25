---
lab:
  title: 偵測並分析臉部
  module: Module 4 - Detecting and Analyze Faces
---

# 偵測並分析臉部

偵測並分析人臉的能力，是核心 AI 功能。 在此練習中，您將探索兩個可用來對影像中的臉部進行處理的 Azure AI 服務：**Azure AI 視覺**服務和**臉部**服務。

> **重要**：無須要求對受限的功能進行任何額外存取，即可完成此實驗室。

> **注意**：從 2022 年 6 月 21 日開始，傳回個人識別資訊的 Azure AI 服務功能僅限已獲授與[有限存取權](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客戶使用。 此外，無法再使用推斷情緒狀態的功能。 如需 Microsoft 所做之變更和原因的詳細資訊，請參閱[臉部辨識的負責任 AI 投資和保護](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (英文)。

## 複製本課程的存放庫

如果您尚未完成此步驟，您必須複製本課程的程式碼存放庫：

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 佈建 Azure AI 服務資源

如果您的訂用帳戶中還沒有 **Azure AI 服務**資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 在頂端搜尋列中搜尋 *Azure AI 服務*、選取 [Azure AI 服務]****，並使用下列設定建立 Azure AI 服務多服務帳戶資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*選擇任何可用的區域*
    - **名稱**：輸入唯一名稱**
    - **定價層**:標準 S0
3. 選取必要的核取方塊並建立資源。
4. 等候部署完成，然後檢視部署的詳細資料。
5. 部署資源後，請移至該資源並檢視其 [金鑰和端點] **** 頁面。 在下一個程序中，您需要此頁面中的端點和其中一個金鑰。

## 準備使用 Azure AI 視覺 SDK

在此練習中，您將完成已部分實作、使用 Azure AI 視覺 SDK 對影像中的臉部進行分析的用戶端應用程式。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **04-face** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **computer-vision** 資料夾，然後開啟整合式終端機。 然後，針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 視覺 SDK 套件：

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 0.15.1-beta.1
    ```

    **Python**

    ```
    pip install azure-ai-vision==0.15.1b1
    ```
    
3. 檢視 **computer-vision** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#**：appsettings.json
    - **Python**：.env

4. 開啟組態檔並更新其所包含的組態值，以反映 Azure AI 服務資源的**端點**和驗證**金鑰**。 儲存您的變更。

5. 請注意，**computer-vision** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：detect-people.py

6. 開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解底下新增下列語言特有程式碼，以匯入您使用 Azure AI 語音 SDK 所需的命名空間：

    **C#**

    ```C#
    // import namespaces
    using Azure.AI.Vision.Common;
    using Azure.AI.Vision.ImageAnalysis;
    ```

    **Python**

    ```Python
    # import namespaces
    import azure.ai.vision as sdk
    ```

## 檢視您將分析的影像

在此練習中，您將使用 Azure AI 視覺服務來分析人物的影像。

1. 在 Visual Studio Code 中，展開 **computer-vision** 資料夾及其包含的 **images** 資料夾。
2. 選取 **people.jpg** 影像加以檢視。

## 偵測影像中的人臉

您現在已準備好使用 SDK 來呼叫視覺服務，並偵測影像中的臉部。

1. 在用戶端應用程式的程式碼檔案 (**Program.cs** 或 **detect-people.py**) 中，請注意 **Main** 函式中已提供可載入組態設定的程式碼。 然後，尋找**驗證 Azure AI 視覺用戶端**註解。 然後，在此註解底下新增下列語言特有程式碼，以建立及驗證 Azure AI 視覺用戶端物件：

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    var cvClient = new VisionServiceOptions(
        aiSvcEndpoint,
        new AzureKeyCredential(aiSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    cv_client = sdk.VisionServiceOptions(ai_endpoint, ai_key)
    ```

2. 在您剛在 **Main** 函式中新增的程式碼底下，您會看到程式碼指定了影像檔的路徑，然後將影像路徑傳遞至名為 **AnalyzeImage** 的函式。 此函式尚未完全實作。

3. 在 **AnalyzeImage** 函式的**指定要擷取的特徵 (人員)** 註解底下，新增下列程式碼：

    **C#**

    ```C#
    // Specify features to be retrieved (PEOPLE)
    Features =
        ImageAnalysisFeature.People
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (PEOPLE)
    analysis_options = sdk.ImageAnalysisOptions()
    
    features = analysis_options.features = (
        sdk.ImageAnalysisFeature.PEOPLE
    )    
    ```

4. 在 **AnalyzeFaces** 函式的**取得影像分析**註解底下，新增下列程式碼：

    **C#**

    ```C
    // Get image analysis
    using var imageSource = VisionSource.FromFile(imageFile);
    
    using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);
    
    var result = analyzer.Analyze();
    
    if (result.Reason == ImageAnalysisResultReason.Analyzed)
    {
        // Get people in the image
        if (result.People != null)
        {
            Console.WriteLine($" People:");
        
            // Prepare image for drawing
            System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
            Graphics graphics = Graphics.FromImage(image);
            Pen pen = new Pen(Color.Cyan, 3);
            Font font = new Font("Arial", 16);
            SolidBrush brush = new SolidBrush(Color.WhiteSmoke);
        
            foreach (var person in result.People)
            {
                // Draw object bounding box if confidence > 50%
                if (person.Confidence > 0.5)
                {
                    // Draw object bounding box
                    var r = person.BoundingBox;
                    Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
                    graphics.DrawRectangle(pen, rect);
        
                    // Return the confidence of the person detected
                    Console.WriteLine($"   Bounding box {person.BoundingBox}, Confidence {person.Confidence:0.0000}");
                }
            }
        
            // Save annotated image
            String output_file = "detected_people.jpg";
            image.Save(output_file);
            Console.WriteLine("  Results saved in " + output_file + "\n");
        }
    }
    else
    {
        var errorDetails = ImageAnalysisErrorDetails.FromResult(result);
        Console.WriteLine(" Analysis failed.");
        Console.WriteLine($"   Error reason : {errorDetails.Reason}");
        Console.WriteLine($"   Error code : {errorDetails.ErrorCode}");
        Console.WriteLine($"   Error message: {errorDetails.Message}\n");
    }
    
    ```

    **Python**
    
    ```Python
    # Get image analysis
    image = sdk.VisionSource(image_file)
    
    image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)
    
    result = image_analyzer.analyze()
    
    if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:
        # Get people in the image
        if result.people is not None:
            print("\nPeople in image:")
        
            # Prepare image for drawing
            image = Image.open(image_file)
            fig = plt.figure(figsize=(image.width/100, image.height/100))
            plt.axis('off')
            draw = ImageDraw.Draw(image)
            color = 'cyan'
        
            for detected_people in result.people:
                # Draw object bounding box if confidence > 50%
                if detected_people.confidence > 0.5:
                    # Draw object bounding box
                    r = detected_people.bounding_box
                    bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
                    draw.rectangle(bounding_box, outline=color, width=3)
            
                    # Return the confidence of the person detected
                    print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
                    
            # Save annotated image
            plt.imshow(image)
            plt.tight_layout(pad=0)
            outputfile = 'detected_people.jpg'
            fig.savefig(outputfile)
            print('  Results saved in', outputfile)
    
    else:
        error_details = sdk.ImageAnalysisErrorDetails.from_result(result)
        print(" Analysis failed.")
        print("   Error reason: {}".format(error_details.reason))
        print("   Error code: {}".format(error_details.error_code))
        print("   Error message: {}".format(error_details.message))
    ```

5. 儲存變更並返回 **computer-vision** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 觀察輸出，其應該指出偵測到的臉部數目。
7. 檢視在與程式碼檔案相同的資料夾中產生的 **detected_people.jpg** 檔案，以查看標註的臉部。 在此情況下，您的程式碼已使用臉部的特性來標示方塊左上角的位置，而周框方塊座標會在每個臉部周圍繪製矩形。

## 準備使用臉部 SDK

雖然 **Azure AI 視覺**服務提供基本臉部偵測 (以及許多其他影像分析功能)，但**臉部**服務提供更完整的臉部分析和辨識功能。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **04-face** 資料夾，然後根據您的語言喜好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **face-api** 資料夾，然後開啟整合式終端機。 然後針對您的語言喜好設定執行適當的命令，以安裝臉部 SDK 套件：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.8.0-preview.3
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.6.0
    ```
    
3. 檢視 **face-api** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#**：appsettings.json
    - **Python**：.env

4. 開啟組態檔並更新其所包含的組態值，以反映 Azure AI 服務資源的**端點**和驗證**金鑰**。 儲存您的變更。

5. 請注意，**face-api** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：analyze-faces.py

6. 開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解底下新增下列語言特有程式碼，以匯入您使用視覺 SDK 所需的命名空間：

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. 在 **Main** 函式中，請注意已經提供可載入組態設定的程式碼。 然後尋找 **Authenticate Face client** 註解。 然後，在此註解之下，新增下列語言特有程式碼，以建立及驗證 **FaceClient** 物件：

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. 在 **Main** 函式中您剛才新增的程式碼之下，請注意程式碼會顯示功能表，可讓您在程式碼中呼叫函式，以探索臉部服務的功能。 您將在本練習的其餘部分實作這些函式。

## 偵測並分析臉部

臉部服務的最基本功能之一就是偵測影像中的臉部，並判斷其特性，例如頭部姿勢、模糊、是否戴眼鏡等等。

1. 在應用程式的程式碼檔案中的 **Main** 函數中，檢查使用者選取功能表選項 **1** 時所執行的程式碼。 此程式碼會呼叫 **DetectFaces** 函式，並將路徑傳遞至影像檔。
2. 在程式碼檔案中尋找 **DetectFaces** 函式，並在 **Specify facial features to be retrieved** 註解之下，新增下列程式碼：

    **C#**

    ```C#
    // Specify facial features to be retrieved
    IList<FaceAttributeType> features = new FaceAttributeType[]
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. 在 **DetectFaces** 函式中您剛才新增的程式碼之下，尋找 **Get faces** 註解並新增下列程式碼：

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count() > 0)
    {
        Console.WriteLine($"{detected_faces.Count()} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.White);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face number {faceCount}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

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

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

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

4. 檢查您新增至 **DetectFaces** 函式的程式碼。 會分析影像檔案並偵測其包含的任何臉部，包括遮蔽、模糊及配戴眼鏡等屬性。 每個臉部的詳細資料都會顯示，包括指派給每個臉部的唯一臉部識別碼；而臉部的位置會使用週框方塊在影像上指出來。
5. 儲存變更並返回 **face-api** 資料夾的整合式終端機，然後輸入下列命令來執行程式：

    **C#**

    ```
    dotnet run
    ```

    *C# 輸出可能會顯示警告，有關現在使用 **await** 運算子之非同步函式。您可以忽略這些警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 出現提示時，請輸入 **1** 並觀察輸出，其中應該包含偵測到的每個臉部的識別碼和特性。
7. 檢視在與程式碼檔案相同的資料夾中產生的 **detected_faces.jpg** 檔案，以查看標註的臉部。

## 其他相關資訊

**臉部**服務中還有幾個額外功能可供使用，但需遵循[負責任 AI 標準](https://aka.ms/aah91ff)，並受有限存取權原則所限制。 這些功能包括識別、驗證及建立臉部辨識模型。 如需深入了解及申請存取權，請參閱 [Azure AI 服務的有限存取權](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)。

如需使用 **Azure AI 視覺**服務來偵測臉部的詳細資訊，請參閱 [Azure AI 視覺文件](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

若要深入了解**臉部**服務，請參閱[臉部文件](https://docs.microsoft.com/azure/cognitive-services/face/)。
