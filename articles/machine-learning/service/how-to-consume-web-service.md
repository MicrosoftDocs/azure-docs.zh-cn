---
title: 创建客户端以使用部署的 Web 服务
titleSuffix: Azure Machine Learning service
description: 了解如何使用在通过 Azure 机器学习模型部署模型时生成的 Web 服务。 该 Web 服务公开一个 REST API。 使用所选的编程语言为此 API 创建客户端。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 12/03/2018
ms.custom: seodec18
ms.openlocfilehash: 4aaf56a00669dbf7e312e2a6f8787ccc664d1190
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023819"
---
# <a name="consume-an-azure-machine-learning-model-deployed-as-a-web-service"></a>使用部署为 Web 服务的 Azure 机器学习模型

将 Azure 机器学习模型部署为 Web 服务可创建 REST API。 可将数据发送到此 API，并接收模型返回的预测。 本文档介绍了如何使用 C#、Go、Java 和 Python 为 Web 服务创建客户端。

将映像部署到 Azure 容器实例、 Azure Kubernetes 服务或现场可编程门阵列 (FPGA) 时创建的 web 服务。 你将基于已注册的模型和评分文件创建映像。 你将使用 [Azure 机器学习 SDK](https://aka.ms/aml-sdk) 检索用来访问 Web 服务的 URI。 如果启用了身份验证，则还可以使用该 SDK 来获取身份验证密钥。

用于创建使用机器学习 Web 服务的客户端的常规工作流为：

1. 使用 SDK 获取连接信息。
1. 确定模型使用的请求数据的类型。
1. 创建调用 Web 服务的应用程序。

## <a name="connection-information"></a>连接信息

> [!NOTE]
> 使用 Azure 机器学习 SDK 获取 Web 服务信息。 这是一个 Python SDK。 可以使用任何语言来为服务创建客户端。

[azureml.core.Webservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py) 类提供了创建客户端所需的信息。 创建客户端应用程序时，以下 `Webservice` 属性非常有用：

* `auth_enabled` - 如果启用了身份验证，则为 `True`；否则为 `False`。
* `scoring_uri` - REST API 地址。

可通过三种方式检索已部署的 Web 服务的此信息：

* 部署模型时，会返回包含有关服务的信息的 `Webservice` 对象：

    ```python
    service = Webservice.deploy_from_model(name='myservice',
                                           deployment_config=myconfig,
                                           models=[model],
                                           image_config=image_config,
                                           workspace=ws)
    print(service.scoring_uri)
    ```

* 可以使用 `Webservice.list` 检索工作区中为模型部署的 Web 服务列表。 可以添加筛选器，以缩小返回的信息列表范围。 有关可以筛选的对象的详细信息，请参阅 [Webservice.list](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.webservice.webservice?view=azure-ml-py) 参考文档。

    ```python
    services = Webservice.list(ws)
    print(services[0].scoring_uri)
    ```

* 如果知道已部署的服务的名称，可以创建 `Webservice` 的新实例，并以参数的形式提供工作区和服务名称。 新对象包含有关已部署的服务的信息。

    ```python
    service = Webservice(workspace=ws, name='myservice')
    print(service.scoring_uri)
    ```

### <a name="authentication-key"></a>身份验证密钥

为部署启用身份验证时，会自动创建身份验证密钥。

* 部署到 Azure Kubernetes 服务时，会默认启用身份验证。
* 部署到 Azure 容器实例时，会默认禁用身份验证。

若要控制身份验证，请在创建或更新部署时使用 `auth_enabled` 参数。

如果已启用身份验证，可以使用 `get_keys` 方法来检索主要和辅助身份验证密钥：

```python
primary, secondary = service.get_keys()
print(primary)
```

> [!IMPORTANT]
> 如需重新生成密钥，请使用 [`service.regen_key`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py)。

## <a name="request-data"></a>请求数据

REST API 预期请求正文是采用以下结构的 JSON 文档：

```json
{
    "data":
        [
            <model-specific-data-structure>
        ]
}
```

> [!IMPORTANT]
> 数据结构需要符合服务中评分脚本和模型的预期。 评分脚本在将数据传递到模型之前可以修改数据。

例如，[在笔记本中训练](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-within-notebook/train-within-notebook.ipynb)示例中的模型预期存在由 10 个数字构成的数组。 此示例的评分脚本从请求创建一个 Numpy 数组，并将其传递给模型。 以下示例演示此服务预期的数据：

```json
{
    "data": 
        [
            [
                0.0199132141783263, 
                0.0506801187398187, 
                0.104808689473925, 
                0.0700725447072635, 
                -0.0359677812752396, 
                -0.0266789028311707, 
                -0.0249926566315915, 
                -0.00259226199818282, 
                0.00371173823343597, 
                0.0403433716478807
            ]
        ]
}
``` 

Web 服务可以接受一个请求中的多个数据集。 它会返回包含响应数组的 JSON 文档。

### <a name="binary-data"></a>二进制数据

如果模型接受二进制数据（如映像），则必须修改用于部署的 `score.py` 文件以接受原始 HTTP 请求。 下面是示例的`score.py`接受二进制数据：

```python 
from azureml.contrib.services.aml_request  import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse

def init():
    print("This is init()")

@rawhttp
def run(request):
    print("This is run()")
    print("Request: [{0}]".format(request))
    if request.method == 'GET':
        # For this example, just return the URL for GETs
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        reqBody = request.get_data(False)
        # For a real world solution, you would load the data from reqBody 
        # and send to the model. Then return the response.
        
        # For demonstration purposes, this example just returns the posted data as the response.
        return AMLResponse(reqBody, 200)
    else:
        return AMLResponse("bad request", 500)
```

> [!IMPORTANT]
> `azureml.contrib` 命名空间会频繁更改，因为我们正在改进服务。 因此，此命名空间中的任何内容都应被视为预览版，Microsoft 并不完全支持。
>
> 如果需要在本地开发环境中对此进行测试，可以使用以下命令安装 `contrib` 命名空间中的组件：
> 
> ```shell
> pip install azureml-contrib-services
> ```

## <a name="call-the-service-c"></a>调用服务 (C#)

此示例演示如何使用 C# 调用[在笔记本中训练](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-within-notebook/train-within-notebook.ipynb)示例创建的 Web 服务：

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using Newtonsoft.Json;

namespace MLWebServiceClient
{
    // The data structure expected by the service
    internal class InputData
    {
        [JsonProperty("data")]
        // The service used by this example expects an array containing
        //   one or more arrays of doubles
        internal double[,] data;
    }
    class Program
    {
        static void Main(string[] args)
        {
            // Set the scoring URI and authentication key
            string scoringUri = "<your web service URI>";
            string authKey = "<your key>";

            // Set the data to be sent to the service.
            // In this case, we are sending two sets of data to be scored.
            InputData payload = new InputData();
            payload.data = new double[,] {
                {
                    0.0199132141783263,
                    0.0506801187398187,
                    0.104808689473925,
                    0.0700725447072635,
                    -0.0359677812752396,
                    -0.0266789028311707,
                    -0.0249926566315915,
                    -0.00259226199818282,
                    0.00371173823343597,
                    0.0403433716478807
                },
                {
                    -0.0127796318808497, 
                    -0.044641636506989, 
                    0.0606183944448076, 
                    0.0528581912385822, 
                    0.0479653430750293, 
                    0.0293746718291555, 
                    -0.0176293810234174, 
                    0.0343088588777263, 
                    0.0702112981933102, 
                    0.00720651632920303
                }
            };

            // Create the HTTP client
            HttpClient client = new HttpClient();
            // Set the auth header. Only needed if the web service requires authentication.
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authKey);

            // Make the request
            try {
                var request = new HttpRequestMessage(HttpMethod.Post, new Uri(scoringUri));
                request.Content = new StringContent(JsonConvert.SerializeObject(payload));
                request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                var response = client.SendAsync(request).Result;
                // Display the response from the web service
                Console.WriteLine(response.Content.ReadAsStringAsync().Result);
            }
            catch (Exception e)
            {
                Console.Out.WriteLine(e.Message);
            }
        }
    }
}
```

返回的结果类似于以下 JSON 文档：

```json
[217.67978776218715, 224.78937091757172]
```

## <a name="call-the-service-go"></a>调用服务 (Go)

此示例演示如何使用 Go 调用[在笔记本中训练](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-within-notebook/train-within-notebook.ipynb)示例创建的 Web 服务：

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// Features for this model are an array of decimal values
type Features []float64

// The web service input can accept multiple sets of values for scoring
type InputData struct {
    Data []Features `json:"data",omitempty`
}

// Define some example data
var exampleData = []Features{
    []float64{
        0.0199132141783263, 
        0.0506801187398187, 
        0.104808689473925, 
        0.0700725447072635, 
        -0.0359677812752396, 
        -0.0266789028311707, 
        -0.0249926566315915, 
        -0.00259226199818282, 
        0.00371173823343597, 
        0.0403433716478807,
    },
    []float64{
        -0.0127796318808497, 
        -0.044641636506989, 
        0.0606183944448076, 
        0.0528581912385822, 
        0.0479653430750293, 
        0.0293746718291555, 
        -0.0176293810234174, 
        0.0343088588777263, 
        0.0702112981933102, 
        0.00720651632920303,
    },
}

// Set to the URI for your service
var serviceUri string = "<your web service URI>"
// Set to the authentication key (if any) for your service
var authKey string = "<your key>"

func main() {
    // Create the input data from example data
    jsonData := InputData{
        Data: exampleData,
    }
    // Create JSON from it and create the body for the HTTP request
    jsonValue, _ := json.Marshal(jsonData)
    body := bytes.NewBuffer(jsonValue)

    // Create the HTTP request
    client := &http.Client{}
    request, err := http.NewRequest("POST", serviceUri, body)
    request.Header.Add("Content-Type", "application/json")

    // These next two are only needed if using an authentication key
    bearer := fmt.Sprintf("Bearer %v", authKey)
    request.Header.Add("Authorization", bearer)

    // Send the request to the web service
    resp, err := client.Do(request)
    if err != nil {
        fmt.Println("Failure: ", err)
    }

    // Display the response received
    respBody, _ := ioutil.ReadAll(resp.Body)
    fmt.Println(string(respBody))
}
```

