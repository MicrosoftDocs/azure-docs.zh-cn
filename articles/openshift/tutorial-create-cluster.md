---
title: 教程 - 创建 Azure Red Hat OpenShift 群集 | Microsoft Docs
description: 了解如何使用 Azure CLI 创建 Microsoft Azure Red Hat OpenShift 群集
services: container-service
author: TylerMSFT
ms.author: twhitney
manager: jeconnoc
ms.topic: tutorial
ms.service: openshift
ms.date: 05/06/2019
ms.openlocfilehash: 5bc71a2d0f29fed163fb5c93ebd27df7f66a1325
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079472"
---
# <a name="tutorial-create-an-azure-red-hat-openshift-cluster"></a>教程：创建 Azure Red Hat OpenShift 群集

本教程是一个系列中的第一部分。 其中将会介绍如何使用 Azure CLI 创建 Microsoft Azure Red Hat OpenShift 群集，对其进行缩放，然后删除该群集以清理资源。

本教程系列的第一部分介绍了如何：

> [!div class="checklist"]
> * 创建 Azure Red Hat OpenShift 群集

在此系列教程中，你会学习如何：
> [!div class="checklist"]
> * 创建 Azure Red Hat OpenShift 群集
> * [缩放 Azure Red Hat OpenShift 群集](tutorial-scale-cluster.md)
> * [删除 Azure Red Hat OpenShift 群集](tutorial-delete-cluster.md)

## <a name="prerequisites"></a>先决条件

在开始学习本教程之前：

确保已[设置开发环境](howto-setup-environment.md)，包括：
- 安装最新的 CLI
- 创建租户
- 创建 Azure 应用程序对象
- 创建 Active Directory 用户用于登录到群集上运行的应用。

## <a name="step-1-sign-in-to-azure"></a>步骤 1：登录 Azure

如果在本地运行 Azure CLI，请打开 Bash 命令 shell，然后运行 `az login` 登录到 Azure。

```bash
az login
```

 如果你有权访问多个订阅，请运行 `az account set -s {subscription ID}`（将 `{subscription ID}` 替换为要使用的订阅）。

## <a name="step-2-create-an-azure-red-hat-openshift-cluster"></a>步骤 2：创建 Azure Red Hat OpenShift 群集

在 Bash 命令窗口中设置以下变量：

> [!IMPORTANT]
> 群集名称必须全部小写，否则群集创建将会失败。

