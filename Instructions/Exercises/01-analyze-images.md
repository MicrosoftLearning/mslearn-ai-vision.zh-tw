---
lab:
  title: 使用 Azure AI 視覺分析影像
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# 使用 Azure AI 視覺分析影像

Azure AI 視覺是一種人工智慧功能，可讓軟體系統藉由分析影像來解譯視覺輸入。 在 Microsoft Azure 中，「視覺」**** Azure AI 服務會提供常見電腦視覺工作的預先建置模型，包括分析影像以建議標題和標籤、偵測常見物件和人物。 您也可以使用 Azure AI 視覺服務來移除背景，或建立影像的前景嵌空。

## 複製本課程的存放庫

如果您尚未將 **Azure AI 視覺**程式碼存放庫複製到您正在進行本實驗的環境，請遵循下列步驟來執行此動作。 否則，請在 Visual Studio Code 中開啟複製的資料夾。

1. 啟動 Visual Studio Code。
2. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
3. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
4. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。 如果出現提示訊息：「在資料夾中偵測到 Azure 函式專案」**，您可以放心關閉該訊息。

## 佈建 Azure AI 服務資源

如果您的訂用帳戶中還沒有 **Azure AI 服務**資源，則必須加以佈建。

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 選取 [建立資源]。
3. 在頂端搜尋列中，搜尋 *Azure AI 服務*，選取 [Azure AI 服務]****，並使用下列設定建立 Azure AI 服務多服務帳戶資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*選擇或建立資源群組 (如果您使用受限制的訂用帳戶，則您可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **區域**：*從美國東部、美國西部、法國中部、南韓中部、北歐、東南亞、西歐或東亞中選擇\**
    - **名稱**：輸入唯一名稱**
    - **定價層**:標準 S0

    \*Azure AI 視覺 4.0 完整功能目前僅適用於這些區域。

4. 選取必要的核取方塊並建立資源。
5. 等候部署完成，然後檢視部署的詳細資料。
6. 部署資源後，請移至該資源並檢視其 [金鑰和端點] **** 頁面。 在下一個程序中，您需要此頁面中的端點和其中一個金鑰。

## 準備使用 Azure AI 視覺 SDK

在此練習中，您將完成部分實作的用戶端應用程式，其會使用 Azure AI 視覺 SDK 來分析影像。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 **Labfiles/01-analyze-images** 資料夾，然後根據您的語言偏好設定展開 **C-Sharp** 或 **Python** 資料夾。
2. 以滑鼠右鍵按一下 **image-analysis** 資料夾，然後開啟整合式終端機。 然後，針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 視覺 SDK 套件：

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    > **注意**：如果出現安裝開發套件延伸模組的提示，您可以放心關閉訊息。

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```

    > **提示**：如果您在自己的電腦上執行此實驗室，您也需要安裝 `matplotlib` 與 `pillow`。
    
3. 檢視 **image-analysis** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#**：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其所包含的組態值，以反映 Azure AI 服務資源的**端點**和驗證**金鑰**。 儲存您的變更。
4. 請注意，**image-analysis** 資料夾包含用戶端應用程式的程式碼檔案：

    - **C#**：Program.cs
    - **Python**：image-analysis.py

    開啟程式碼檔案，然後在頂端的現有命名空間參考之下，尋找 **Import namespaces** 註解。 然後，在此註解底下新增下列語言特有程式碼，以匯入您使用 Azure AI 語音 SDK 所需的命名空間：

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
    
## 檢視您將分析的影像

在此練習中，您將使用 Azure AI 視覺服務來分析多張影像。

1. 在 Visual Studio Code 中，展開 **image-analysis** 資料夾及其包含的 **images** 資料夾。
2. 接著依序選取每個影像檔案以在 Visual Studio Code 中檢視。

## 分析影像以建議標題

您現在已準備好使用 SDK 來呼叫「視覺」服務和分析影像。

1. 在用戶端應用程式的程式碼檔案 (**Program.cs** 或 **image-analysis.py**) 中，請注意 **Main** 函式中已經提供可載入組態設定的程式碼。 然後，尋找**驗證 Azure AI 視覺用戶端**註解。 然後，在此註解底下新增下列語言特有程式碼，以建立及驗證 Azure AI 視覺用戶端物件：

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

2. 在 **Main** 函式中您剛才新增的程式碼之下，請注意程式碼會指定影像檔的路徑，然後將影像路徑傳遞至其他兩個函式 (**AnalyzeImage** 和 **BackgroundForeground**)。 這些函式尚未完全實作。

3. 在 **AnalyzeImage** 函式的註解 **Get result with specify features to be retrieved** 下方新增下列程式碼：

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
result = cv_client.analyze(
    image_data=image_data,
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.DENSE_CAPTIONS,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE],
)
```
    
4. 在 **AnalyzeImage** 函式的註解 **Display analysis results** 下方新增下列程式碼 (包括指出您稍後將新增更多程式碼的位置註解)：

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```
    
5. 儲存變更並返回 **image-analysis** 資料夾的整合式終端機，然後輸入下列命令來執行包含引數 **images/street.jpg** 程式：

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 觀察輸出，其中應該包含 **street.jpg** 影像的建議標題。
7. 再次執行程式，這次使用引數 **images/building.jpg** 來查看為 **building.jpg** 影像產生的標題。
8. 重複上一個步驟以產生 **images/person.jpg** 檔案的標題。

## 取得影像的建議標籤

有時候，識別提供影像內容線索的相關*標籤*可能很有用。

1. 在 **AnalyzeImage** 函式的 **Get image tags** 註解之下，新增下列程式碼：

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔案執行一次程式，觀察除了影像標題之外，是否也有顯示建議標籤的清單。

## 偵測並找出影像中的物件

*物件偵測*是一種特別形式的電腦視覺，用於識別影像內的個別物件，並用週框方塊指示其所在位置。

1. 在 **AnalyzeImage** 函式的 **Get objects in the image** 註解之下，新增下列程式碼：

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔案執行一次程式，並觀察是否偵測到任何物件。 在每次執行後，檢視在與程式碼檔案相同的資料夾中產生的 **objects.jpg** 檔案，以查看標註的物件。

## 偵測並找出影像中的人物

*人物偵測*是一種特別形式的電腦視覺，用於識別影像內的個別人物，並用週框方塊指示其所在位置。

1. 在 **AnalyzeImage** 函式的註解 **Get people in the image** 下方新增下列程式碼：

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (選用) 取消註解 [傳回偵測到人物的信賴度]**** 區段底下的 **Console.Writeline** 命令，以檢閱傳回特定影像位置偵測到人物的信賴度。

3. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔案執行一次程式，並觀察是否偵測到任何物件。 在每次執行後，檢視在與程式碼檔案相同的資料夾中產生的 **objects.jpg** 檔案，以查看標註的物件。

> **注意**：在上述工作中，您使用是單一方法來分析影像，然後以累加方式新增程式碼來剖析並顯示結果。 SDK 也提供個別方法來建議標題、識別標籤、偵測物件等等，這表示您可以使用最適當的方法來只傳回所需的資訊，減少需要傳回的資料承載大小。 如需詳細資訊，請參閱 [.NET SDK 文件](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet)或 [Python SDK 文件](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision)。

## 拿掉背景或產生影像的前景嵌空

在某些情況下，您可能需要建立已移除背景的影像，或可能會想要建立該影像的前景嵌空。 讓我們從背景移除開始。

1. 在您的程式碼檔案中，尋找 **BackgroundForeground** 函式；然後在註解 **Remove the background from the image or generate a foreground matte** 下方新增下列程式碼：

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("background.png", "wb") as file:
    file.write(image)
print('  Results saved in background.png \n')
```
    
2. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔執行一次程式，開啟與每個影像程式碼檔案相同資料夾中產生的 **background.png** 檔案。  您會注意到背景已從每個影像中移除。

現在讓我們為影像產生前景嵌空。

3. 在您的程式碼檔案中，尋找 **BackgroundForeground** 函式；然後在註解 **Define the API version and mode** 下方將模式變數變更為 `foregroundMatting`。

4. 儲存您的變更，並針對 **images** 資料夾中的每個影像檔執行一次程式，開啟與每個影像程式碼檔案相同資料夾中產生的 **background.png** 檔案。  您會注意到影像已產生前景嵌空。

## 清除資源

如果您不將此實驗室中建立的 Azure 資源用於其他訓練模組，可以將其刪除，以避免產生進一步的費用。 方法如下：

1. 開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。

2. 在頂端搜尋列中，搜尋「Azure AI 服務多服務帳戶」**，並選取您在此實驗室中所建立的 Azure AI 服務多服務帳戶資源。

3. 在資源頁面上選取 [刪除]****，然後依照指示刪除資源。

## 其他相關資訊

在此練習中，您已探索 Azure AI 視覺服務的一些影像分析和操作功能。 此服務也包含偵測物件和人物，以及其他電腦視覺工作。

如需使用 **Azure AI 視覺**服務的詳細資訊，請參閱 [Azure AI 視覺文件](https://learn.microsoft.com/azure/ai-services/computer-vision/)。
