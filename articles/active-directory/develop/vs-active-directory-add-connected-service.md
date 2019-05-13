---
title: 在 Visual Studio 中使用连接服务添加 Azure Active Directory
description: 使用 Visual Studio 中的“添加连接服务”对话框添加 Azure Active Directory
services: active-directory
author: ghogen
manager: douge
ms.assetid: f599de6b-e369-436f-9cdc-48a0165684cb
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.devlang: multiple
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9bea0a870a6ef0685f4f4bce5ad3b0d1ff1f616a
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65414012"
---
# <a name="adding-an-azure-active-directory-by-using-connected-services-in-visual-studio"></a>在 Visual Studio 中使用连接服务添加 Azure Active Directory

通过使用 Azure Active Directory (Azure AD)，可以支持 ASP.NET MVC Web 应用程序或 Web API 服务中的 Active Directory 身份验证的单一登录 (SSO)。 通过 Azure AD 身份验证，用户可以使用其帐户从 Azure Active Directory 连接到 Web 应用程序。 通过 Web API 的 Azure AD 身份验证的优点包括，从 Web 应用程序公开 API 时提供增强的数据安全性。 通过 Azure AD，不需要使用其自己的帐户和用户管理来管理单独的身份验证系统。

本文及其同类文章提供了对 Active Directory 使用 Visual Studio 连接服务功能的详细信息。 功能是可在 Visual Studio 2015 及更高版本。

目前，Active Directory 连接服务不支持 ASP.NET Core 应用程序。

## <a name="prerequisites"></a>必备组件

- Azure 帐户：如果没有 Azure 帐户，可以[注册免费试用帐户](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)，或者[激活 Visual Studio 订户权益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F)。
- **Visual Studio 2015** 或更高版本。 [立即下载 Visual Studio](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。

### <a name="connect-to-azure-active-directory-using-the-connected-services-dialog"></a>使用“连接服务”对话框连接到 Azure Active Directory

1. 在 Visual Studio 中，创建或打开 ASP.NET MVC 项目或 ASP.NET Web API 项目。 可以使用 MVC、Web API、单页应用程序、Azure API 应用、Azure 移动应用和 Azure 移动服务模板。

1. 选择“项目”>“添加连接服务...”菜单命令，或双击解决方案资源管理器中项目下的“连接的服务”节点。

1. 在“连接服务”页，选择“使用 Azure Active Directory 进行身份验证”。

    ![“连接服务”页](./media/vs-azure-active-directory/connected-services-add-active-directory.png)

1. 在“简介”页上，选择“下一步”。 如果在此页上看到错误，请参阅[使用 Azure Active Directory 连接服务诊断错误](vs-active-directory-error.md)。

    ![“简介”页](./media/vs-azure-active-directory/configure-azure-ad-wizard-1.png)

1. 在“单一登录”页上，从“域”下拉列表中选择域。 该列表包含在 Visual Studio 的“帐户设置”对话框（“文件”>“帐户设置...”）中列出的帐户可以访问的所有域。如果没有找到要查找的域，作为替代方法，可以输入域名，如 `mydomain.onmicrosoft.com`。 可以选择用于创建 Azure Active Directory 应用的选项，也可以使用现有 Azure Active Directory 应用中的设置。 完成后，选择“下一步”。

    ![“单一登录”页](./media/vs-azure-active-directory/configure-azure-ad-wizard-2.png)

1. 在“目录访问权限”页上，根据需要选择“读取目录数据”。 开发人员通常会选择此选项。

    ![“目录访问权限”页](./media/vs-azure-active-directory/configure-azure-ad-wizard-3.png)

1. 选择“完成”以开始对项目进行修改，从而启用 Azure AD 身份验证。 Visual Studio 在此期间会显示进度：

    ![Active Directory 连接服务进度](./media/vs-azure-active-directory/active-directory-connected-service-output.png)

1. 处理完成后，Visual Studio 会根据项目类型，在浏览器中打开以下文章之一：

    - [.NET MVC 项目入门](vs-active-directory-dotnet-getting-started.md)
    - [WebAPI 项目入门](vs-active-directory-webapi-getting-started.md)

1. 还可以在 [Azure 门户](https://go.microsoft.com/fwlink/p/?LinkID=525040)中看到 Active Directory 域。

## <a name="how-your-project-is-modified"></a>项目的修改情况

运行“添加连接服务”向导时，Visual Studio 会将 Azure Active Directory 和关联的引用添加到项目。 还会修改项目中的配置文件和代码文件以添加对 Azure AD 的支持。 Visual Studio 所做的特定修改取决于项目类型。 有关详细信息，请参阅以下文章：

- [我的 .NET MVC 项目发生了什么情况？](vs-active-directory-dotnet-what-happened.md)
- [我的 Web API 项目发生了什么情况？](vs-active-directory-webapi-what-happened.md)

## <a name="next-steps"></a>后续步骤

- [Azure Active Directory 的身份验证方案](authentication-scenarios.md)
- [向 ASP.NET Web 应用添加 Microsoft 登录功能](quickstart-v1-aspnet-webapp.md)
