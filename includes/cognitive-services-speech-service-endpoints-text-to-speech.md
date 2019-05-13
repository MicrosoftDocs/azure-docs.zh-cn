---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 05/06/2019
ms.author: wolfma
ms.openlocfilehash: d5a4b3a07854c2664de7ec60f3677b666798a9bd
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65237086"
---
### <a name="standard-and-neural-voices"></a>标准和神经语音

使用此表来确定的标准和神经语音可用性区域/终结点：

| 区域 | 终结点 | 标准语音 | 神经语音 |
|--------|----------|-----------------|---------------|
| 澳大利亚东部 | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 加拿大中部 | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 美国中部 | `https://centralus.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 东亚 | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 美国东部 | `https://eastus.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 美国东部 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 法国中部 | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 印度中部 | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 日本东部 | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 韩国中部 | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 美国中北部 | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 北欧 | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 美国中南部 | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 东南亚 | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 英国南部 | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 西欧 | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |
| 美国西部 | `https://westus.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “否” |
| 美国西部 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/v1` | “是” | “是” |

### <a name="custom-voices"></a>自定义语音

如果你已创建自定义语音字体，使用已创建的终结点。 您还可以使用下面列出的终结点替换`{deploymentId}`的语音模型的部署 id。

| 区域 | 终结点 |
|--------|----------|
| 澳大利亚东部 | `https://australiaeast.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 加拿大中部 | `https://canadacentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国中部 | `https://centralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 东亚 | `https://eastasia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国东部 | `https://eastus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国东部 2 | `https://eastus2.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 法国中部 | `https://francecentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 印度中部 | `https://centralindia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 日本东部 | `https://japaneast.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 韩国中部 | `https://koreacentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国中北部 | `https://northcentralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 北欧 | `https://northeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国中南部 | `https://southcentralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 东南亚 | `https://southeastasia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 英国南部 | `https://uksouth.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 西欧 | `https://westeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国西部 | `https://westus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| 美国西部 2 | `https://westus2.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
