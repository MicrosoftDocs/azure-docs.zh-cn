---
title: 快速入门：识别数字墨迹的墨迹识别器 REST API 和 Node.js
description: 使用手写识别器 API 开始认识到数字墨迹笔画。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: article
ms.date: 05/02/2019
ms.author: aahi
ms.openlocfilehash: 651474fd538123e760022ac59efbbaf0b9b83d70
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2019
ms.locfileid: "65519679"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-javascript"></a>快速入门：识别使用手写识别器 REST API 和 JavaScript 数字墨迹

使用本快速入门教程开始使用手写识别器 API 上数字墨迹笔画。 此 JavaScript 应用程序发送包含 JSON 格式的墨迹笔划数据的 API 请求，并显示响应。

此应用程序以 Javascript 编写，并在 web 浏览器中运行，而 API 是 RESTful web 服务与大多数编程语言兼容。

通常你会从数字墨迹书写应用调用 API。 本快速入门将下面的手写示例的墨迹笔划数据发送从 JSON 文件。

![手写文本图像的](../media/handwriting-sample.jpg)

可以在 [GitHub](https://go.microsoft.com/fwlink/?linkid=2089905) 上找到此快速入门的源代码。

## <a name="prerequisites"></a>必备组件

- Web 浏览器
- 本快速入门示例墨迹笔划数据都位于[GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/javascript/InkRecognition/quickstart/example-ink-strokes.json)。


[!INCLUDE [cognitive-services-ink-recognizer-signup-requirements](../../../../includes/cognitive-services-ink-recognizer-signup-requirements.md)]

## <a name="create-a-new-application"></a>创建新应用程序

1. 在你最喜欢的 IDE 或编辑器，创建一个新`.html`文件。 然后将基本 HTML 添加到它，稍后我们将添加的代码。
    
    ```html
    <!DOCTYPE html>
    <html>
    
        <head>
            <script type="text/javascript">
            </script>
        </head>
        
        <body>
        </body>
    
    </html>
    ```

2. 在`<body>`标记中，添加以下 html:
    1. 用于显示 JSON 请求和响应的两个文本区域。
    2. 调用一个按钮`recognizeInk()`将更高版本创建的函数。
    
    ```HTML
    <!-- <body>-->
        <h2>Send a request to the Ink Recognition API</h2>
        <p>Request:</p>
        <textarea id="request" style="width:800px;height:300px"></textarea>
        <p>Response:</p>
        <textarea id="response" style="width:800px;height:300px"></textarea>
        <br>
        <button type="button" onclick="recognizeInk()">Recognize Ink</button>
    <!--</body>-->
    ```

## <a name="load-the-example-json-data"></a>加载示例 JSON 数据

1. 在`<script>`标记中，为 sampleJson 创建变量。 然后，创建一个名为的 JavaScript 函数`openFile()`这将打开文件资源管理器，这样您就可以选择 JSON 文件。 当`Recognize ink`单击按钮时，它将调用此函数，并开始读取文件。
2. 使用`FileReader`对象的`onload()`函数来以异步方式处理该文件。 
    1. 替换为任何`\n`或`\r`用空字符串文件中的字符。 
    2. 使用`JSON.parse()`将文本转换为有效的 JSON
    3. 更新`request`应用程序中的文本框。 使用`JSON.stringify()`要设置格式的 JSON 字符串。 
    
    ```javascript
    var sampleJson = "";
    function openFile(event) {
        var input = event.target;
    
        var reader = new FileReader();
        reader.onload = function(){
            sampleJson = reader.result.replace(/(\\r\\n|\\n|\\r)/gm, "");
            sampleJson = JSON.parse(sampleJson);
            document.getElementById('request').innerHTML = JSON.stringify(sampleJson, null, 2);
        };
        reader.readAsText(input.files[0]);
    };
    ```

## <a name="send-a-request-to-the-ink-recognizer-api"></a>将请求发送到墨迹识别器 API

1. 内`<script>`标记中，创建一个名为函数`recognizeInk()`。 更高版本，此函数将调用 API，并使用响应更新页面。 此函数中的以下步骤从添加的代码。 
        
    ```javascript
    function recognizeInk() {
    // add the code from the below steps here 
    }
    ```

    1. 创建终结点 URL、 订阅密钥和示例 JSON 的变量。 然后创建`XMLHttpRequest`发送 API 请求的对象。 
        
        ```javascript
        // Replace the below URL with the correct one for your subscription. 
        // Your endpoint can be found in the Azure portal. For example: https://westus2.api.cognitive.microsoft.com
        var SERVER_ADDRESS = "YOUR-SUBSCRIPTION-URL";
        var ENDPOINT_URL = SERVER_ADDRESS + "/inkrecognizer/v1.0-preview/recognize";
        // Replace the subscriptionKey string value with your valid subscription key.
        var SUBSCRIPTION_KEY = "YOUR-SUBSCRIPTION-KEY";
        var xhttp = new XMLHttpRequest();
        ```
    2. 创建用于返回的函数`XMLHttpRequest`对象。 此函数将分析 API 响应从成功的请求，并显示在应用程序中。 
            
        ```javascript
        function returnFunction(xhttp) {
            var response = JSON.parse(xhttp.responseText);
            console.log("Response: %s ", response);
            document.getElementById('response').innerHTML = JSON.stringify(response, null, 2);
        }
        ```
    3. 创建请求对象的错误函数。 此函数将错误记录到控制台。 
            
        ```javascript
        function errorFunction() {
            console.log("Error: %s, Detail: %s", xhttp.status, xhttp.responseText);
        }
        ```

    4. 创建请求对象的函数`onreadystatechange`属性。 请求对象的就绪状态更改时，将应用更高版本的返回和 error 函数。
            
        ```javascript
        xhttp.onreadystatechange = function () {
            if (this.readyState === 4) {
                if (this.status === 200) {
                    returnFunction(xhttp);
                } else {
                    errorFunction(xhttp);
                }
            }
        };
        ```
    
    5. 发送 API 请求。 添加到你的订阅密钥`Ocp-Apim-Subscription-Key`标头，并设置`content-type`到 `application/json`
    
        ```javascript
        xhttp.open("PUT", ENDPOINT_URL, true);
        xhttp.setRequestHeader("Ocp-Apim-Subscription-Key", SUBSCRIPTION_KEY);
        xhttp.setRequestHeader("content-type", "application/json");
        xhttp.send(JSON.stringify(sampleJson));
        };

## Run the application and view the response

This application can be run within your web browser. A successful response is returned in JSON format. You can also find the JSON response on [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/javascript/InkRecognition/quickstart/example-response.json):

## Next steps

> [!div class="nextstepaction"]
> [REST API reference](https://go.microsoft.com/fwlink/?linkid=2089907)

To see how the Ink Recognition API works in a digital inking app, take a look at the following sample applications on GitHub:
* [C# and Universal Windows Platform(UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# and Windows Presentation Foundation(WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Javascript web-browser app](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Java and Android mobile app](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Swift and iOS mobile app](https://go.microsoft.com/fwlink/?linkid=2089805)
