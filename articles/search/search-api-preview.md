---
title: 预览 Azure 搜索 2019年-05-06-预览-REST API，Azure 搜索
description: Azure 搜索服务 REST API 版本 2019年-05-06-Preview 包括试验功能，如知识应用商店和客户托管的加密密钥。
services: search
author: HeidiSteen
manager: cgronlun
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: search
ms.date: 05/02/2019
ms.author: HeidiSteen
ms.custom: seodec2018
ms.openlocfilehash: 91c58507d8758a65772110afba71354deecd3b12
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65024286"
---
# <a name="azure-search-service-rest-api-version-2019-05-06-preview"></a>Azure 搜索服务 REST api 版本 2019年-05-06-预览版
本文介绍了 `api-version=2019-05-06-Preview` 版的 Azure 搜索服务 REST API，该版提供尚未正式发布的实验性功能。

> [!NOTE]
> 预览功能可用于目的为收集反馈的测试和试验，可能随时更改。 强烈建议不要在生产应用程序中使用预览版 API。


## <a name="new-in-2019-05-06-preview"></a>2019-05-06-Preview 中的新增功能

[**知识应用商店**](knowledge-store-concept-intro.md)是一个基于 AI 的扩充管道的新目标。 除了索引，您现在可以保留在 Azure 存储中的索引编制过程中创建的填充的数据结构。 控制通过技能组合，包括如何调整数据，无论数据存储在表存储或 Blob 存储中，以及是否有多个视图中的元素将数据的物理结构。

[**客户托管的加密密钥**](search-security-manage-encryption-keys.md)的服务端静态加密也是新的预览版功能。 除了内置的静态加密由 Microsoft 管理，您可以应用一层额外的加密密钥的唯一所有者所在。

## <a name="other-preview-features"></a>其他预览功能

公共预览版中仍包含早期预览版中发布的功能。 如果你正在调用使用较早预览版 API 版本的 API，可以继续使用该版本或切换到 `2019-05-06-Preview`，而不会对预期行为作出任何更改。

+ [moreLikeThis 查询参数](search-more-like-this.md)查找与特定文档相关的文档。 早期预览版中已有此功能。 


## <a name="how-to-call-a-preview-api"></a>如何调用预览版 API

早期预览版仍然可用，但随着时间推移会变得过时。 如果代码调用 `api-version=2016-09-01-Preview` 或 `api-version=2017-11-11-Preview`，那些调用仍然有效。 但是，只有最新预览版会获得改进。 

以下示例语法说明了对预览 API 版本的调用。

    GET https://[service name].search.windows.net/indexes/[index name]/docs?search=*&api-version=2019-05-06-Preview

Azure 搜索服务在多个版本内可用。 有关详细信息，请参阅 [API 版本](search-api-versions.md)。

## <a name="next-steps"></a>后续步骤

查看 Azure 搜索服务 REST API 参考文档。 如果遇到问题，我们寻求帮助上[堆栈溢出](https://stackoverflow.com/)或[请联系支持人员](https://azure.microsoft.com/support/community/?product=search)。

> [!div class="nextstepaction"]
> [搜索服务 REST API 参考](https://docs.microsoft.com/rest/api/searchservice/)