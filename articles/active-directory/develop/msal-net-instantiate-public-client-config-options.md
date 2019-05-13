---
title: 实例化包含选项 （适用于.NET 的 Microsoft 身份验证库） 的公共客户端应用程序 |Azure
description: 了解如何使用配置选项使用 Microsoft 身份验证库.NET (MSAL.NET) 实例化的公共客户端应用程序。
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: celested
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/30/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3a483cd6e22ce3c9fafb9b2b4d839e22c248f020
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65158695"
---
# <a name="instantiate-a-public-client-application-with-configuration-options-using-msalnet"></a>使用配置选项，使用 MSAL.NET 实例化的公共客户端应用程序

本文介绍如何实例化[公共客户端应用程序](msal-client-applications.md)使用 Microsoft 身份验证库.NET (MSAL.NET)。  使用设置文件中定义的配置选项实例化应用程序。

在之前初始化应用程序，首先需要向[注册](quickstart-register-app.md)它，以便您的应用程序可以与 Microsoft 标识平台集成。 注册后，可能需要以下信息 （这可在 Azure 门户中找到）：

- 客户端 ID （表示 GUID 的字符串）
- （命名实例） 的标识提供程序 URL 和应用程序的登录受众。 这两个参数统称为颁发机构。
- 如果你正在编写业务线应用程序仅为你的组织 （也名为单租户应用程序） 租户的 ID。
- 对于 web 应用，以及有时公共客户端应用程序 （尤其是当您的应用程序需要使用代理时），也将具有设置重定向 Uri 其中标识提供程序将联系后使用安全令牌的应用程序。


.NET Core 控制台应用程序可能具有以下*appsettings.json*配置文件：

```json
{
  "Authentication": {
    "AzureCloudInstance": "AzurePublic",
    "AadAuthorityAudience": "AzureAdMultipleOrgs",
    "ClientId": "ebe2ab4d-12b3-4446-8480-5c3828d04c50"
  },

  "WebAPI": {
    "MicrosoftGraphBaseEndpoint": "https://graph.microsoft.com"
  }
}
```

以下代码将读取此文件使用.NET 配置框架：

```csharp
public class SampleConfiguration
{
    /// <summary>
    /// Authentication options
    /// </summary>
    public PublicClientApplicationOptions PublicClientApplicationOptions { get; set; }

    /// <summary>
    /// Base URL for Microsoft Graph (it varies depending on whether the application is ran
    /// in Microsoft Azure public clouds or national / sovereign clouds
    /// </summary>
    public string MicrosoftGraphBaseEndpoint { get; set; }

    /// <summary>
    /// Reads the configuration from a json file
    /// </summary>
    /// <param name="path">Path to the configuration json file</param>
    /// <returns>SampleConfiguration as read from the json file</returns>
    public static SampleConfiguration ReadFromJsonFile(string path)
    {
        // .NET configuration
        IConfigurationRoot Configuration;
        var builder = new ConfigurationBuilder()
          .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile(path);
        Configuration = builder.Build();

        // Read the auth and graph endpoint config
        SampleConfiguration config = new SampleConfiguration()
        {
            PublicClientApplicationOptions = new PublicClientApplicationOptions()
        };
        Configuration.Bind("Authentication", config.PublicClientApplicationOptions);
        config.MicrosoftGraphBaseEndpoint = Configuration.GetValue<string>("WebAPI:MicrosoftGraphBaseEndpoint");
        return config;
    }
}
```

以下代码将创建应用程序，从设置文件中使用的配置：

```csharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .Build();
```

