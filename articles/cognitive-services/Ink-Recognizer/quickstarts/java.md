---
title: 快速入门：识别墨迹识别器 REST API 和 Java 与数字墨迹
description: 使用手写识别器 API 开始认识到数字墨迹笔画。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: article
ms.date: 05/02/2019
ms.author: aahi
ms.openlocfilehash: 6237253922544dc47bb11aec4dd58139f99eb0da
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518613"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-java"></a>快速入门：识别墨迹识别器 REST API 和 Java 与数字墨迹

使用本快速入门教程开始使用手写识别器 API 上数字墨迹笔画。 此 Java 应用程序将发送包含 JSON 格式的墨迹笔划数据的 API 请求，并获取响应。

虽然此应用程序是使用 Java 编写的，但 API 是一种 RESTful Web 服务，与大多数编程语言兼容。

通常你会从数字墨迹书写应用调用 API。 本快速入门将下面的手写示例的墨迹笔划数据发送从 JSON 文件。

![手写文本图像的](../media/handwriting-sample.jpg)

可以在 [GitHub](https://go.microsoft.com/fwlink/?linkid=2089904) 上找到此快速入门的源代码。

## <a name="prerequisites"></a>必备组件

- [Java&trade;开发 Kit(JDK) 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)或更高版本。

- 从 Maven 存储库导入这些库
    - [在 Java 中的 JSON](https://mvnrepository.com/artifact/org.json/json)包
    - [Apache HttpClient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)包

- 本快速入门示例墨迹笔划数据都位于[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/InkRecognition/quickstart/example-ink-strokes.json)。

[!INCLUDE [cognitive-services-ink-recognizer-signup-requirements](../../../../includes/cognitive-services-ink-recognizer-signup-requirements.md)]

## <a name="create-a-new-application"></a>创建新应用程序

1. 在你最喜欢的 IDE 或编辑器中新建一个 Java 项目，并导入以下库。

    ```java
    import org.apache.http.HttpEntity;
    import org.apache.http.client.methods.CloseableHttpResponse;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.entity.StringEntity;
    import org.apache.http.impl.client.CloseableHttpClient;
    import org.apache.http.impl.client.HttpClients;
    import org.apache.http.util.EntityUtils;
    import java.io.IOException;
    import java.nio.file.Files;
    import java.nio.file.Paths;
    ```

2. 为你的订阅密钥和你的终结点创建变量。 下面是可用于手写识别的 URI。 它将追加到你的服务终结点更高版本，若要创建的 API 请求 URL。

    ```java
    // Replace the subscriptionKey string value with your valid subscription key.
    static final String subscriptionKey = "YOUR_SUBSCRIPTION_KEY";
    // Replace the dataPath string with a path to the JSON formatted ink stroke data file.
    static final String dataPath = "PATH_TO_INK_STROKE_DATA";
    
    static final String endpoint = "https://api.cognitive.microsoft.com";
    static final String inkRecognitionUrl = "/inkrecognizer/v1.0-preview/recognize";
    ```

## <a name="create-a-function-to-send-requests"></a>创建一个函数来发送请求

1. 创建一个名为的新函数`sendRequest()`采用上面创建的变量。 然后，执行以下步骤。

2. 创建`CloseableHttpClient`可以向 API 发送请求的对象。 将请求发送到`HttpPut`通过合并你的终结点和手写识别器 URL 的请求对象。

3. 使用请求的`setHeader()`函数来设置`Content-Type`标头`application/json`，并添加你的订阅密钥的`Ocp-Apim-Subscription-Key`标头。

4. 使用请求的`setEntity()`要发送的数据的函数。   

5. 使用客户端`execute()`函数发送请求，并将其保存到`CloseableHttpResponse`对象。 

6. 创建`HttpEntity`对象来存储响应内容。 获取与内容`getEntity()`。 如果响应不为空，则将其返回。
    
    ```java
    static String sendRequest(String apiAddress, String endpoint, String subscriptionKey, String requestData) {
        try (CloseableHttpClient client = HttpClients.createDefault()) {
            HttpPut request = new HttpPut(endpoint + apiAddress);
            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
            request.setEntity(new StringEntity(requestData));
            try (CloseableHttpResponse response = client.execute(request)) {
                HttpEntity respEntity = response.getEntity();
                if (respEntity != null) {
                    return EntityUtils.toString(respEntity, "utf-8");
                }
            } catch (Exception respEx) {
                respEx.printStackTrace();
            }
        } catch (IOException ex) {
            System.err.println("Exception recognizing ink: " + ex.getMessage());
            ex.printStackTrace();
        }
        return null;
    }
    ```

## <a name="send-an-ink-recognition-request"></a>发送墨迹识别请求

创建一个名为方法`recognizeInk()`识别墨迹笔划数据。 调用`sendRequest()`上面创建的终结点、 url、 订阅密钥和 json 数据的方法。 获取结果，并将其打印到控制台。

```java
static void recognizeInk(String requestData) {
    System.out.println("Sending an ink recognition request");
    String result = sendRequest(inkRecognitionUrl, endpoint, subscriptionKey, requestData);
    System.out.println(result);
}
```

## <a name="load-your-digital-ink-data-and-send-the-request"></a>加载数字墨迹数据并发送请求

1. 在您的应用程序，在包含将添加到请求的数据的 JSON 文件中读取的主要方法。

2. 在上面创建的墨迹识别函数调用。
    
    ```java
    public static void main(String[] args) throws Exception {
        String requestData = new String(Files.readAllBytes(Paths.get(dataPath)), "UTF-8");
        recognizeInk(requestData);
    }
    ```

## <a name="run-the-application-and-view-the-response"></a>运行应用程序并查看响应

运行应用程序。 JSON 格式返回成功的响应。 您还可以在找到 JSON 响应[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/java/InkRecognition/quickstart/example-response.json)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [REST API 参考](https://go.microsoft.com/fwlink/?linkid=2089907)


若要了解墨迹识别 API 在数字墨迹应用中的工作原理，请查看 GitHub 上的以下示例应用程序：
* [C# 和通用 Windows 平台 (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# 和 Windows Presentation Foundation(WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Javascript Web 浏览器应用](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Java 和 Android 移动应用](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Swift 和 iOS 移动应用](https://go.microsoft.com/fwlink/?linkid=2089805)
