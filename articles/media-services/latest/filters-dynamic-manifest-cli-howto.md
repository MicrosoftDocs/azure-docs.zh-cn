---
title: 通过 CLI 使用 Azure 媒体服务创建筛选器 | Microsoft Docs
description: 本主题展示如何通过 CLI 使用媒体服务创建筛选器。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 8e1c031643fc3ce75d99ad619ce46b38c9cba82c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65472693"
---
# <a name="creating-filters-with-cli"></a>使用 CLI 创建筛选器 

将内容传送到客户（直播流事件或点播视频）时，客户端所需的灵活性可能比默认资产的清单文件中描述的灵活性更高。 使用 Azure 媒体服务可为内容定义帐户筛选器和资产筛选器。 有关详细信息，请参阅[筛选器和动态清单](filters-dynamic-manifest-overview.md)。

本主题说明如何为点播视频资产配置筛选器，并使用适用于媒体服务 v3 的 CLI 创建[帐户筛选器](https://docs.microsoft.com/cli/azure/ams/account-filter?view=azure-cli-latest)和[资产筛选器](https://docs.microsoft.com/cli/azure/ams/asset-filter?view=azure-cli-latest)。 

## <a name="prerequisites"></a>必备组件 

- [创建媒体服务帐户](create-account-cli-how-to.md)。 请务必记住资源组名称和媒体服务帐户名称。 
- 查看[筛选器和动态清单](filters-dynamic-manifest-overview.md)。

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="define-a-filter"></a>定义筛选器 

以下示例定义添加到最终清单的曲目择条件。 此筛选器包括属于 EC-3 的任何音频曲目和比特率在 0-1000000 范围内的任何视频曲目。

> [!TIP]
> 如果你打算定义**筛选器**在 REST 中，请注意，需要包括"属性"包装器 JSON 对象。  

```json
[
    {
        "trackSelections": [
            {
                "property": "Type",
                "value": "Audio",
                "operation": "Equal"
            },
            {
                "property": "FourCC",
                "value": "EC-3",
                "operation": "NotEqual"
            }
        ]
    },
    {
        "trackSelections": [
            {
                "property": "Type",
                "value": "Video",
                "operation": "Equal"
            },
            {
                "property": "Bitrate",
                "value": "0-1000000",
                "operation": "Equal"
            }
        ]
    }
]
```

## <a name="create-account-filters"></a>创建帐户筛选器

以下 [az ams account-filter](https://docs.microsoft.com/cli/azure/ams/account-filter?view=azure-cli-latest) 命令创建帐户筛选器，其中包含[之前定义的](#define-a-filter)筛选器曲目选项。 

该命令允许你传递一个可选 `--tracks` 参数，该参数包含表示曲目选择的 JSON。  使用 @{file} 从文件加载 JSON。 如果在本地使用 Azure CLI，请指定整个文件路径：

```azurecli
az ams account-filter create -a amsAccount -g resourceGroup -n filterName --tracks @tracks.json
```

此外，请参阅[筛选器的 JSON 示例](https://docs.microsoft.com/rest/api/media/accountfilters/createorupdate#create_an_account_filter)。

## <a name="create-asset-filters"></a>创建资产筛选器

以下 [az ams asset-filter](https://docs.microsoft.com/cli/azure/ams/asset-filter?view=azure-cli-latest) 命令创建资产筛选器，其中包含[之前定义的](#define-a-filter)筛选器曲目选项。 

```azurecli
az ams asset-filter create -a amsAccount -g resourceGroup -n filterName --asset-name assetName --tracks @tracks.json
```

此外，请参阅[筛选器的 JSON 示例](https://docs.microsoft.com/rest/api/media/assetfilters/createorupdate#create_an_asset_filter)。


## <a name="associate-filters-with-streaming-locator"></a>将筛选器与流式处理定位符相关联

可以指定将适用于你的流式处理定位符的资产或帐户筛选器的列表。 [动态打包程序 （流式处理终结点）](dynamic-packaging-overview.md)适用此列表以及这些客户端在 URL 中指定的筛选器。 此组合生成[动态清单](filters-dynamic-manifest-overview.md)，后者基于在 URL 中的筛选器 + 流式处理定位符指定的筛选器。 我们建议你使用此功能，如果想要应用筛选器，但不是希望公开在 URL 中的筛选器名称。

以下代码演示如何创建流式处理定位符，并指定`filters`。 这是一个可选属性，使用资产筛选器名称和/或帐户筛选器名称以空格分隔列表。

```azurecli
az ams streaming-locator create -a amsAccount -g resourceGroup -n streamingLocatorName \
                                --asset-name assetName \                               
                                --streaming-policy-name policyName \
                                --filters filterName1 filterName2
                                
```

## <a name="stream-using-filters"></a>Stream 使用筛选器

定义筛选器后，客户端可以在流式 URL 中使用它们。 筛选器可以应用于自适应比特率流式处理协议：Apple HTTP Live Streaming (HLS)、MPEG-DASH 和平滑流式处理。

下表显示了一些包含筛选器的 URL 示例：

|Protocol|示例|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|平滑流式处理|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-step"></a>后续步骤

[流视频](stream-files-tutorial-with-api.md) 

## <a name="see-also"></a>另请参阅

[Azure CLI](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
