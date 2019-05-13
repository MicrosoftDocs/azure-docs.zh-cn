---
title: Azure API 管理策略示例 | Microsoft Docs
description: 了解可在 Azure API 管理中使用的策略。
services: api-management
documentationcenter: ''
author: vladvino
manager: cflower
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: sample
ms.date: 10/31/2017
ms.author: apimpm
ms.custom: mvc
ms.openlocfilehash: 550161ce39aa944d0e01bb349ba48acbf719a860
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60859369"
---
# <a name="api-management-policy-samples"></a>API 管理策略示例

[策略](api-management-howto-policies.md)是一项强大的系统功能，可让发布者通过配置更改 API 的行为。 策略是一组语句，在请求或 API 的响应时按顺序执行。 下表包含示例链接，并提供每个示例的简要说明。

|                                                                                                                                                                      |                                                                                                                                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **入站策略**                                                                                                                                                 |                                                                                                                                                                                                                             |
| [添加 Forwarded 标头，使后端 API 能够构造正确的 URL](./policies/set-header-to-enable-backend-to-construct-urls.md?toc=api-management/toc.json) | 演示如何在入站请求中添加 Forwarded 标头，使后端 API 能够构造正确的 URL。                                                                                                        |
| [添加包含相关 ID 的标头](./policies/add-correlation-id.md?toc=api-management/toc.json)                                                             | 演示如何将包含相关 ID 的标头添加到入站请求。                                                                                                                                        |
| [将功能添加到后端服务并缓存响应](./policies/cache-response.md?toc=api-management/toc.json)                                             | 演示如何将功能添加到后端服务。 例如，接受位置的名称而不是天气预报 API 中的纬度和经度。                                                                    |
| [基于 JWT 声明授权访问权限](./policies/authorize-request-based-on-jwt-claims.md?toc=api-management/toc.json)                                              | 演示如何基于 JWT 声明授予对 API 中特定 HTTP 方法的访问权限。                                                                                                                                       |
| [使用外部授权者授权请求](./policies/authorize-request-using-external-authorizer.md)                                                   | 演示如何使用外部授权者保护 API 访问。                                                                                                                                                               |
| [使用 Google OAuth 令牌授予访问权限](./policies/use-google-as-oauth-token-provider.md?toc=api-management/toc.json)                                            | 演示如何使用 Google 作为 OAuth 令牌提供程序授予对终结点的访问权限。                                                                                                                                    |
| [生成共享访问签名并将请求转发到 Azure 存储](./policies/generate-shared-access-signature.md?toc=api-management/toc.json)                  | 演示如何使用表达式生成[共享访问签名](https://docs.microsoft.com/azure/storage/storage-dotnet-shared-access-signature-part-1)并使用 rewrite-uri 策略将请求转发到 Azure 存储。 |
| [从 AAD 获取 OAuth2 访问令牌并将其转发到后端](./policies/use-oauth2-for-authorization.md?toc=api-management/toc.json)                             | 提供一个示例用于演示如何使用 OAuth2 在网关与后端之间授权。 该示例演示如何从 AAD 获取访问令牌并将其转发到后端。                                                    |
| [使用发送请求策略从 SAP 网关获取 X-CSRF 令牌](./policies/get-x-csrf-token-from-sap-gateway.md?toc=api-management/toc.json)                           | 演示如何实现许多 API 所用的 X-CSRF 模式。 此示例特定于 SAP 网关。                                                                                                                           |
| [基于请求的正文大小路由请求](./policies/route-requests-based-on-size.md?toc=api-management/toc.json)                                            | 演示如何基于请求的正文大小路由请求。                                                                                                                                                       |
| [将请求上下文信息转发到后端服务](./policies/send-request-context-info-to-backend-service.md?toc=api-management/toc.json)                    | 演示如何将一些上下文信息发送到后端服务进行日志记录或处理。                                                                                                                                |
| [设置响应缓存持续时间](./policies/set-cache-duration.md?toc=api-management/toc.json)                                                                          | 演示如何使用后端发送的 Cache-Control 标头中的 maxAge 值设置响应缓存持续时间。                                                                                                             |
| **出站策略**                                                                                                                                                |                                                                                                                                                                                                                             |
| [筛选响应内容](./policies/filter-response-content.md?toc=api-management/toc.json)                                                                         | 演示如何基于与请求关联的产品从响应有效负载中筛选数据元素。                                                                                                        |
| **出错时的策略**                                                                                                                                                |                                                                                                                                                                                                                             |
| [将错误记录到 Stackify](./policies/log-errors-to-stackify.md?toc=api-management/toc.json)                                                                           | 演示如何添加错误日志记录策略，以便将错误发送到 Stackify 进行日志记录。                                                                                                                                            |