```bash
CLUSTER_NAME=<cluster name in lowercase>
```

 使用[创建新应用注册](howto-aad-app-configuration.md#create-a-new-app-registration)的步骤 6 中所选的相同群集名称。

```bash
LOCATION=<location>
```

选择创建群集所在的位置。 有关支持 OpenShift on Azure 的 Azure 区域列表，请参阅[支持的区域](supported-resources.md#azure-regions)。 例如：`LOCATION=eastus`。

将 `FQDN` 设置为群集的完全限定名称。 此名称由群集名称和位置组成，其末尾追加了 `.cloudapp.azure.com`。 此 URL 与在[创建新应用注册](howto-aad-app-configuration.md#create-a-new-app-registration)的步骤 6 中创建的登录 URL 相同。 例如：  

```bash
FQDN=$CLUSTER_NAME.$LOCATION.cloudapp.azure.com
```

将 `APPID` 设置为在[创建新应用注册](howto-aad-app-configuration.md#create-a-new-app-registration)的步骤 9 中保存的值。  

```bash
APPID=<app ID value>
```

将 `SECRET` 设置为在[创建客户端机密](howto-aad-app-configuration.md#create-a-client-secret)的步骤 6 中保存的值。  

```bash
SECRET=<secret value>
```

将 `TENANT` 设置为在[创建新租户](howto-create-tenant.md#create-a-new-azure-ad-tenant)的步骤 7 中保存的租户 ID 值。  

```bash
TENANT=<tenant ID>
```

为群集创建资源组。 在用于定义上述变量的 Bash shell 中运行以下命令：

```bash
az group create --name $CLUSTER_NAME --location $LOCATION
```

### <a name="optional-connect-the-clusters-virtual-network-to-an-existing-virtual-network"></a>可选：将群集的虚拟网络连接到现有的虚拟网络

如果不需要将所创建群集的虚拟网络 (VNET) 连接到现有 VNET，请跳过此步骤。

首先获取现有 VNET 的标识符。 该标识符采用以下格式：`/subscriptions/{subscription id}/resourceGroups/{resource group of VNET}/providers/Microsoft.Network/virtualNetworks/{VNET name}`。

如果你不知道现有 VNET 所属的网络名称或资源组，请转到[“虚拟网络”边栏选项卡](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Network%2FvirtualNetworks)并单击你的虚拟网络。 此时会出现“虚拟网络”页，其中列出了该 VNET 所属的网络名称和资源组。

在 BASH shell 中使用以下 CLI 命令定义一个 VNET_ID 变量：

```bash
VNET_ID=$(az network vnet show -n {VNET name} -g {VNET resource group} --query id -o tsv)
```

例如： `VNET_ID=$(az network vnet show -n MyVirtualNetwork -g MyResourceGroup --query id -o tsv`

### <a name="create-the-cluster"></a>创建群集

现在已准备好创建群集。

 如果不需要将群集的虚拟网络连接到现有虚拟网络，请省略以下示例中的尾随 `--vnet-peer-id $VNET_ID` 参数。

```bash
az openshift create --resource-group $CLUSTER_NAME --name $CLUSTER_NAME -l $LOCATION --fqdn $FQDN --aad-client-app-id $APPID --aad-client-app-secret $SECRET --aad-tenant-id $TENANT --vnet-peer-id $VNET_ID
```

几分钟后，`az openshift create` 将成功完成并返回包含群集详细信息的 JSON 响应。

> [!NOTE]
> 如果有错误指出主机名不可用，原因可能是群集名称不唯一。 请尝试删除原始应用注册，并使用不同的群集名称重新执行 [创建新应用注册] (howto-aad-app-configuration.md#create-a-new-app-registration) 中的步骤（省略最后一个步骤，即创建新用户，因为已创建了一个用户）。

## <a name="step-3-sign-in-to-the-openshift-console"></a>步骤 3：登录到 OpenShift 控制台

现在可以登录到新群集的 OpenShift 控制台。 使用 [OpenShift Web 控制台](https://docs.openshift.com/dedicated/architecture/infrastructure_components/web_console.html)可以可视化、浏览和管理 OpenShift 项目的内容。

我们将以创建的用于测试的[新 Azure AD 用户](howto-aad-app-configuration.md#create-a-new-active-directory-user)身份登录。 为此，需要一个全新的浏览器实例，其中尚未缓存平时用来登录 Azure 门户的标识。

1. 打开 *incognito* 窗口 (Chrome) 或 *InPrivate* 窗口 (Microsoft Edge)。
2. 导航到在[创建新应用注册](howto-aad-app-configuration.md#create-a-new-app-registration)的步骤 6 中创建的登录 URL。 例如： https://constoso.eastus.cloudapp.azure.com

> [!NOTE]
> OpenShift 控制台使用自签名证书。
> 浏览器中出现提示时，请忽略警告，并接受“不受信任的”证书。

使用在[创建新的 Active Directory 用户](howto-aad-app-configuration.md#create-a-new-active-directory-user)中创建的用户和密码登录。出现“请求权限”对话框时，请依次选择“代表组织许可”、“接受”。

现已登录到群集控制台。

[OpenShift 群集控制台的屏幕截图](./media/aro-console.png)

 可以在 [Red Hat OpenShift](https://docs.openshift.com/dedicated/welcome/index.html) 文档中详细了解如何[使用 OpenShift 控制台](https://docs.openshift.com/dedicated/getting_started/developers_console.html)创建和生成映像。

## <a name="step-4-install-the-openshift-cli"></a>步骤 4：安装 OpenShift CLI

[OpenShift CLI](https://docs.openshift.com/dedicated/cli_reference/get_started_cli.html)（或 OC 工具）提供了用于管理应用程序的命令，以及用于与 OpenShift 群集各组件进行交互的低级别实用工具。

在 OpenShift 控制台中，单击右上角登录名旁边的问号，然后选择“命令行工具”。  单击“最新版本”链接下载并安装适用于 Linux、MacOS 或 Windows 的受支持 oc CLI。

> [!NOTE]
> 如果右上角未显示问号图标，请从左上方的下拉列表中选择“服务目录”或“应用程序控制台”。
>
> 或者，可以直接[下载 oc CLI](https://www.okd.io/download.html)。

“命令行工具”页提供了一个采用 `oc login https://<your cluster name>.<azure region>.cloudapp.azure.com --token=<token value>` 格式的命令。  请单击“复制到剪贴板”按钮以复制此命令。  在终端窗口中，将[路径设置](https://docs.okd.io/latest/cli_reference/get_started_cli.html#installing-the-cli)为包含 oc 工具的本地安装。 然后使用复制的 oc CLI 命令登录到群集。

如果使用上述步骤无法获取令牌值，请从 `https://<your cluster name>.<azure region>.cloudapp.azure.com/oauth/token/request` 获取令牌值。

## <a name="next-steps"></a>后续步骤

本教程的此部分介绍了如何：

> [!div class="checklist"]
> * 创建 Azure Red Hat OpenShift 群集

转到下一教程：
> [!div class="nextstepaction"]
> [缩放 Azure Red Hat OpenShift 群集](tutorial-scale-cluster.md)