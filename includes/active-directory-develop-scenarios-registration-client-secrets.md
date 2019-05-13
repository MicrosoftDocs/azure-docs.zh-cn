---
title: include 文件
description: 包括适用于登录页面 （守护程序、 Web 应用、 Web API） 的机密客户端方案文件
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/18/2018
ms.author: jmprieur
ms.custom: include file
ms.openlocfilehash: 9ee7422b372993d60c629524eb036b9678e5776c
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65074645"
---
## <a name="registration-of-secrets-or-certificates"></a>机密或证书的注册

如任何机密客户端应用程序中，你需要注册机密或证书。 可以通过中的交互式体验注册应用程序机密[Azure 门户](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview)，或使用命令行工具 （如 PowerShell)

### <a name="registering-client-secrets-using-the-application-registration-portal"></a>使用应用程序注册门户注册客户端机密

客户端凭据的管理分**证书和机密**页为应用程序：

![image](../articles/active-directory/develop/media/quickstart-update-azure-ad-app-preview/credentials-certificates-secrets-expanded.png)

- 应用程序机密 （也名为客户端机密） 是由 Azure AD，在机密客户端应用程序的注册过程中生成的。 选择时，会发生这一代**新的客户端机密**。 此时，您必须机密中的字符串复制剪贴板，以便使用在应用中，选择前**保存**。 不会再显示此字符串。
- 在应用程序注册中使用上传证书**上传证书**按钮

有关详细信息，请参阅[快速入门：配置客户端应用程序以访问 web Api |将凭据添加到你的应用程序](../articles/active-directory/develop/quickstart-configure-app-access-web-apis.md#add-credentials-to-your-web-application)

### <a name="registering-client-secrets-using-powershell"></a>使用 PowerShell 注册客户端机密

或者，可以使用命令行工具与 Azure AD 注册应用程序。 [处于活动状态的目录-dotnetcore-守护程序-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2)示例演示如何使用 Azure AD 应用程序注册应用程序机密或证书：

- 有关如何注册应用程序密码的详细信息，请参阅[AppCreationScripts/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts/Configure.ps1#L190)
- 有关如何与应用程序注册的证书的详细信息，请参阅[AppCreationScripts withCert/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts-withCert/Configure.ps1#L162-L178)