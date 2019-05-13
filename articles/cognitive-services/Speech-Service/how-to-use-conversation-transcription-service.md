---
title: 转录语音 SDK-语音服务与多个参与者对话
titleSuffix: Azure Cognitive Services
description: 了解如何使用语音 SDK 会话的脚本。 适用于C++， C#，和 Java。
services: cognitive-services
author: jhakulin
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: jhakulin
ms.openlocfilehash: e9de4faf18c54f7c7582ef5a8ab0648629d4f48e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65190155"
---
# <a name="transcribe-multi-participant-conversations-with-the-speech-sdk"></a>转录语音 SDK 与多个参与者对话

Speech SDK **ConversationTranscriber** API 允许你能够添加、 删除和标识参与者通过流式处理到语音服务使用的音频转录会议/会话`PullStream`或`PushStream`.

## <a name="limitations"></a>限制

* 支持会话 transcriber C++， C#，并在 Windows、 Linux 和 Android 的 Java。
* ROOBO DevKit 所支持的硬件环境，用于创建会话，因为提供循环多麦克风阵列可有效地利用说话人识别会话脚本服务。 [有关详细信息，请参阅语音设备 SDK](speech-devices-sdk.md)。
* 语音 SDK 支持仅限于使用的音频拉取和推送模式下使用八个频道的 PCM 音频流。

## <a name="prerequisites"></a>必备组件

* [了解如何使用语音到文本和语音 SDK。](quickstart-csharp-dotnet-windows.md)
* [获取语音试用版订阅。](https://azure.microsoft.com/try/cognitive-services/)

## <a name="create-voice-signatures-for-participants"></a>为参与方创建语音签名

第一步是为会话参与者创建语音签名。 创建语音签名是必需的高效说话人识别。
在下面的示例中，我们将[使用 REST API 来获取语音签名。](https://aka.ms/cts/signaturegenservice)

下面的示例显示了两种不同方式创建语音签名：
```csharp
class Program
{
    static async Task CreateVoiceSignatureByUsingFormData()
    {
        byte[] fileBytes = File.ReadAllBytes(@"speakerVoice.wav");
        var form = new MultipartFormDataContent();
        var content = new ByteArrayContent(fileBytes);
        form.Add(content, "file", "file");
        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "YourSubscriptionKey");
        var response = await client.PostAsync($"https://{region}.signature.speech.microsoft.com/api/v1/Signature/GenerateVoiceSignatureFromFormData", form);
        // A voice signature can be extracted from the jsonData
        var jsonData = await response.Content.ReadAsStringAsync();
    }

    static async Task CreateVoiceSignatureByUsingBody()
    {
        byte[] fileBytes = File.ReadAllBytes(@"speakerVoice.wav");
        var content = new ByteArrayContent(fileBytes);

        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "YourSubscriptionKey");
        var response = await client.PostAsync($"https://{region}.cts.speech.microsoft.com/api/v1/Signature/GenerateVoiceSignatureFromByteArray", content);
        // A voice signature can be extracted from the jsonData
        var jsonData = await response.Content.ReadAsStringAsync();
    }

    static void Main(string[] args)
    {
        CreateVoiceSignatureByUsingFormData().Wait();
        CreateVoiceSignatureByUsingBody().Wait();
    }
}
```

## <a name="transcribing-conversations"></a>转录对话

若要转录与多个参与者的对话，创建`ConversationTranscriber`与关联的对象`AudioConfig`对话会话中和音频流使用创建的对象`PullAudioInputStream`或`PushAudioInputStream`。

我们假设您有一个名为 ConversationTranscriber 类`MyConversationTranscriber`。 你的代码可能如下所示：

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
using Microsoft.CognitiveServices.Speech.Conversation;

public class MyConversationTranscriber
{
    private static string endpoint = "YourOwnEndpoint";

    public static async Task ConversationWithPullAudioStreamAsync()
    {
        // Creates an instance of a speech config with specified subscription key and service region.
        // Replace with your own endpoint and subscription key.
        var config = SpeechConfig.FromEndpoint(new Uri(endpoint), "YourSubScriptionKey");
        var stopTranscription = new TaskCompletionSource<int>();

        // Create an audio stream from a wav file.
        // Replace with your own audio file name and Helper class which implements AudioConfig using PullAudioInputStreamCallback
        using (var audioInput = Helper.OpenWavFile(@"8channelsOfRecordedPCMAudio.wav"))
        {
            // Creates a conversation transcriber using audio stream input.
            using (var transcriber = new ConversationTranscriber(config, audioInput))
            {
                // Subscribes to events.
                transcriber.Recognizing += (s, e) =>
                {
                    Console.WriteLine($"RECOGNIZING: Text={e.Result.Text}");
                };

                transcriber.Recognized += (s, e) =>
                {
                    if (e.Result.Reason == ResultReason.RecognizedSpeech)
                    {
                        Console.WriteLine($"RECOGNIZED: Text={e.Result.Text}, SpeakerID={e.Result.SpeakerId}");
                    }
                    else if (e.Result.Reason == ResultReason.NoMatch)
                    {
                        Console.WriteLine($"NOMATCH: Speech could not be recognized.");
                    }
                };

                // Sets a conversation Id.
                transcriber.ConversationId = "AConversationFromTeams";

                // Add participants to the conversation.
                // Create data for voice signatures using REST API described in the earlier section in this document.
                // How to create voice signatureA, signatureB & signatureC variables, please check the SDK API samples.

                var speakerA = Participant.From("Speaker_A", "en-us", signatureA);
                var speakerB = Participant.From("Speaker_B", "en-us", signatureB);
                var speakerC = Participant.From("SPeaker_C", "en-us", signatureC);
                transcriber.AddParticipant(speakerA);
                transcriber.AddParticipant(speakerB);
                transcriber.AddParticipant(speakerC);

                // Starts transcribing of the conversation. Uses StopTranscribingAsync() to stop transcribing when all participants leave.
                await transcriber.StartTranscribingAsync().ConfigureAwait(false);

                // Waits for completion.
                // Use Task.WaitAny to keep the task rooted.
                Task.WaitAny(new[] { stopTranscription.Task });

                // Stop transcribing the conversation.
                await transcriber.StopTranscribingAsync().ConfigureAwait(false);

                // Ends the conversation.
                await transcriber.EndConversationAsync().ConfigureAwait(false);
            }
        }
    }
}
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的示例](https://aka.ms/csspeech/samples)
