---
title: 桌面应用程序调用 web Api （代码配置）-Microsoft 标识平台
description: 了解如何构建桌面应用调用 web Api （应用程序的代码配置）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/o7/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: c52f6fc66187d961dc93089a9f81f6de4d67fe41
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65075935"
---
# <a name="desktop-app-that-calls-web-apis---code-configuration"></a>桌面应用程序调用 web Api 的代码配置

现在，已创建你的应用程序，您将了解如何配置应用程序的坐标的代码。

## <a name="msal-libraries"></a>MSAL 库

唯一的 MSAL 库现在支持桌面应用程序是 MSAL.NET

## <a name="public-client-application"></a>公共客户端应用程序

代码的角度来看，从桌面应用程序是公共客户端应用程序，而这正是将生成并处理 MSAL.NET `IPublicClientApplication`。 再次操作将稍有不同您使用交互式身份验证。

![IPublicClientApplication](media/scenarios/public-client-application.png)

### <a name="exclusively-by-code"></a>以独占方式通过代码

下面的代码实例化的公共客户端应用程序中，在 Microsoft Azure 公有云，与他们的工作和学校帐户，或者其个人 Microsoft 帐户登录用户。

```CSharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

如果你想要使用交互式身份验证，如上图所示，你想要使用`.WithRedirectUri`修饰符：

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithRedirectUri(PublicClientApplicationBuilder.DefaultInteractiveDesktopRedirectUri)
         .Build();
```

### <a name="using-configuration-files"></a>使用配置文件

下面的代码实例化来自配置对象，这可能是已填写的以编程方式或从配置文件中读取的公共客户端应用程序

```CSharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="more-elaborated-configuration"></a>更详细的配置

可以详细阐述通过添加多个修饰符的构建的应用程序。 例如，如果你希望应用程序是国家/地区云 （此处美国政府版） 中的多租户应用程序，您可以编写：

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment,
                         AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

MSAL.NET 还包含有关 ADFS 2019 修饰符：

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

最后，如果你想要获取的令牌，为 Azure AD B2C 租户，可以指定你的租户，如下面的代码段中所示：

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```

### <a name="learn-more"></a>了解详细信息

若要了解如何配置 MSAL.NET 桌面应用程序的详细信息：

- 有关所有修饰符上可用的列表`PublicClientApplicationBuilder`，请参阅参考文档[PublicClientApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.appconfig.publicclientapplicationbuilder?view=azure-dotnet-preview#methods)
- 有关在中公开的所有选项的说明`PublicClientApplicationOptions`请参阅[PublicClientApplicationOptions](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.appconfig.publicclientapplicationoptions?view=azure-dotnet-preview)中的参考文档

## <a name="complete-example-with-configuration-options"></a>使用配置选项的完整示例

假设具有以下的.NET Core 控制台应用程序`appsettings.json`配置文件：

```JSon
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

有一些代码来读取此文件使用.NET 提供配置框架;

```CSharp
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
  config.MicrosoftGraphBaseEndpoint =
  Configuration.GetValue<string>("WebAPI:MicrosoftGraphBaseEndpoint");
  return config;
 }
}
```

现在，若要创建你的应用程序，您将只需编写以下代码：

```CSharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .Build();
```

和之前调用`.Build()`方法，您可以重写您的配置对调用进行`.WithXXX`方法之前所示。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [获取令牌的桌面应用程序](scenario-desktop-acquire-token.md)
