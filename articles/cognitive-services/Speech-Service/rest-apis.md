---
title: 语音服务 REST API
description: 语音服务 REST API 的参考。
services: cognitive-services
author: v-jerkin
ms.service: cognitive-services
ms.technology: speech
ms.topic: article
ms.date: 05/09/2018
ms.author: v-jerkin
ms.openlocfilehash: 64dce26303c0e700da54d371af5cb275b1613d70
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/28/2018
ms.locfileid: "43122097"
---
# <a name="speech-service-rest-apis"></a>语音服务 REST API

统一语音服务的 REST API 与[语音 API](https://docs.microsoft.com/azure/cognitive-services/Speech)（以前称为必应语音服务）提供的 API 非常类似。 该终结点与旧版语音服务使用的终结点不同。

## <a name="speech-to-text"></a>语音转文本

在语音转文本 API 中，只有所用的终结点与旧版语音服务语音识别 API 不同。 新的终结点如下表所示。 使用与订阅区域匹配的终结点。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-speech-to-text.md)]

除此以外，语音转文本 API 与旧版语音 API 的 [REST API](https://docs.microsoft.com/azure/cognitive-services/speech/getstarted/getstartedrest) 非常类似。

语音转文本 REST API 仅支持简短话语。 请求最多可包含 10 秒的音频，总共持续 14 秒。 REST API 只返回最终结果，不返回部分或中间结果。

> [!NOTE]
> 如果已自定义声学/语言模型或发音，请改用自定义终结点。

## <a name="text-to-speech"></a>文本到语音转换

新的文本转语音 API 支持 24-KHz 的音频输出。 `X-Microsoft-OutputFormat` 标头现可包含以下值。

|||
|-|-|
`raw-16khz-16bit-mono-pcm`         | `audio-16khz-16kbps-mono-siren`
`riff-16khz-16kbps-mono-siren`     | `riff-16khz-16bit-mono-pcm`
`audio-16khz-128kbitrate-mono-mp3` | `audio-16khz-64kbitrate-mono-mp3`
`audio-16khz-32kbitrate-mono-mp3`  | `raw-24khz-16bit-mono-pcm`
`riff-24khz-16bit-mono-pcm`        | `audio-24khz-160kbitrate-mono-mp3`
`audio-24khz-96kbitrate-mono-mp3`  | `audio-24khz-48kbitrate-mono-mp3`

语音服务现提供两种 24-KHz 语音：

区域设置 | 语言   | 性别 | 服务名称映射
-------|------------|--------|------------
en-US  | 美式英语 | 女 | “Microsoft 服务器语音的文本转语音（en-US，Jessa24kRUS）” 
en-US  | 美式英语 | 男   | “Microsoft 服务器语音的文本转语音（en-US，Guy24kRUS）”

下面是统一语音服务文本转语音 API 的 REST 终结点。 请使用与订阅区域匹配的终结点。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

在参考介绍旧版语音 API 的 [REST API 文档](https://docs.microsoft.com/azure/cognitive-services/speech/api-reference-rest/bingvoiceoutput)时，请牢记这些差异。

## <a name="authentication"></a>身份验证

需具备访问令牌才能请求语音服务的 REST API。 可通过向区域语音服务 `issueToken` 终结点提供订阅密钥获取令牌，如下表所示。 请使用与订阅区域匹配的终结点。

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-token-service.md)]

每个访问令牌的有效期为 10 分钟。 可随时获取新令牌，包括在每个语音 REST API 请求之前获取（如果需要）。 但是，为了最大限度地减少流量和延迟，建议同一个令牌使用 9 分钟。

以下部分介绍如何获取令牌以及如何在请求中使用令牌。

### <a name="getting-a-token-http"></a>获取令牌：HTTP

下面介绍用于获取令牌的示例 HTTP 请求。 将 `YOUR_SUBSCRIPTION_KEY` 替换为语音服务订阅密钥。 如果订阅不在美国西部区域中，请将 `Host` 标头替换为所在区域的主机名。

```
POST /sts/v1.0/issueToken HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: westus.api.cognitive.microsoft.com
Content-type: application/x-www-form-urlencoded
Content-Length: 0
```

对此请求的响应主体是 Java Web 令牌 (JWT) 格式的访问令牌。

### <a name="getting-a-token-powershell"></a>获取令牌：PowerShell

下面的 Windows PowerShell 脚本介绍了访问令牌的获取方式。 将 `YOUR_SUBSCRIPTION_KEY` 替换为语音服务订阅密钥。 如果订阅不在美国西部区域中，请相应地更改给定 URI 的主机名。

```Powershell
$FetchTokenHeader = @{
  'Content-type'='application/x-www-form-urlencoded';
  'Content-Length'= '0';
  'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
}

$OAuthToken = Invoke-RestMethod -Method POST -Uri https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken
 -Headers $FetchTokenHeader

# show the token received
$OAuthToken

```

### <a name="getting-a-token-curl"></a>获取令牌：cURL

cURL 是 Linux（及面向 Linux 的 Windows 子系统）中提供的一种命令行工具。 以下 cURL 命令介绍了访问令牌的获取方式。 将 `YOUR_SUBSCRIPTION_KEY` 替换为语音服务订阅密钥。 如果订阅不在美国西部区域中，请相应地更改给定 URI 的主机名。

> [!NOTE]
> 为方便阅读，该命令分行显示，但应在 shell 提示符处单行输入。

