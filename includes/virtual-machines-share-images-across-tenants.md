---
title: include 文件
description: include 文件
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: cb7cf932433927d4ead853952ff74d55c55b9cf7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160015"
---
共享的映像库可以使用 RBAC 共享映像。 您可以使用 RBAC 共享你的租户，以及甚至对个人租户外部的映像。 但是，如果你想要共享 Azure 租户，在规模较大，外部图像应创建的应用注册，以方便共享。  使用应用注册可以支持更复杂的共享方案，如： 

* 管理共享的映像时一家公司收购另一个，并且 Azure 基础结构会分摊单独的租户。 
* Azure 合作伙伴代表客户管理 Azure 基础结构。 映像的自定义完成合作伙伴租户中，但基础结构部署都会发生在客户的租户。 


## <a name="create-the-app-registration"></a>创建应用程序注册

创建这两个租户将用于共享图像库资源的应用程序注册。
1. 打开[在 Azure 门户中的应用注册 （预览版）](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/)。    
1. 选择**新的注册**从页面顶部的菜单。
1. 在中**名称**，类型*myGalleryApp*。
1. 在中**支持的帐户类型**，选择**中的任何组织的目录和个人 Microsoft 帐户的帐户**。
1. 在中**重定向 URI**，类型*https://www.microsoft.com* ，然后选择**注册**。 创建应用注册后，将打开概述页。
1. 在概述页上，复制**应用程序 （客户端） ID**和更高版本将保存供使用。   
1. 选择**证书和机密**，然后选择**新的客户端机密**。
1. 在中**描述**，类型*共享映像库跨租户应用程序密码*。
1. 在中**Expires**，保留默认值为**1 年内**，然后选择**添加**。
1. 复制机密的值并将其保存到安全的位置。 在您离开页面后，无法检索它。


为提供使用共享的映像库的应用程序注册权限。
1. 在 Azure 门户中，选择你想要与其他租户共享共享映像库。
1. 选择**选择访问控制 (IAM)**，然后在**添加的角色分配**选择*添加*。 
1. 下**角色**，选择**读取器**。
1. 下**分配访问权限：**，将其作为**Azure AD 用户、 组或服务主体**。
1. 下**选择**，类型*myGalleryApp* ，然后选择它时它都显示在列表中。 完成后，选择**保存**。


创建租户 2 用于通过使用浏览器登录中请求访问应用程序的服务主体。 替换*<Tenant2 ID>* 与想要分享您的图片库和租户的租户 ID。 替换 *< 应用程序 （客户端） ID >* 创建的应用程序注册的应用程序 id。 完成后使替换内容，将 URL 粘贴到浏览器中，按照登录提示登录到租户 2。

```
https://login.microsoftonline.com/<Tenant 2 ID>/oauth2/authorize?client_id=<Application (client) ID>&response_type=code&redirect_uri=https%3A%2F%2Fwww.microsoft.com%2F 
```

在中[Azure 门户](https://portal.azure.com)以租户 2 身份登录，将应用注册访问权限提供给资源组你想要创建 VM。

1. 选择资源组，然后选择**访问控制 (IAM)**。 下**添加的角色分配**选择**添加**。 
1. 下**角色**，类型**参与者**。
1. 下**分配访问权限：**，将其作为**Azure AD 用户、 组或服务主体**。
1. 下**选择**类型*myGalleryApp*时它都显示在列表中选择它。 完成后，选择**保存**。

> [!NOTE]
> 需要要等待的时间才能完全完成正在生成并复制，可以使用相同的托管的映像创建另一个映像版本的映像版本。