返回的结果类似于以下 JSON 文档：

```json
[217.67978776218715, 224.78937091757172]
```

## <a name="call-the-service-java"></a>调用服务 (Java)

此示例演示如何使用 Java 调用[在笔记本中训练](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-within-notebook/train-within-notebook.ipynb)示例创建的 Web 服务：

```java
import java.io.IOException;
import org.apache.http.client.fluent.*;
import org.apache.http.entity.ContentType;
import org.json.simple.JSONArray;
import org.json.simple.JSONObject;

public class App {
    // Handle making the request
    public static void sendRequest(String data) {
        // Replace with the scoring_uri of your service
        String uri = "<your web service URI>";
        // If using authentication, replace with the auth key
        String key = "<your key>";
        try {
            // Create the request
            Content content = Request.Post(uri)
            .addHeader("Content-Type", "application/json")
            // Only needed if using authentication
            .addHeader("Authorization", "Bearer " + key)
            // Set the JSON data as the body
            .bodyString(data, ContentType.APPLICATION_JSON)
            // Make the request and display the response.
            .execute().returnContent();
            System.out.println(content);
        }
        catch (IOException e) {
            System.out.println(e);
        }
    }
    public static void main(String[] args) {
        // Create the data to send to the service
        JSONObject obj = new JSONObject();
        // In this case, it's an array of arrays
        JSONArray dataItems = new JSONArray();
        // Inner array has 10 elements
        JSONArray item1 = new JSONArray();
        item1.add(0.0199132141783263);
        item1.add(0.0506801187398187);
        item1.add(0.104808689473925);
        item1.add(0.0700725447072635);
        item1.add(-0.0359677812752396);
        item1.add(-0.0266789028311707);
        item1.add(-0.0249926566315915);
        item1.add(-0.00259226199818282);
        item1.add(0.00371173823343597);
        item1.add(0.0403433716478807);
        // Add the first set of data to be scored
        dataItems.add(item1);
        // Create and add the second set
        JSONArray item2 = new JSONArray();
        item2.add(-0.0127796318808497);
        item2.add(-0.044641636506989);
        item2.add(0.0606183944448076);
        item2.add(0.0528581912385822);
        item2.add(0.0479653430750293);
        item2.add(0.0293746718291555);
        item2.add(-0.0176293810234174);
        item2.add(0.0343088588777263);
        item2.add(0.0702112981933102);
        item2.add(0.00720651632920303);
        dataItems.add(item2);
        obj.put("data", dataItems);

        // Make the request using the JSON document string
        sendRequest(obj.toJSONString());
    }
}
```

