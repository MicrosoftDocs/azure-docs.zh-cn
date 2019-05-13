---
title: 执行图像操作 - Java
titlesuffix: Azure Cognitive Services
description: 介绍一款使用 Azure 认知服务中的计算机视觉 API 的基本 Java Swing 应用。 执行 OCR、创建缩略图，并处理图像中的视觉特征。
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.author: kefre
ms.custom: seodec18
ms.date: 04/30/2019
ms.openlocfilehash: a22308e0c7ff924205f715692d011a4572b2bdb8
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232629"
---
# <a name="use-computer-vision-features-with-the-rest-api-and-java"></a>使用 REST API 和 Java 中使用计算机视觉功能

本教程展示 Azure 认知服务计算机视觉 REST API 的功能。

介绍一款 Java Swing 应用程序，它使用计算机视觉 REST API 执行光学符号识别 (OCR)、创建智能裁剪的缩略图，还检测、分类、标记和描述图像中的人脸等视觉特征。 通过本示例可提交一个图像 URL 进行分析和处理。 你可使用此开源示例作为模板，在 Java 中自行构建使用计算机视觉 REST API 的应用。

本教程将介绍如何使用计算机视觉执行以下操作：

> [!div class="checklist"]
> * 分析图像
> * 识别图像中的自然或人造地标
> * 识别图像中的名人
> * 创建图像的质量缩略图
> * 读取图像中的印刷体文本
> * 读取图像中的手写文本

Java Swing 窗体应用程序已编写但尚无功能。 在本教程中，你将添加特定于计算机视觉 REST API 的代码，以补全应用程序的功能。

## <a name="prerequisites"></a>先决条件

### <a name="platform-requirements"></a>平台要求

