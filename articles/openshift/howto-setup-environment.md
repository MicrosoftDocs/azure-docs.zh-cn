---
title: 设置 Azure Red Hat OpenShift 开发环境 |Microsoft Docs
description: 以下是使用 Microsoft Azure Red Hat OpenShift 的先决条件。
services: openshift
keywords: red hat openshift 的安装程序设置
author: TylerMSFT
ms.author: twhitney
ms.date: 05/06/2019
ms.topic: conceptual
ms.service: openshift
manager: jeconnoc
ms.openlocfilehash: 3c265d6695af7ba1bc5833db59966a626cb29cb9
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65416068"
---
# <a name="set-up-your-azure-red-hat-openshift-dev-environment"></a>设置 Azure Red Hat OpenShift 开发环境

若要生成并运行 Microsoft Azure Red Hat OpenShift 的应用程序，你将需要：

* 购买 Azure 虚拟机保留实例。
* 安装版本 2.0.64 （或更高版本） 的 Azure CLI （或使用 Azure Cloud Shell）。
* 注册`openshiftmanagedcluster`功能和关联的资源提供程序。
* 创建一个 Azure Active Directory (Azure AD) 租户。
* 创建 Azure AD 应用程序对象。
* 创建 Azure AD 用户。

以下说明将指导你完成所有这些系统必备组件。

## <a name="purchase-azure-virtual-machine-reserved-instances"></a>购买 Azure 虚拟机保留实例

可以使用 Azure Red Hat OpenShift 之前，你将需要购买 Azure 虚拟机保留实例。

如果你是 Azure 客户，此处的如何[购买 Azure 虚拟机保留实例](https://aka.ms/openshift/buy)。 保留可减少你费用由预付费的完全托管的 Azure 服务。 请参阅[*什么是 Azure 预订*](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations)若要详细了解保留和如何他们节省资金。

如果您不是 Azure 客户[联系销售人员](https://aka.ms/openshift/contact-sales)并填写销售窗体底部的页后，可以启动进程。

## <a name="install-the-azure-cli"></a>安装 Azure CLI

Azure Red Hat OpenShift 需要版本 2.0.64 或更高版本的 Azure cli。 如果你已安装 Azure CLI，可以检查已通过运行哪个版本：

```bash
az --version
```

输出的第一行将具有的 CLI 版本，例如`azure-cli (2.0.64)`。

以下是有关的说明[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)如果需要全新安装或升级。

或者，可以使用[Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview)。 在使用 Azure Cloud Shell，请务必选择**Bash**如果你打算按照环境[创建和管理 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)系列教程。

## <a name="register-providers-and-features"></a>注册提供程序和功能

`Microsoft.ContainerService openshiftmanagedcluster`功能， `Microsoft.Solutions`，和`Microsoft.Network`必须到你的订阅部署第一个 Azure Red Hat OpenShift 群集之前手动注册提供程序。

若要手动注册这些提供程序和功能，可在 Azure 门户中使用以下说明从 Bash shell 中，如果你已安装 CLI，或从 Azure Cloud Shell (Bash) 会话:。
1. 如果你有多个 Azure 订阅，请指定相关的订阅 ID:

    ```bash
    az account set --subscription <SUBSCRIPTION ID>
    ```

2. 注册 Microsoft.ContainerService openshiftmanagedcluster 功能：

    ```bash
    az feature register --namespace Microsoft.ContainerService -n openshiftmanagedcluster
    ```

3. 注册 Microsoft.Solutions 提供程序：

    ```bash
    az provider register -n Microsoft.Solutions --wait
    ```

4. 注册 Microsoft.Network 提供程序：

    ```bash
    az provider register -n Microsoft.Network --wait
    ```

5. 注册 Microsoft.KeyVault 提供程序：

    ```bash
    az provider register -n Microsoft.KeyVault --wait
    ```

6. 刷新的注册 Microsoft.ContainerService 资源提供程序：

    ```bash
    az provider register -n Microsoft.ContainerService --wait
    ```

## <a name="create-an-azure-active-directory-azure-ad-tenant"></a>创建 Azure Active Directory (Azure AD) 租户

Azure Red Hat OpenShift 服务需要向 Microsoft 表示你的组织和其关系的关联的 Azure Active Directory (Azure AD) 租户。 Azure AD 租户，可注册、 构建和管理应用，以及使用其他 Azure 服务。

如果你没有 Azure AD 要作为租户用于 Azure Red Hat OpenShift 群集，或如果想要创建用于测试租户，请按照中的说明[创建 Azure Red Hat OpenShift 群集的 Azure AD 租户](howto-create-tenant.md)之前继续执行本指南。

## <a name="create-an-azure-ad-application-object-and-user"></a>创建 Azure AD 应用程序对象和用户

Azure Red Hat OpenShift 需要权限才能在你的群集，例如配置存储上执行任务。 这些权限都是通过[服务主体](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object)以及在注册 Azure AD 应用程序表示你想要在 Azure Red Hat OpenShift 上托管的工作负荷时创建。 此外需要创建一个新的 Active Directory 用户用于测试你的 Azure Red Hat OpenShift 群集上运行的应用。

按照中的说明[创建 Azure AD 应用程序对象和用户](howto-aad-app-configuration.md)若要了解如何创建服务主体，生成你的应用，客户端机密和身份验证回调 URL，并为测试创建新的 Active Directory 用户。

## <a name="next-steps"></a>后续步骤

现在，你可以使用 Azure Red Hat OpenShift ！

请尝试教程：
> [!div class="nextstepaction"]
> [创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)

[azure-cli-install]: https://docs.microsoft.com/cli/azure/install-azure-cli