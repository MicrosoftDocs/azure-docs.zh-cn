---
title: 创建 Azure Red Hat OpenShift 的 Azure AD 租户 |Microsoft Docs
description: 下面介绍了如何创建 Azure Active Directory (Azure AD) 租户来承载 Microsoft Azure Red Hat OpenShift 群集。
author: tylermsft
ms.author: twhitney
ms.service: container-service
manager: jeconnoc
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 3c7f0b263cc775b7f9fec62e4d5388250e5124fb
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079115"
---
# <a name="create-an-azure-ad-tenant-for-azure-red-hat-openshift"></a>创建 Azure Red Hat OpenShift 的 Azure AD 租户

Microsoft Azure Red Hat OpenShift 需要[Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant)租户要在其中创建群集。 一个*租户*是专用的 Azure AD 的组织或应用程序的开发人员接收时它们创建与 Microsoft 通过注册 Azure、 Microsoft Intune 或 Microsoft 365 关系实例。 每个 Azure AD 租户是不同的独立于其他 Azure AD 租户并具有其自己的工作和学校标识和应用注册。

如果还没有 Azure AD 租户，请按照以下说明创建一个。

## <a name="create-a-new-azure-ad-tenant"></a>创建新的 Azure AD 租户

若要创建租户：

1. 登录到[Azure 门户](https://portal.azure.com/)使用你想要将与你的 Azure Red Hat OpenShift 群集相关联的帐户。
2. 打开[Azure Active Directory 边栏选项卡](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory)创建新租户 (也称为新*Azure Active Directory*)。
3. 提供**组织名称**。
4. 提供**初始域名**。 这将产生*onmicrosoft.com*追加到它。 可以重复使用的值*组织名称*此处。
5. 选择国家/地区或区域将在其中创建租户。
6. 单击**创建**。
7. 创建 Azure AD 租户后，选择**单击此处可管理新目录**链接。 新的租户名称应显示在 Azure 门户的右上方：  

    ![在右上角显示租户名称的门户屏幕截图][tenantcallout]  

8. 请记下的*租户 ID*以便稍后可以指定要在其中创建 Azure Red Hat OpenShift 群集。 在门户中，为新租户，现在应看到 Azure Active Directory 概述边栏选项卡。 选择**属性**复制的值和你**Directory ID**。 我们将把此值作为`tenant id`中[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程。

[tenantcallout]: ./media/howto-create-tenant/tenant-callout.png

## <a name="resources"></a>资源

请查看[Azure Active Directory 文档](https://docs.microsoft.com/azure/active-directory/)有关详细信息[Azure AD 租户](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant)。

## <a name="next-steps"></a>后续步骤

了解如何创建服务主体、 生成客户端机密和身份验证回调 URL，并测试 Azure Red Hat OpenShift 群集上的应用创建新的 Active Directory 用户。

[创建 Azure AD 应用程序对象和用户](howto-aad-app-configuration.md)