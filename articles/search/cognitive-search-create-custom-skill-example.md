---
title: 示例：在认知搜索管道中创建自定义技能 - Azure搜索
description: 演示如何使用映射到 Azure 搜索中认知搜索索引管道的自定义技能中的文本翻译 API。
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 82d49a6a82251f440c06db03edc92851fce87741
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023611"
---
# <a name="example-create-a-custom-skill-using-the-text-translate-api"></a>示例：使用文本翻译 API 创建自定义技能

在此示例中，学习如何创建 Web API 自定义技能，可接受任何语言的文本并将其翻译为英语。 该示例使用 [Azure Function](https://azure.microsoft.com/services/functions/) 来包装[翻译文本 API](https://azure.microsoft.com/services/cognitive-services/translator-text-api/)，以便实现自定义技能接口。

## <a name="prerequisites"></a>必备组件

+ 如果不熟悉自定义技能应实现的输入/输出接口，请阅读[自定义技能接口](cognitive-search-custom-skill-interface.md)一文。

+ [注册文本翻译 API](../cognitive-services/translator/translator-text-how-to-signup.md)，并获取 API 密钥以使用它。

+ 安装 [Visual Studio 2017 版本 15.5](https://www.visualstudio.com/vs/) 或更高版本，包括 Azure 开发工作负荷。

## <a name="create-an-azure-function"></a>创建 Azure 函数

尽管此示例使用 Azure Function 来托管 Web API，但并非必须如此。  只要满足[认知技能的接口需求](cognitive-search-custom-skill-interface.md)，采用的方法并不重要。 但是，可通过 Azure Functions 轻松创建自定义技能。

### <a name="create-a-function-app"></a>创建函数应用

1. 在 Visual Studio 中，从“文件”菜单中选择“新建” > “项目”。

1. 在“新建项目”对话框中，选择“已安装”，展开“Visual C#” > “云”，选择“Azure Functions”，键入项目的名称，然后选择“确定”。 函数应用名称必须可以充当 C# 命名空间，因此请勿使用下划线、连字符或任何其他的非字母数字字符。

1. 选择**Azure Functions v2 (.NET Core)**。 也可以使用版本 1 执行此操作，但下面的代码基于 v2 模板编写。

1. 选择“HTTP 触发器”作为类型

1. 对于存储帐户，可选择“无”，因为此函数不需要任何存储。

1. 选择“确定”以创建函数项目和 HTTP 触发的函数。

### <a name="modify-the-code-to-call-the-translate-cognitive-service"></a>修改代码以调用翻译认知服务

Visual Studio 将创建一个项目，并在该项目中创建一个包含所选函数类型的样本代码的类。 方法中的 *FunctionName* 属性设置函数的名称。 *HttpTrigger* 属性指定该函数将由某个 HTTP 请求触发。

现在，将文件 Function1.cs 的所有内容替换为以下代码：

```csharp
using System;
using System.Net.Http;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.IO;
using System.Text;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Azure.WebJobs.Host;
using Newtonsoft.Json;

namespace TranslateFunction
{
    // This function will simply translate messages sent to it.
    public static class Function1
    {
        static string path = "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0";

        // NOTE: Replace this example key with a valid subscription key.
        static string key = "<enter your api key here>";

        #region classes used to serialize the response
        private class WebApiResponseError
        {
            public string message { get; set; }
        }

        private class WebApiResponseWarning
        {
            public string message { get; set; }
        }

        private class WebApiResponseRecord
        {
            public string recordId { get; set; }
            public Dictionary<string, object> data { get; set; }
            public List<WebApiResponseError> errors { get; set; }
            public List<WebApiResponseWarning> warnings { get; set; }
        }

        private class WebApiEnricherResponse
        {
            public List<WebApiResponseRecord> values { get; set; }
        }
        #endregion

        [FunctionName("Translate")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)]HttpRequest req,
            TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            string recordId = null;
            string originalText = null;
            string toLanguage = null;
            string translatedText = null;

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            // Validation
            if (data?.values == null)
            {
                return new BadRequestObjectResult(" Could not find values array");
            }
            if (data?.values.HasValues == false || data?.values.First.HasValues == false)
            {
                // It could not find a record, then return empty values array.
                return new BadRequestObjectResult(" Could not find valid records in values array");
            }

            recordId = data?.values?.First?.recordId?.Value as string;
            originalText = data?.values?.First?.data?.text?.Value as string;
            toLanguage = data?.values?.First?.data?.language?.Value as string;

            if (recordId == null)
            {
                return new BadRequestObjectResult("recordId cannot be null");
            }

            translatedText = TranslateText(originalText, toLanguage).Result;
        
            // Put together response.
            WebApiResponseRecord responseRecord = new WebApiResponseRecord();
            responseRecord.data = new Dictionary<string, object>();
            responseRecord.recordId = recordId;
            responseRecord.data.Add("text", translatedText);

            WebApiEnricherResponse response = new WebApiEnricherResponse();
            response.values = new List<WebApiResponseRecord>();
            response.values.Add(responseRecord);

            return (ActionResult)new OkObjectResult(response);
        }


        /// <summary>
        /// Use Cognitive Service to translate text from one language to another.
        /// </summary>
        /// <param name="originalText">The text to translate.</param>
        /// <param name="toLanguage">The language you want to translate to.</param>
        /// <returns>Asynchronous task that returns the translated text. </returns>
        async static Task<string> TranslateText(string originalText, string toLanguage)
        {
            System.Object[] body = new System.Object[] { new { Text = originalText } };
            var requestBody = JsonConvert.SerializeObject(body);

            var uri = $"{path}&to={toLanguage}";

            string result = "";

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(uri);
                request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
                request.Headers.Add("Ocp-Apim-Subscription-Key", key);

                var response = await client.SendAsync(request);
                var responseBody = await response.Content.ReadAsStringAsync();

                dynamic data = JsonConvert.DeserializeObject(responseBody);
                result = data?.First?.translations?.First?.text?.Value as string;

            }
            return result;
        }
    }
}
```

请确保根据注册翻译文本 API 时获得的密钥，在 TranslateText 方法中输入你自己的密钥值。

此示例是一个简单的扩充器，一次仅适用于一条记录。 稍后设置技能组合的批大小时，此事实则显得重要。

## <a name="test-the-function-from-visual-studio"></a>从 Visual Studio 中测试函数

按 F5 运行程序并测试函数行为。 在此示例中，我们将使用下面的函数将西班牙语文本翻译成英语。 使用 Postman 或 Fiddler 发出如下所示的调用：

```http
POST https://localhost:7071/api/Translate
```
### <a name="request-body"></a>请求正文
```json
{
   "values": [
        {
            "recordId": "a1",
            "data":
            {
               "text":  "Este es un contrato en Inglés",
               "language": "en"
            }
        }
   ]
}
```
### <a name="response"></a>响应
你应该看到类似于以下示例的响应：

```json
{
    "values": [
        {
            "recordId": "a1",
            "data": {
                "text": "This is a contract in English"
            },
            "errors": null,
            "warnings": null
        }
    ]
}
```

## <a name="publish-the-function-to-azure"></a>将函数发布到 Azure

当你对函数行为满意时，可将其发布。

1. **在“解决方案资源管理器”** 中，右键单击该项目并选择“发布”。 选择“新建” > “发布”。

1. 如果尚未将 Visual Studio 连接到 Azure 帐户，请选择“添加帐户...”

1. 请按照屏幕上的提示操作。 系统会要求你指定要使用的 Azure 帐户、资源组、托管计划和存储帐户。 如果尚不拥有这些资源，可创建新资源组、新托管计划和存储帐户。 完成后，选择“创建”

1. 部署完成后，请注意，该网站的 URL。 这是 Azure 中你的函数应用的地址。 

1. 在 [Azure 门户](https://portal.azure.com)中，导航到资源组，然后查找发布的翻译函数。 在“管理”部分下，应可看到主机密钥。 对默认主机密钥选择“复制”图标。  

## <a name="test-the-function-in-azure"></a>在 Azure 中测试函数

现在有了默认主机密钥，按以下方式测试函数：

```http
POST https://translatecogsrch.azurewebsites.net/api/Translate?code=[enter default host key here]
```
### <a name="request-body"></a>请求正文
```json
{
   "values": [
        {
            "recordId": "a1",
            "data":
            {
               "text":  "Este es un contrato en Inglés",
               "language": "en"
            }
        }
   ]
}
```

此操作产生的结果与之前在本地环境中运行函数时看到的结果类似。

## <a name="connect-to-your-pipeline"></a>连接到管道
现在有了新的自定义技能，可将其添加到技能组合。 以下示例显示如何调用技能。 由于该技能不处理批，因此添加一个说明，指示最大批大小仅为 ```1```，从而一次发送一个文档。

```json
{
    "skills": [
      ...,  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new translator custom skill",
        "uri": "https://translatecogsrch.azurewebsites.net/api/Translate?code=[enter default host key here]",
        "batchSize":1,
        "context": "/document",
        "inputs": [
          {
            "name": "text",
            "source": "/document/content"
          },
          {
            "name": "language",
            "source": "/document/destinationLanguage"
          }
        ],
        "outputs": [
          {
            "name": "text",
            "targetName": "translatedText"
          }
        ]
      }
  ]
}
```

## <a name="next-steps"></a>后续步骤
祝贺你！ 已创建第一个自定义扩充器。 现在，可按照相同的模式添加自己的自定义功能。 

+ [将自定义技能添加到认知搜索管道](cognitive-search-custom-skill-interface.md)
+ [如何定义技能组合](cognitive-search-defining-skillset.md)
+ [创建技能组合 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [如何映射扩充的域](cognitive-search-output-field-mapping.md)
