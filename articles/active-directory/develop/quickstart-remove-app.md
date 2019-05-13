---
title: 删除注册到 Microsoft 标识平台的应用程序 | Azure
description: 了解如何删除注册到 Microsoft 标识平台的应用程序。
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 271fe45082749afc310e8f217e8119d3931c3cac
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413742"
---
# <a name="quickstart-remove-an-application-registered-with-the-microsoft-identity-platform"></a>快速入门：删除注册到 Microsoft 标识平台的应用程序

已经向 Microsoft 标识平台注册了应用程序的企业开发人员和软件即服务 (SaaS) 提供商可能需要删除应用程序的注册。

在本快速入门中，你将学习如何：

* 删除你或你的组织编写的应用程序
* 删除其他组织编写的应用程序

## <a name="prerequisites"></a>先决条件

拥有一个其中已注册了应用程序的租户。 若要了解如何添加和注册应用，请参阅[将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)。

## <a name="remove-an-application-authored-by-you-or-your-organization"></a>删除你或你的组织编写的应用程序

由你或你的组织注册的应用程序是由租户中的应用程序对象与服务主体对象表示的。 有关详细信息，请参阅[应用程序对象和服务主体对象](active-directory-application-objects.md)。

### <a name="to-remove-an-application"></a>删除应用程序

1. 使用工作或学校帐户或个人 Microsoft 帐户登录到 [Azure 门户](https://portal.azure.com)。
2. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
3. 在左侧导航窗格中，选择“Azure Active Directory”服务，然后选择“应用注册”。 找到并选择要配置的应用程序。 选择应用以后，会看到应用程序的“概览”页。
4. 在“概览”页中，选择“删除”。
5. 选择“是”以确认要删除该应用。

   > [!NOTE]
   > 若要删除应用程序，你必须是应用程序的所有者或者具有管理员权限。

## <a name="remove-an-application-authored-by-another-organization"></a>删除其他组织编写的应用程序

如果在租户上下文中查看“应用注册”，则会发现在“所有应用”选项卡下显示的应用程序子集来自另一个租户，并已在许可过程中注册到你的租户。 更具体地说，它们仅由租户中的服务主体对象表示，没有相应的应用程序对象。 有关应用程序对象与服务主体对象之间的差别的详细信息，请参阅 [Azure AD 中的应用程序对象和服务主体对象](active-directory-application-objects.md)。

若要删除应用程序对目录的访问权限（在授予许可后），公司管理员必须删除该应用程序的服务主体。 管理员必须拥有全局管理员访问权限，并且可以通过 Azure 门户删除应用程序或使用 [Azure AD PowerShell Cmdlet](https://go.microsoft.com/fwlink/?LinkId=294151) 删除访问权限。

## <a name="next-steps"></a>后续步骤

了解下述其他相关的应用管理快速入门：

* [将应用程序注册到 Microsoft 标识平台](quickstart-register-app.md)
* [将客户端应用程序配置为访问 Web API](quickstart-configure-app-access-web-apis.md)
* [将应用程序配置为公开 Web API](quickstart-configure-app-expose-web-apis.md)
* [修改应用程序支持的帐户](quickstart-modify-supported-accounts.md)