```
curl -v -X POST 
 "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken" 
 -H "Content-type: application/x-www-form-urlencoded" 
 -H "Content-Length: 0" 
 -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

### <a name="getting-a-token-c"></a>获取令牌：C#

以下 C# 类介绍了访问令牌的获取方式。 实例化类时，传递语音服务订阅密钥。 如果订阅不在美国西部区域中，请适当地更改 `FetchTokenUri` 的主机名。

```cs
    /*
     * This class demonstrates how to get a valid access token.
     */
    public class Authentication
    {
        public static readonly string FetchTokenUri =
            "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
        private string subscriptionKey;
        private string token;

        public Authentication(string subscriptionKey)
        {
            this.subscriptionKey = subscriptionKey;
            this.token = FetchTokenAsync(FetchTokenUri, subscriptionKey).Result;
        }

        public string GetAccessToken()
        {
            return this.token;
        }

        private async Task<string> FetchTokenAsync(string fetchUri, string subscriptionKey)
        {
            using (var client = new HttpClient())
            {
                client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
                UriBuilder uriBuilder = new UriBuilder(fetchUri);

                var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
                Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
                return await result.Content.ReadAsStringAsync();
            }
        }
    }
```

### <a name="using-a-token"></a>使用令牌

要在 REST API 请求中使用令牌，请在 `Authorization` 标头的 `Bearer` 单词后面提供该令牌。 例如，下面是包含令牌的文本转语音 REST 请求示例。 将 `YOUR_ACCESS_TOKEN` 替换为实际令牌，并在 `Host` 标头中使用正确的主机名。

```xml
POST /cognitiveservices/v1 HTTP/1.1
Authorization: Bearer YOUR_ACCESS_TOKEN
Host: westus.tts.speech.microsoft.com
Content-type: application/ssml+xml
Content-Length: 199
Connection: Keep-Alive

<speak version='1.0' xmlns="http://www.w3.org/2001/10/synthesis" xml:lang='en-US'>
<voice name='Microsoft Server Speech Text to Speech Voice (en-US, Jessa24kRUS)'>
    Hello, world!
</voice></speak>
```
> [!NOTE]
> Custom Voice服务中 ``` <voice> ``` 中 ``` name ``` 属性应为Font Name，否则将会报错。
  
  ![FontName](https://raw.githubusercontent.com/SylvesterLi/NoteRepo/master/PicSource/FontName.png)

### <a name="renewing-authorization"></a>续订授权

授权令牌在 10 分钟后过期。 通过在过期（如 9 分钟后）之前获取新令牌，可续订授权。 

以下 C# 代码是先前所述的类的直接替换项。 `Authentication` 类使用计时器每 9 分钟自动获取一个新的访问令牌。 此方法可确保在程序运行期间始终具备有效令牌。

> [!NOTE]
> 除使用计时器外，可存储获取当前令牌的时间戳，然后仅在当前令牌即将过期时请求新的令牌。 这样可避免在不必要的情况下请求新的令牌，它可能更适合用于不常发出语音请求的程序。

如上述一样，请确保 `FetchTokenUri` 值与订阅区域匹配。 实例化类时，传递订阅密钥。

```cs
    /*
     * This class demonstrates how to maintain a valid access token.
     */
    public class Authentication
    {
        public static readonly string FetchTokenUri = 
            "https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken";
        private string subscriptionKey;
        private string token;
        private Timer accessTokenRenewer;

        //Access token expires every 10 minutes. Renew it every 9 minutes.
        private const int RefreshTokenDuration = 9;

        public Authentication(string subscriptionKey)
        {
            this.subscriptionKey = subscriptionKey;
            this.token = FetchToken(FetchTokenUri, subscriptionKey).Result;

            // renew the token on set duration.
            accessTokenRenewer = new Timer(new TimerCallback(OnTokenExpiredCallback),
                                           this,
                                           TimeSpan.FromMinutes(RefreshTokenDuration),
                                           TimeSpan.FromMilliseconds(-1));
        }

        public string GetAccessToken()
        {
            return this.token;
        }

        private void RenewAccessToken()
        {
            this.token = FetchToken(FetchTokenUri, this.subscriptionKey).Result;
            Console.WriteLine("Renewed token.");
        }

        private void OnTokenExpiredCallback(object stateInfo)
        {
            try
            {
                RenewAccessToken();
            }
            catch (Exception ex)
            {
                Console.WriteLine(string.Format("Failed renewing access token. Details: {0}", ex.Message));
            }
            finally
            {
                try
                {
                    accessTokenRenewer.Change(TimeSpan.FromMinutes(RefreshTokenDuration), TimeSpan.FromMilliseconds(-1));
                }
                catch (Exception ex)
                {
                    Console.WriteLine(string.Format("Failed to reschedule the timer to renew access token. Details: {0}", ex.Message));
                }
            }
        }

        private async Task<string> FetchToken(string fetchUri, string subscriptionKey)
        {
            using (var client = new HttpClient())
            {
                client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
                UriBuilder uriBuilder = new UriBuilder(fetchUri);

                var result = await client.PostAsync(uriBuilder.Uri.AbsoluteUri, null);
                Console.WriteLine("Token Uri: {0}", uriBuilder.Uri.AbsoluteUri);
                return await result.Content.ReadAsStringAsync();
            }
        }
    }
```

## <a name="next-steps"></a>后续步骤

- [获取语音试用订阅](https://azure.microsoft.com/try/cognitive-services/)
- [自定义声学模型](how-to-customize-acoustic-models.md)
- [自定义语言模型](how-to-customize-language-model.md)