本教程是采用 NetBeans IDE 开发的。 具体而言，Java SE 版的 NetBeans 可[在此处下载](https://netbeans.org/downloads/index.html)。

### <a name="subscribe-to-computer-vision-api-and-get-a-subscription-key"></a>订阅计算机视觉 API 并获得订阅密钥

在创建之前示例中，您必须订阅计算机视觉 API 的 Azure 认知服务的一部分。 有关订阅和密钥管理的详细信息，请参阅[订阅](https://azure.microsoft.com/try/cognitive-services/)。 主密钥和辅助密钥均适用于本教程。

## <a name="acquire-incomplete-tutorial-project"></a>获取不完整的教程项目

### <a name="download-the-project"></a>下载项目

1. 转到[认知服务 Java 计算机视觉教程](https://github.com/Azure-Samples/cognitive-services-java-computer-vision-tutorial)存储库。
1. 单击“克隆”或“下载”按钮。
1. 单击“下载 ZIP”，下载教程项目的 .zip 文件。

无需提取 .zip 文件的内容，因为 NetBeans 从 .zip 文件导入项目。

### <a name="import-the-tutorial-project"></a>导入教程项目

将 cognitive-services-java-computer-vision-tutorial-master.zip 文件导入 NetBeans。

1. 在 NetBeans 中单击“文件” > “导入项目” > “从 ZIP...”。随即出现“从 ZIP 导入项目”对话框。
1. 在“ZIP 文件:”字段中，单击“浏览”按钮找到 cognitive-services-java-computer-vision-tutorial-master.zip 文件，然后单击“打开”。
1. 单击“从 ZIP 导入项目”对话框中的“导入”。
1. 在“项目”面板中，展开“ComputerVision” > “源包” > &lt;默认包&gt;。 
   某些版本的 NetBeans 使用“src”而不是“源包” > &lt;默认包&gt;。 在这种情况下，展开“src”。
1. 双击“MainFrame.java”，将文件加载到 NetBeans 编辑器。 随即出现 MainFrame.java 文件的“设计”选项卡。
1. 单击“源”选项卡查看 Java 源代码。

### <a name="build-and-run-the-tutorial-project"></a>生成并运行教程项目

1. 按 F6 生成并运行教程应用程序。

    在教程应用程序中，单击选项卡以显示该功能的窗格。 按钮的方法为空，因此不执行任何操作。

    窗口底部是字段“订阅密钥”和“订阅区域”。 必须使用有效的订阅密钥和该订阅密钥正确的区域填充这些字段。 要获取订阅密钥，请参阅[订阅](https://azure.microsoft.com/try/cognitive-services/)。 如果是通过该链接获取的免费试用版的订阅密钥，则默认区域 **westcentralus** 是订阅密钥的正确区域。

1. 退出教程应用程序。

## <a name="add-tutorial-code-to-the-project"></a>将教程代码添加到项目

为 Java Swing 应用程序设置了六个选项卡。 每个选项卡展示计算机视觉的不同功能（分析、OCR 等）。 六个教程部分没有相互依赖关系，因此可以添加一个部分、六个部分全部添加，或者添加任意子集。 可以按任意顺序添加部分。

### <a name="analyze-an-image"></a>分析图像

计算机视觉的分析功能可扫描图像中超过 2,000 个可识别的对象、生物、风景和动作。 分析完成后，分析功能返回一个 JSON 对象，它使用描述性的标签、色彩分析和题注等解说图像。

要完成教程应用程序的分析功能，请执行以下步骤：

#### <a name="add-the-event-handler-code-for-the-analyze-button"></a>添加分析按钮的事件处理程序代码

analyzeImageButtonActionPerformed 事件处理程序方法将表单清空、显示 URL 中指定的图像，然后调用 AnalyzeImage 方法来分析图像。 当 AnalyzeImage 返回时，该方法会在“响应”文本区域中显示格式化的 JSON 响应，从 JSONObject 中提取第一个标题，并显示标题以及标题正确的可信度。

将以下代码复制粘贴到 analyzeImageButtonActionPerformed 方法。

> [!NOTE]
> NetBeans 不允许粘贴到方法定义行 (```private void```) 或该方法的右大括号。 要复制代码，请复制方法定义和右大括号之间的代码行，并将其粘贴到方法的内容中。

```java
    private void analyzeImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL analyzeImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        analyzeImage.setIcon(new ImageIcon());
        analyzeCaptionLabel.setText("");
        analyzeResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            analyzeImageUrl = new URL(analyzeImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(analyzeImageUrl);
            scaleAndShowImage(bImage, analyzeImage);
        } catch(IOException e) {
            analyzeResponseTextArea.setText("Error loading Analyze image: " + e.getMessage());
            return;
        }
        
        // Analyze the image.
        JSONObject jsonObj = AnalyzeImage(analyzeImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        analyzeResponseTextArea.setText(jsonObj.toString(2));
                
        // Extract the text and confidence from the first caption in the description object.
        if (jsonObj.has("description") && jsonObj.getJSONObject("description").has("captions")) {

            JSONObject jsonCaption = jsonObj.getJSONObject("description").getJSONArray("captions").getJSONObject(0);
            
            if (jsonCaption.has("text") && jsonCaption.has("confidence")) {
                
                analyzeCaptionLabel.setText("Caption: " + jsonCaption.getString("text") + 
                        " (confidence: " + jsonCaption.getDouble("confidence") + ").");
            }
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>添加用于 REST API 调用的包装器

AnalyzeImage 方法包装 REST API 调用以分析图像。 该方法返回描述图像的 JSONObject，或者如果出现错误，则返回 null。

复制 AnalyzeImage 方法并将其粘贴到 analyzeImageButtonActionPerformed 方法的正下方。

```java
    /**
     * Encapsulates the Microsoft Cognitive Services REST API call to analyze an image.
     * @param imageUrl: The string URL of the image to analyze.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject AnalyzeImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call for Analyze Image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseAnalyze;
            URIBuilder builder = new URIBuilder(uriString);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color,Adult");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
 ```

#### <a name="run-the-analyze-function"></a>运行分析函数

按 F6 运行应用程序。 在“订阅密钥”字段中填写订阅密钥，并验证确保“订阅区域中”使用的区域正确无误。 输入图像 URL 进行分析，然后单击“分析图像”按钮以分析图像并查看结果。

### <a name="recognize-a-landmark"></a>识别地标

计算机视觉的地标功能可分析图像中的自然和人造地标，例如山脉或著名的建筑物。 分析完成后，地标功能返回一个 JSON 对象，它标识在图像中找到的地标。

要完成教程应用程序的地标功能，请执行以下步骤：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>为表单按钮添加事件处理程序代码

landmarkImageButtonActionPerformed 事件处理程序方法将表单清空、显示 URL 中指定的图像，然后调用 LandmarkImage 方法来分析图像。 当 LandmarkImage 返回时，该方法会在“响应”文本区域中显示格式化的 JSON 响应，然后从 JSONObject 中提取第一个地标名称并在窗口中显示该名称以及正确识别地标的可信度。

将以下代码复制粘贴到 landmarkImageButtonActionPerformed 方法。

> [!NOTE]
> NetBeans 不允许粘贴到方法定义行 (```private void```) 或该方法的右大括号。 要复制代码，请复制方法定义和右大括号之间的代码行，并将其粘贴到方法的内容中。

```java
    private void landmarkImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL landmarkImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        landmarkImage.setIcon(new ImageIcon());
        landmarkCaptionLabel.setText("");
        landmarkResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            landmarkImageUrl = new URL(landmarkImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(landmarkImageUrl);
            scaleAndShowImage(bImage, landmarkImage);
        } catch(IOException e) {
            landmarkResponseTextArea.setText("Error loading Landmark image: " + e.getMessage());
            return;
        }
        
        // Identify the landmark in the image.
        JSONObject jsonObj = LandmarkImage(landmarkImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        landmarkResponseTextArea.setText(jsonObj.toString(2));
                
        // Extract the text and confidence from the first caption in the description object.
        if (jsonObj.has("result") && jsonObj.getJSONObject("result").has("landmarks")) {

            JSONObject jsonCaption = jsonObj.getJSONObject("result").getJSONArray("landmarks").getJSONObject(0);
            
            if (jsonCaption.has("name") && jsonCaption.has("confidence")) {

                landmarkCaptionLabel.setText("Caption: " + jsonCaption.getString("name") + 
                        " (confidence: " + jsonCaption.getDouble("confidence") + ").");
            }
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>添加用于 REST API 调用的包装器

LandmarkImage 方法包装 REST API 调用以分析图像。 该方法返回描述图像中找到的地标的 JSONObject，或者如果出现错误，则返回 null。

复制 LandmarkImage 方法并将其粘贴到 landmarkImageButtonActionPerformed 方法的正下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to identify a landmark in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject LandmarkImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to identify a Landmark in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseLandmark;
            URIBuilder builder = new URIBuilder(uriString);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-landmark-function"></a>运行地标函数

按 F6 运行应用程序。 在“订阅密钥”字段中填写订阅密钥，并验证确保“订阅区域中”使用的区域正确无误。 单击“地标”选项卡，输入地标图像的 URL，然后单击“分析图像”按钮以分析图像并查看结果。

### <a name="recognize-celebrities"></a>识别名人

计算机视觉的名人功能可分析图像中的名人。 分析完成后，名人功能返回一个 JSON 对象，它标识在图像中找到的名人。

要完成教程应用程序的名人功能，请执行以下步骤：

#### <a name="add-the-event-handler-code-for-the-celebrities-button"></a>添加名人按钮的事件处理程序代码

celebritiesImageButtonActionPerformed 事件处理程序方法将表单清空、显示 URL 中指定的图像，然后调用 CelebritiesImage 方法来分析图像。 当 CelebritiesImage 返回时，该方法会在“响应”文本区域中显示格式化的 JSON 响应，然后从 JSONObject 中提取第一个名人姓名并在窗口中显示该姓名以及正确识别名人的可信度。

将以下代码复制粘贴到 celebritiesImageButtonActionPerformed 方法。

> [!NOTE]
> NetBeans 不允许粘贴到方法定义行 (```private void```) 或该方法的右大括号。 要复制代码，请复制方法定义和右大括号之间的代码行，并将其粘贴到方法的内容中。

```java
    private void celebritiesImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL celebritiesImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        celebritiesImage.setIcon(new ImageIcon());
        celebritiesCaptionLabel.setText("");
        celebritiesResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            celebritiesImageUrl = new URL(celebritiesImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(celebritiesImageUrl);
            scaleAndShowImage(bImage, celebritiesImage);
        } catch(IOException e) {
            celebritiesResponseTextArea.setText("Error loading Celebrity image: " + e.getMessage());
            return;
        }
        
        // Identify the celebrities in the image.
        JSONObject jsonObj = CelebritiesImage(celebritiesImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        celebritiesResponseTextArea.setText(jsonObj.toString(2));
                
        // Extract the text and confidence from the first caption in the description object.
        if (jsonObj.has("result") && jsonObj.getJSONObject("result").has("celebrities")) {

            JSONObject jsonCaption = jsonObj.getJSONObject("result").getJSONArray("celebrities").getJSONObject(0);
            
            if (jsonCaption.has("name") && jsonCaption.has("confidence")) {

                celebritiesCaptionLabel.setText("Caption: " + jsonCaption.getString("name") + 
                        " (confidence: " + jsonCaption.getDouble("confidence") + ").");
            }
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>添加用于 REST API 调用的包装器

CelebritiesImage 方法包装 REST API 调用以分析图像。 该方法返回描述图像中找到的名人的 JSONObject，或者如果出现错误，则返回 null。

复制 CelebritiesImage 方法并将其粘贴到 celebritiesImageButtonActionPerformed 方法的正下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to identify celebrities in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject CelebritiesImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to identify celebrities in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseCelebrities;
            URIBuilder builder = new URIBuilder(uriString);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-celebrities-function"></a>运行名人函数

按 F6 运行应用程序。 在“订阅密钥”字段中填写订阅密钥，并验证确保“订阅区域中”使用的区域正确无误。 单击“名人”选项卡，输入名人图像的 URL，然后单击“分析图像”按钮以分析图像并查看结果。

### <a name="intelligently-generate-a-thumbnail"></a>智能生成缩略图

计算机视觉的缩略图功能可通过图像生成缩略图。 通过使用“智能裁剪”功能，缩略图功能将识别图像中的兴趣区域并集中于此区域创建缩略图，进而生成视觉上更美观的缩略图。

要完成教程应用程序的缩略图功能，请执行以下步骤：

#### <a name="add-the-event-handler-code-for-the-thumbnail-button"></a>添加缩略图按钮的事件处理程序代码

thumbnailImageButtonActionPerformed 事件处理程序方法将表单清空、显示 URL 中指定的图像，然后调用 getThumbnailImage 方法来创建缩略图。 当 getThumbnailImage 返回时，该方法会显示生成的缩略图。

将以下代码复制粘贴到 thumbnailImageButtonActionPerformed 方法。

> [!NOTE]
> NetBeans 不允许粘贴到方法定义行 (```private void```) 或该方法的右大括号。 要复制代码，请复制方法定义和右大括号之间的代码行，并将其粘贴到方法的内容中。

```java
    private void thumbnailImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL thumbnailImageUrl;
        JSONObject jsonError[] = new JSONObject[1];
        
        // Clear out the previous image, response, and thumbnail, if any.
        thumbnailSourceImage.setIcon(new ImageIcon());
        thumbnailResponseTextArea.setText("");
        thumbnailImage.setIcon(new ImageIcon());

        // Display the image specified in the text box.
        try {
            thumbnailImageUrl = new URL(thumbnailImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(thumbnailImageUrl);
            scaleAndShowImage(bImage, thumbnailSourceImage);
        } catch(IOException e) {
            thumbnailResponseTextArea.setText("Error loading image to thumbnail: " + e.getMessage());
            return;
        }
        
        // Get the thumbnail for the image.
        BufferedImage thumbnail = getThumbnailImage(thumbnailImageUrl.toString(), jsonError);
        
        // A non-null value indicates error.
        if (jsonError[0] != null) {
            // Format and display the JSON error.
            thumbnailResponseTextArea.setText(jsonError[0].toString(2));
            return;
        }
        
        // Display the thumbnail.
        if (thumbnail != null) {
            scaleAndShowImage(thumbnail, thumbnailImage);
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>添加用于 REST API 调用的包装器

getThumbnailImage 方法包装 REST API 调用以分析图像。 该方法返回包含缩略图的 BufferedImage，或者如果出现错误，则返回 null。 在 jsonError 字符串数组的第一个元素中将返回错误消息。

复制以下 getThumbnailImage 方法，并将其粘贴到 thumbnailImageButtonActionPerformed 方法的正下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to create a thumbnail for an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A BufferedImage containing the thumbnail, or null if a runtime error occurs. In the case
     * of an error, the error message will be returned in the first element of the jsonError string array.
     */
    private BufferedImage getThumbnailImage(String imageUrl, JSONObject[] jsonError) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to identify celebrities in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseThumbnail;
            URIBuilder uriBuilder = new URIBuilder(uriString);

            // Request parameters.
            uriBuilder.setParameter("width", "100");
            uriBuilder.setParameter("height", "150");
            uriBuilder.setParameter("smartCropping", "true");

            // Prepare the URI for the REST API call.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity requestEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(requestEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // Check for success.
            if (response.getStatusLine().getStatusCode() == 200)
            {
                // Return the thumbnail.
                return ImageIO.read(entity.getContent());
            }
            else
            {
                // Format and display the JSON error message.
                String jsonString = EntityUtils.toString(entity);
                jsonError[0] = new JSONObject(jsonString);
                return null;
            }
        }
        catch (Exception e)
        {
            String errorMessage = e.getMessage();
            System.out.println(errorMessage);
            jsonError[0] = new JSONObject(errorMessage);
            return null;
        }
    }
```

#### <a name="run-the-thumbnail-function"></a>运行缩略图函数

按 F6 运行应用程序。 在“订阅密钥”字段中填写订阅密钥，并验证确保“订阅区域中”使用的区域正确无误。 单击“缩略图”选项卡，输入图像 URL，然后单击“生成缩略图”按钮以分析图像并查看结果。

### <a name="read-printed-text-ocr"></a>读取印刷体文本 (OCR)

计算机视觉的光学字符识别 (OCR) 功能可分析图像中的印刷体文本。 分析完成后，OCR 返回一个 JSON 对象，它包含图像中的文本及其位置。

要完成教程应用程序的 OCR 功能，请执行以下步骤：

#### <a name="add-the-event-handler-code-for-the-ocr-button"></a>添加 OCR 按钮的事件处理程序代码

ocrImageButtonActionPerformed 事件处理程序方法将表单清空、显示 URL 中指定的图像，然后调用 OcrImage 方法来分析图像。 当 OcrImage 返回时，该方法会在“响应”文本区域中以格式化 JSON 的形式显示检测到的文本。

将以下代码复制粘贴到 ocrImageButtonActionPerformed 方法。

> [!NOTE]
> NetBeans 不允许粘贴到方法定义行 (```private void```) 或该方法的右大括号。 要复制代码，请复制方法定义和右大括号之间的代码行，并将其粘贴到方法的内容中。

```java
    private void ocrImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL ocrImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        ocrImage.setIcon(new ImageIcon());
        ocrResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            ocrImageUrl = new URL(ocrImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(ocrImageUrl);
            scaleAndShowImage(bImage, ocrImage);
        } catch(IOException e) {
            ocrResponseTextArea.setText("Error loading OCR image: " + e.getMessage());
            return;
        }
        
        // Read the text in the image.
        JSONObject jsonObj = OcrImage(ocrImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        ocrResponseTextArea.setText(jsonObj.toString(2));
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>添加用于 REST API 调用的包装器

OcrImage 方法包装 REST API 调用以分析图像。 该方法返回从调用返回的 JSON 数据的 JSONObject，或者如果出现错误，则返回 null。

复制以下 OcrImage 方法并将其粘贴到 ocrImageButtonActionPerformed 方法的正下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to read text in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject OcrImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to read text in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseOcr;
            URIBuilder uriBuilder = new URIBuilder(uriString);

            // Request parameters.
            uriBuilder.setParameter("language", "unk");
            uriBuilder.setParameter("detectOrientation ", "true");

            // Prepare the URI for the REST API call.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-ocr-function"></a>运行 OCR 函数

按 F6 运行应用程序。 在“订阅密钥”字段中填写订阅密钥，并验证确保“订阅区域中”使用的区域正确无误。 单击“OCR”选项卡，输入印刷体文本图像的 URL，然后单击“读取图像”按钮以分析图像并查看结果。

### <a name="read-handwritten-text-handwriting-recognition"></a>读取手写文本（识别手写体）

计算机视觉的手写体识别功能可分析图像中的手写文本。 分析完成后，手写体识别功能返回一个 JSON 对象，它包含图像中的文本及其位置。

要完成教程应用程序的手写体识别功能，请执行以下步骤：

#### <a name="add-the-event-handler-code-for-the-handwriting-button"></a>添加手写按钮的事件处理程序代码

handwritingImageButtonActionPerformed 事件处理程序方法将表单清空、显示 URL 中指定的图像，然后调用 HandwritingImage 方法来分析图像。 当 HandwritingImage 返回时，该方法会在“响应”文本区域中以格式化 JSON 的形式显示检测到的文本。

将以下代码复制粘贴到 handwritingImageButtonActionPerformed 方法。

> [!NOTE]
> NetBeans 不允许粘贴到方法定义行 (```private void```) 或该方法的右大括号。 要复制代码，请复制方法定义和右大括号之间的代码行，并将其粘贴到方法的内容中。

```java
    private void handwritingImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL handwritingImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        handwritingImage.setIcon(new ImageIcon());
        handwritingResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            handwritingImageUrl = new URL(handwritingImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(handwritingImageUrl);
            scaleAndShowImage(bImage, handwritingImage);
        } catch(IOException e) {
            handwritingResponseTextArea.setText("Error loading Handwriting image: " + e.getMessage());
            return;
        }
        
        // Read the text in the image.
        JSONObject jsonObj = HandwritingImage(handwritingImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        handwritingResponseTextArea.setText(jsonObj.toString(2));
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>添加用于 REST API 调用的包装器

HandwritingImage 方法包装分析图像时所需的两个 REST API 调用。 由于手写体识别非常耗时，因此分两步执行。 第一个调用提交图像进行分析；第二个调用检索完成处理时检测到的文本。

检索文本后，HandwritingImage 方法返回描述文本和文本位置的 JSONObject，或者如果出现错误，则返回 null。

复制以下 HandwritingImage 方法并将其粘贴到 handwritingImageButtonActionPerformed 方法的正下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to read handwritten text in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject HandwritingImage(String imageUrl) {
        try (CloseableHttpClient textClient = HttpClientBuilder.create().build();
             CloseableHttpClient resultClient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to read text in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseHandwriting;
            URIBuilder uriBuilder = new URIBuilder(uriString);
            
            // Request parameters.
            uriBuilder.setParameter("handwriting", "true");

            // Prepare the URI for the REST API call.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response.
            HttpResponse textResponse = textClient.execute(request);
            
            // Check for success.
            if (textResponse.getStatusLine().getStatusCode() != 202) {
                // An error occurred. Return the JSON error message.
                HttpEntity entity = textResponse.getEntity();
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            }
            
            String operationLocation = null;

            // The 'Operation-Location' in the response contains the URI to retrieve the recognized text.
            Header[] responseHeaders = textResponse.getAllHeaders();
            for(Header header : responseHeaders) {
                if(header.getName().equals("Operation-Location"))
                {
                    // This string is the URI where you can get the text recognition operation result.
                    operationLocation = header.getValue();
                    break;
                }
            }
            
            // NOTE: The response may not be immediately available. Handwriting recognition is an
            // async operation that can take a variable amount of time depending on the length
            // of the text you want to recognize. You may need to wait or retry this operation.
            //
            // This example checks once per second for ten seconds.
            
            JSONObject responseObj = null;
            int i = 0;
            do {
                // Wait one second.
                Thread.sleep(1000);
                
                // Check to see if the operation completed.
                HttpGet resultRequest = new HttpGet(operationLocation);
                resultRequest.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());
                HttpResponse resultResponse = resultClient.execute(resultRequest);
                HttpEntity responseEntity = resultResponse.getEntity();
                if (responseEntity != null)
                {
                    // Get the JSON response.
                    String jsonString = EntityUtils.toString(responseEntity);
                    responseObj = new JSONObject(jsonString);
                }
            }
            while (i < 10 && responseObj != null &&
                    !responseObj.getString("status").equalsIgnoreCase("Succeeded"));
            
            // If the operation completed, return the JSON object.
            if (responseObj != null) {
                return responseObj;
            } else {
                // Return null for timeout error.
                System.out.println("Timeout error.");
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-handwriting-function"></a>运行手写函数

按 F6 运行应用程序。 在“订阅密钥”字段中填写订阅密钥，并验证确保“订阅区域中”使用的区域正确无误。 单击“读取手写文本”选项卡，输入手写文本图像的 URL，然后单击“读取图像”按钮以分析图像并查看结果。

## <a name="next-steps"></a>后续步骤

在本指南中，您使用计算机视觉 REST API 与 Java 配合使用来测试的许多可用的图像分析功能。 接下来，请参阅若要了解有关 Api 的详细信息所涉及的参考文档。

- [计算机视觉 REST API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa)
