---
title: 快速入门：识别文本中的语言，Go - 文本翻译 API
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，你将使用文本翻译 API 和 Go 识别源文本的语言。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: quickstart
ms.date: 02/21/2019
ms.author: erhopf
ms.openlocfilehash: 60f677c994eb3a30f9dcfb4c341a242b46960c94
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64916612"
---
# <a name="quickstart-use-the-translator-text-api-to-detect-text-language-using-go"></a>快速入门：使用文本翻译 API 通过 Go 来检测文本语言

本快速入门介绍如何使用 Go 和文本翻译 REST API 来检测所提供文本的语言。

此快速入门需要包含文本翻译资源的 [Azure 认知服务帐户](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 如果没有帐户，可以使用[免费试用版](https://azure.microsoft.com/try/cognitive-services/)获取订阅密钥。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* [Go](https://golang.org/doc/install)
* 适用于文本翻译的 Azure 订阅密钥

## <a name="create-a-project-and-import-required-modules"></a>创建一个项目并导入必需的模块

使用最喜欢的 IDE 或编辑器创建新的 Go 项目。 然后，将此代码片段复制到项目的名为 `detect-language.go` 的文件中。

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "net/url"
    "os"
)
```

## <a name="create-the-main-function"></a>创建 main 函数

此示例将尝试从环境变量 `TRANSLATOR_TEXT_KEY` 读取文本翻译订阅密钥。 如果不熟悉环境变量，则可将 `subscriptionKey` 设置为字符串并注释掉条件语句。

将以下代码复制到项目中：

```go
func main() {
    /*
     * Read your subscription key from an env variable.
     * Please note: You can replace this code block with
     * var subscriptionKey = "YOUR_SUBSCRIPTION_KEY" if you don't
     * want to use env variables. If so, be sure to delete the "os" import.
     */
    subscriptionKey := os.Getenv("TRANSLATOR_TEXT_KEY")
    if subscriptionKey == "" {
       log.Fatal("Environment variable TRANSLATOR_TEXT_KEY is not set.")
    }
    /*
     * This calls our detect function, which we'll
     * create in the next section. It takes a single argument,
     * the subscription key.
     */
    detect(subscriptionKey)
}
```

## <a name="create-a-function-to-detect-the-text-language"></a>创建文本语言检测函数

创建文本语言检测函数。 此函数将接受一个参数，即文本翻译订阅密钥。

```go
func detect(subscriptionKey string) {
    /*  
     * In the next few sections, we'll add code to this
     * function to make a request and handle the response.
     */
}
```

接下来，构造 URL。 使用 `Parse()` 和 `Query()` 方法生成 URL。

将此代码复制到 `detect` 函数中。

```go
// Build the request URL. See: https://golang.org/pkg/net/url/#example_URL_Parse
u, _ := url.Parse("https://api.cognitive.microsofttranslator.com/detect?api-version=3.0")
q := u.Query()
u.RawQuery = q.Encode()
```

>[!NOTE]
> 有关终结点、路由和请求参数的详细信息，请参阅[文本翻译 API 3.0：检测](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-detect)。

## <a name="create-a-struct-for-your-request-body"></a>创建请求正文的结构

接下来，创建请求正文的匿名结构，并使用 `json.Marshal()` 将其编码为 JSON。 将此代码添加到 `detect` 函数中。

```go
// Create an anonymous struct for your request body and encode it to JSON
body := []struct {
    Text string
}{
    {Text: "Salve, Mondo!"},
}
b, _ := json.Marshal(body)
```

## <a name="build-the-request"></a>生成请求

将请求正文编码为 JSON 后，可以生成 POST 请求并调用文本翻译 API。

```go
// Build the HTTP POST request
req, err := http.NewRequest("POST", u.String(), bytes.NewBuffer(b))
if err != nil {
    log.Fatal(err)
}
// Add required headers
req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
req.Header.Add("Content-Type", "application/json")

// Call the Translator Text API
res, err := http.DefaultClient.Do(req)
if err != nil {
    log.Fatal(err)
}
```

## <a name="handle-and-print-the-response"></a>处理并输出响应

将此代码添加到 `detect` 函数以解码 JSON 响应，然后格式化并输出结果。

```go
// Decode the JSON response
var result interface{}
if err := json.NewDecoder(res.Body).Decode(&result); err != nil {
    log.Fatal(err)
}
// Format and print the response to terminal
prettyJSON, _ := json.MarshalIndent(result, "", "  ")
fmt.Printf("%s\n", prettyJSON)
```

## <a name="put-it-all-together"></a>将其放在一起

就是这样，你已构建了一个简单的程序。该程序可以调用文本翻译 API 并返回 JSON 响应。 现在，可以运行该程序了：

```console
go run detect-language.go
```

如果希望将你的代码与我们的进行比较，请查看 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Go) 上提供的完整示例。

## <a name="sample-response"></a>示例响应

请在此[语言列表](https://docs.microsoft.com/azure/cognitive-services/translator/language-support)中查找国家/地区缩写。

```json
[
  {
    "alternatives": [
      {
        "isTranslationSupported": true,
        "isTransliterationSupported": false,
        "language": "pt",
        "score": 1
      },
      {
        "isTranslationSupported": true,
        "isTransliterationSupported": false,
        "language": "en",
        "score": 1
      }
    ],
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "language": "it",
    "score": 1
  }
]
```

## <a name="next-steps"></a>后续步骤

从 GitHub 上的 [Azure SDK for Go](https://github.com/Azure/azure-sdk-for-go) 中浏览认知服务 API 的 Go 程序包。

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 Go 程序包](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices)

## <a name="see-also"></a>另请参阅

了解如何使用文本翻译 API 执行以下操作：

* [翻译文本](quickstart-go-translate.md)
* [直译文本](quickstart-go-transliterate.md)
* [获取备用翻译](quickstart-go-dictionary.md)
* [获取支持的语言的列表](quickstart-go-languages.md)
* [根据输入确定句子长度](quickstart-go-sentences.md)