返回的结果类似于以下 JSON 文档：

```json
[217.67978776218715, 224.78937091757172]
```

## <a name="call-the-service-python"></a>调用服务 (Python)

此示例演示如何使用 Python 调用[在笔记本中训练](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-within-notebook/train-within-notebook.ipynb)示例创建的 Web 服务：

```python
import requests
import json

# URL for the web service
scoring_uri = '<your web service URI>'
# If the service is authenticated, set the key
key = '<your key>'

# Two sets of data to score, so we get two results back
data = {"data": 
            [
                [
                    0.0199132141783263, 
                    0.0506801187398187, 
                    0.104808689473925, 
                    0.0700725447072635, 
                    -0.0359677812752396, 
                    -0.0266789028311707, 
                    -0.0249926566315915, 
                    -0.00259226199818282, 
                    0.00371173823343597, 
                    0.0403433716478807
                ],
                [
                    -0.0127796318808497, 
                    -0.044641636506989, 
                    0.0606183944448076, 
                    0.0528581912385822, 
                    0.0479653430750293, 
                    0.0293746718291555, 
                    -0.0176293810234174, 
                    0.0343088588777263, 
                    0.0702112981933102, 
                    0.00720651632920303]
            ]
        }
# Convert to JSON string
input_data = json.dumps(data)

# Set the content type
headers = { 'Content-Type':'application/json' }
# If authentication is enabled, set the authorization header
headers['Authorization']=f'Bearer {key}'

# Make the request and display the response
resp = requests.post(scoring_uri, input_data, headers = headers)
print(resp.text)

```

返回的结果类似于以下 JSON 文档：

```JSON
[217.67978776218715, 224.78937091757172]
```
