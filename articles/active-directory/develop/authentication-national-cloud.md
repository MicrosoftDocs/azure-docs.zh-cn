---
title: 在国家云中使用 Azure Active Directory 身份验证
description: 了解国家云的应用注册和身份验证终结点。
services: active-directory
documentationcenter: ''
author: negoe
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: negoe
ms.reviewer: negoe,CelesteDG
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: a0d4586df23548854f4acbfefd32081a36906097
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65067890"
---
# <a name="national-clouds"></a>国家/地区云

国家/地区云是 Azure 的物理独立的实例。 Azure 的这些区域旨在确保数据驻留、主权和合规性要求在地理边界内得到遵从。

包括全球云，Azure ActiveDirectory 在以下国家云中部署：  

- Azure 美国政府版
- Azure 德国
- Azure 中国世纪互联

各国云是唯一的，不同于 Azure 全局环境。 因此，在为这些环境开发应用程序（例如注册应用程序、获取令牌和配置终结点）时，了解一些关键差异非常重要。

## <a name="app-registration-endpoints"></a>应用注册终结点

没有为国家/地区云的每个单独的 Azure 门户。 若要在各国云中将应用程序与 Microsoft 标识平台集成，则需要在每个特定于环境的 Azure 门户中单独注册应用程序。

下表列出了用于为每个国家云注册应用程序的 Azure Active Directory (Azure AD) 终结点的基 URL。

| 国家云 | Azure AD 门户终结点 |
|----------------|--------------------------|
| 适用于美国政府的 Azure AD | `https://portal.azure.us` |
| Azure AD 德国 | `https://portal.microsoftazure.de` |
| 由世纪互联运营的 Azure AD 中国 | `https://portal.azure.cn` |
| Azure AD（全局服务） |`https://portal.azure.com` |

## <a name="azure-ad-authentication-endpoints"></a>Azure AD 身份验证终结点

所有各国云在每个环境中分别对用户进行身份验证，并具有单独的身份验证终结点。

下表列出了用于获取令牌以针对每个国家云的 Azure Active Directory (Azure AD) 终结点的基 URL。

| 国家云 | Azure AD 身份验证终结点 |
|----------------|-------------------------|
| 适用于美国政府的 Azure AD | `https://login.microsoftonline.us` |
| Azure AD 德国| `https://login.microsoftonline.de` |
| 由世纪互联运营的 Azure AD 中国 | `https://login.chinacloudapi.cn` |
| Azure AD（全局服务）| `https://login.microsoftonline.com` |

- 可以使用适当的特定于区域的基 URL 来形成对 Azure AD 授权或令牌终结点的请求。 例如，对于 Azure 德国：

  - 授权常用终结点为 `https://login.microsoftonline.de/common/oauth2/authorize`。
  - 令牌常用终结点为 `https://login.microsoftonline.de/common/oauth2/token`。

- 对于单租户应用程序，请将先前 URL 中的 common 替换为你的租户 ID 或名称，例如 `https://login.microsoftonline.de/contoso.com`。

> [!NOTE]
> [Azure AD v2.0 授权]( https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)和令牌终结点仅适用于全局服务。 国家云部署尚不支持它。

## <a name="microsoft-graph-api"></a>Microsoft Graph API

若要了解如何在各国云环境中调用 Microsoft Graph API，请转到各国云中的 [Microsoft Graph](https://developer.microsoft.com/graph/docs/concepts/deployments)。

> [!IMPORTANT]
> 全局服务的特定区域中的某些服务和功能可能无法在所有国家云中使用。 若要了解哪些服务可用，请访问[各区域的产品可用性](https://azure.microsoft.com/global-infrastructure/services/?products=all&regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia,china-non-regional,china-east,china-east-2,china-north,china-north-2,germany-non-regional,germany-central,germany-northeast)。

按照这[Microsoft 身份验证库 (MSAL) 教程](msal-national-cloud.md)若要了解如何构建使用 Microsoft 标识平台的应用程序。 具体而言，此应用程序将登录用户，获取访问令牌调用 Microsoft Graph API。

## <a name="next-steps"></a>后续步骤

了解有关以下方面的详细信息：

- [Azure Government](https://docs.microsoft.com/azure/azure-government/)
- [Azure 中国世纪互联](https://docs.microsoft.com/azure/china/)
- [Azure 德国](https://docs.microsoft.com/azure/germany/)
- [Azure AD 身份验证基础知识](authentication-scenarios.md)
