---
title: 快速入门：识别数字墨迹的墨迹识别器 REST API 和C#
description: 使用手写识别器 API 开始认识到数字墨迹笔画。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: article
ms.date: 05/02/2019
ms.author: aahi
ms.openlocfilehash: 9bb9c23cc1f807cae1d0d22f1652e8f4408f1f91
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518674"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-c"></a>快速入门：识别数字墨迹的墨迹识别器 REST API 和C#

使用此快速入门，开始将数字墨迹笔画发送到墨迹识别器 API。 此C#应用程序将发送包含 JSON 格式的墨迹笔划数据的 API 请求并获取响应。

虽然此应用程序是使用 C# 编写的，但 API 是一种 RESTful Web 服务，与大多数编程语言兼容。

通常你会从数字墨迹书写应用调用 API。 本快速入门将下面的手写示例的墨迹笔划数据发送从 JSON 文件。

![手写文本图像的](../media/handwriting-sample.jpg)

可以在 [GitHub](https://go.microsoft.com/fwlink/?linkid=2089502) 上找到此快速入门的源代码。

## <a name="prerequisites"></a>必备组件

- 任何版本的 [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)。
- [Newtonsoft.Json](https://www.newtonsoft.com/json)
    - 若要安装 Newtonsoft.Json NuGet 包在 Visual studio 中的形式：
        1. 右键单击**解决方案管理器**
        2. 单击“管理 NuGet 包”。
        3. 搜索`Newtonsoft.Json`和安装包
- 如果使用的 Linux/MacOS，可以使用运行此应用程序[Mono](https://www.mono-project.com/)。

- 本快速入门示例墨迹笔划数据都位于[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/InkRecognition/quickstart/example-ink-strokes.json)。

[!INCLUDE [cognitive-services-ink-recognizer-signup-requirements](../../../../includes/cognitive-services-ink-recognizer-signup-requirements.md)]


## <a name="create-a-new-application"></a>创建新应用程序

1. 在 Visual Studio 中，创建新的控制台解决方案并添加以下包。 

    ```csharp
    using System;
    using System.IO;
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    ```

2. 为你的订阅密钥和你的终结点创建变量。 下面是可用于手写识别的 URI。 它将追加到你的服务终结点更高版本才能创建 API 请求 URl。

    ```csharp
    // Replace the subscriptionKey string with your valid subscription key.
    const string subscriptionKey = "YOUR_SUBSCRIPTION_KEY";

    // Replace the dataPath string with a path to the JSON formatted ink stroke data.
    const string dataPath = @"PATH-TO-INK-STROKE-DATA"; 

    // URI information for ink recognition:
    const string endpoint = "https://api.cognitive.microsoft.com";
    const string inkRecognitionUrl = "/inkrecognizer/v1.0-preview/recognize";
    ```

## <a name="create-a-function-to-send-requests"></a>创建一个函数来发送请求

1. 创建一个名为的新异步函数`Request`采用上面创建的变量。

2. 设置客户端的安全协议和标头信息使用`HttpClient`对象。 请确保添加到你的订阅密钥`Ocp-Apim-Subscription-Key`标头。 然后创建`StringContent`请求对象。
 
3. 发送包含请求`PutAsync()`。 如果请求成功，返回的响应。  
    
    ```csharp
    static async Task<string> Request(string apiAddress, string endpoint, string subscriptionKey, string requestData){
        
        using (HttpClient client = new HttpClient { BaseAddress = new Uri(apiAddress) }){
            System.Net.ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12 | SecurityProtocolType.Tls11 | SecurityProtocolType.Tls;
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

            var content = new StringContent(requestData, Encoding.UTF8, "application/json");
            var res = await client.PutAsync(endpoint, content);
            if (res.IsSuccessStatusCode){
                return await res.Content.ReadAsStringAsync();
            }
            else{
                return $"ErrorCode: {res.StatusCode}";
            }
        }
    }
    ```

## <a name="send-an-ink-recognition-request"></a>发送墨迹识别请求

1. 创建一个名为的新函数`recognizeInk()`。 构造请求并将其发送通过调用`Request()`函数与终结点，订阅密钥 API 和数字墨迹笔划数据的 URL。

2. 反序列化 JSON 对象，并将其写入到控制台。 
    
    ```csharp
    static void recognizeInk(string requestData){

        //construct the request
        var result = Request(
            endpoint,
            inkRecognitionUrl,
            subscriptionKey,
            requestData).Result;

        dynamic jsonObj = Newtonsoft.Json.JsonConvert.DeserializeObject(result);
        System.Console.WriteLine(jsonObj);
    }
    ```

## <a name="load-your-digital-ink-data"></a>数字墨迹数据加载

创建一个名为函数`LoadJson()`加载墨迹数据的 JSON 文件。 使用`StreamReader`并`JsonTextReader`若要创建`JObject`并将其返回。
    
```csharp
public static JObject LoadJson(string fileLocation){

    var jsonObj = new JObject();

    using (StreamReader file = File.OpenText(fileLocation))
    using (JsonTextReader reader = new JsonTextReader(file)){
        jsonObj = (JObject)JToken.ReadFrom(reader);
    }
    return jsonObj;
}
```

## <a name="send-the-api-request"></a>发送 API 请求

1. 在应用程序的 main 方法中，将 JSON 数据加载使用上面创建的函数。 

2. 调用`recognizeInk()`上面创建的函数。 使用`System.Console.ReadKey()`后运行应用程序保留在控制台窗口打开。
    
    ```csharp
    static void Main(string[] args){

        var requestData = LoadJson(dataPath);
        string requestString = requestData.ToString(Newtonsoft.Json.Formatting.None);
        recognizeInk(requestString);
        System.Console.WriteLine("\nPress any key to exit ");
        System.Console.ReadKey();
        }
    ```

## <a name="run-the-application-and-view-the-response"></a>运行应用程序并查看响应

运行应用程序。 JSON 格式返回成功的响应。 您还可以在找到 JSON 响应[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/InkRecognition/quickstart/example-response.json)。


## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [REST API 参考](https://go.microsoft.com/fwlink/?linkid=2089907)


若要了解墨迹识别 API 在数字墨迹应用中的工作原理，请查看 GitHub 上的以下示例应用程序：
* [C# 和通用 Windows 平台 (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# 和 Windows Presentation Foundation(WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Javascript Web 浏览器应用](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Java 和 Android 移动应用](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Swift 和 iOS 移动应用](https://go.microsoft.com/fwlink/?linkid=2089805)
