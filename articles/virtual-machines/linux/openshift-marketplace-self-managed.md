---
title: 部署 OpenShift 容器平台自行管理的 Marketplace 产品/服务在 Azure 中 |Microsoft Docs
description: 部署 OpenShift 容器平台自行管理的 Marketplace 产品/服务在 Azure 中。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldwongms
manager: mdotson
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/7/2019
ms.author: haroldw
ms.openlocfilehash: 9b981924dcaf715dd1d05d452b756a40b63f8dac
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233093"
---
# <a name="configure-prerequisites"></a>配置先决条件

在使用之前的 Marketplace 产品/服务部署在 Azure 中的自我管理的 OpenShift 容器平台群集，必须配置几个先决条件。  读取[OpenShift 先决条件](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-prerequisites)一文，了解如何创建 ssh 密钥 （无密码）、 Azure 密钥保管库、 密钥保管库机密和服务主体。

 
## <a name="deploy-using-the-marketplace-offer"></a>使用 Marketplace 产品/服务进行部署

若要部署到 Azure 中的自托管的 OpenShift 容器平台群集的最简单方法是使用[Azure Marketplace 产品/服务](https://azuremarketplace.microsoft.com/marketplace/apps/redhat.openshift-container-platform?tab=Overview)。

此选项最为简单，但它还自定义功能有限。 Marketplace 产品/服务部署 OpenShift 容器平台 3.11.82 并包括以下配置选项：

- **主节点**：三 (3) 个包含可配置实例类型的主节点。
- **Infra 节点**：三 (3) 个包含可配置实例类型的 Infra 节点。
- **节点**：（1 到 9） 之间的节点和实例类型的数量进行配置。
- **磁盘类型**：使用托管磁盘。
- **网络**：支持新的或现有网络和自定义的 CIDR 范围。
- **CNS**：可以启用 CNS。
- **指标**：可以启用 hawkular 指标。
- **日志记录**：可以启用 EFK 日志记录。
- **Azure 云提供商**：默认情况下启用，可禁用。

在 Azure 门户的左上方，单击**创建资源**，在搜索框中输入 openshift 容器平台并按 Enter。

   ![新的资源搜索](media/openshift-marketplace-self-managed/ocp-search.png)  
<br>

结果页将打开与**Red Hat OpenShift 容器平台自我管理**列表中。 

   ![新的资源搜索结果](media/openshift-marketplace-self-managed/ocp-searchresult.png)  
<br>

单击产品/服务以查看产品/服务的详细信息。 若要部署此产品/服务，请单击**创建**。 将显示 UI 来输入必要的参数。 第一个屏幕是**基础知识**边栏选项卡。

   ![产品/服务标题页](media/openshift-marketplace-self-managed/ocp-titlepage.png)  
<br>

**基础知识**

若要获取有关任何输入参数的帮助，悬停***我***参数名称旁边。

为输入参数输入值，然后单击**确定**。

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| VM 管理员用户名 | 所有 VM 实例上创建的管理员用户 |
| SSH 公钥的管理员用户 | 用于登录到 VM 的 SSH 公钥必须没有通行短语 |
| 订阅 | 要将群集部署到 azure 订阅 |
| 资源组 | 创建新的资源组或选择现有的空资源组的群集资源 |
| Location | 要将群集部署到 azure 区域 |

   ![产品/服务基础知识边栏选项卡](media/openshift-marketplace-self-managed/ocp-basics.png)  
<br>

**基础结构设置**

为输入参数输入值，然后单击**确定**。

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| OCP 群集名称前缀 | 群集用于配置的所有节点的主机名的前缀。 介于 1 到 20 个字符之间 |
| 主节点大小 | 接受默认 VM 大小，或单击**更改大小**选择不同的 VM 大小。  选择您的工作负载的适当 VM 大小 |
| 基础结构的节点大小 | 接受默认 VM 大小，或单击**更改大小**选择不同的 VM 大小。  选择您的工作负载的适当 VM 大小 |
| 应用程序节点数 | 接受默认 VM 大小，或单击**更改大小**选择不同的 VM 大小。  选择您的工作负载的适当 VM 大小 |
| 应用程序的节点大小 | 接受默认 VM 大小，或单击**更改大小**选择不同的 VM 大小。  选择您的工作负载的适当 VM 大小 |
| 堡垒主机大小 | 接受默认 VM 大小，或单击**更改大小**选择不同的 VM 大小。  选择您的工作负载的适当 VM 大小 |
| 新的或现有虚拟网络 | 创建新的 vNet （默认值） 或使用现有的 vNet |
| 选择默认 CIDR 设置或自定义 IP 范围 (CIDR) | 接受默认的 CIDR 范围或选择**自定义 IP 范围**并输入自定义的 CIDR 信息。  默认设置将与 10.0.0.0/14，具有 10.1.0.0/16，基础结构主机子网的 CIDR 创建 vNet，具有 10.2.0.0/16，子网和 10.3.0.0/16 计算和 cn 子网 |
| 密钥保管库资源组名称 | 包含密钥保管库的资源组的名称 |
| Key Vault 名称 | 名称包含通过使用机密的密钥保管库的 ssh 私钥。  只有字母数字字符和短划线允许，以及介于 3 到 24 个字符之间 |
| 机密名称 | 包含机密的名称 ssh 私钥。  允许仅字母数字字符和短划线 |

   ![产品/服务基础结构边栏选项卡](media/openshift-marketplace-self-managed/ocp-inframain.png)  
<br>

**更改大小**

若要选择不同的 VM 大小，请单击***更改大小***。  将打开 VM 选择窗口。  选择 VM 大小，并单击**选择**。

   ![选择 VM 大小](media/openshift-marketplace-self-managed/ocp-selectvmsize.png)  
<br>

**现有的虚拟网络**

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| 现有虚拟网络名称 | 现有的 vNet 的名称。 |
| 主节点的子网名称 | 主节点的现有子网名称。  需要包含至少 16 个 IP 地址并且符合 RFC 1918 |
| 子网名称的基础结构节点 | 名称的现有基础结构节点的子网。  需要包含至少 32 个 IP 地址并且符合 RFC 1918 |
| 计算和 cn 节点的子网名称 | 计算和 cn 节点的现有子网名称。  需要包含至少 32 个 IP 地址并且符合 RFC 1918 |
| 现有虚拟网络的资源组 | 包含现有的 vNet 的资源组的名称 |

   ![提供基础结构存在的 vnet](media/openshift-marketplace-self-managed/ocp-existingvnet.png)  
<br>

**自定义 IP 范围**

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| 虚拟网络的地址范围 | 为 vNet 的 CIDR 自定义 |
| 包含主节点的子网的地址范围 | 自定义 CIDR 主子网 |
| 包含基础结构节点的子网的地址范围 | 自定义 CIDR 子网基础结构 |
| 包含计算和 cn 节点的子网地址范围 | 自定义的计算和 cn 节点的 CIDR |

   ![提供基础结构自定义 IP 范围](media/openshift-marketplace-self-managed/ocp-customiprange.png)  
<br>

**OpenShift 容器平台**

为输入参数输入值，然后单击**确定**

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| OpenShift 管理员用户密码 | 初始 OpenShift 用户密码。  此用户还将是群集管理员 |
| 确认 OpenShift 管理员用户密码 | 重新键入 OpenShift 管理员用户密码 |
| Red Hat 订阅管理器用户名称 | 用户名称以访问你的 Red Hat 订阅或组织 id。  此凭据用于将 RHEL 实例注册到你的订阅，将不会存储由 Microsoft 或 Red Hat |
| Red Hat 订阅管理器用户密码 | 若要访问你的 Red Hat 订阅或激活密钥的密码。  此凭据用于将 RHEL 实例注册到你的订阅，将不会存储由 Microsoft 或 Red Hat |
| Red Hat 订阅管理器 OpenShift 池 ID | 包含 OpenShift 容器平台权利的池 ID。 确保具有足够授权的 OpenShift 容器平台群集的安装 |
| Broker 的 Red Hat 订阅管理器 OpenShift 池 ID / 主节点 | 池 ID 中包含 Broker 的 OpenShift 容器平台权利 / 主节点。 确保具有足够授权的 OpenShift 容器平台群集的安装。 如果不使用代理 / 主池 ID 中，输入应用程序节点的池 ID |
| 配置 Azure 云提供程序 | 配置 OpenShift 以将 Azure 云提供程序。 如果使用 Azure 磁盘的永久性卷将附加有必要。  默认值为是 |
| Azure AD 服务主体客户端 ID GUID | Azure AD 服务主体客户端 ID 的 GUID-也称为 AppID。 仅当配置 Azure 云提供程序设置为需要**是** |
| Azure AD 服务主体客户端 ID 密钥 | Azure AD 服务主体客户端 ID 密钥。 仅当配置 Azure 云提供程序设置为需要**是** |
 
   ![产品/服务 OpenShift 边栏选项卡](media/openshift-marketplace-self-managed/ocp-ocpmain.png)  
<br>

**其他设置**

其他设置边栏选项卡可以配置的 CN glusterfs 存储日志记录、 指标和路由器 Sub 域。  默认值不会安装下列任一选项，并且将使用 nip.io 作为路由器 sub 域以进行测试。 启用 CN 将安装包含三个将承载 glusterfs pod 其他附加磁盘的三个额外的计算节点。  

为输入参数输入值，然后单击**确定**

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| 配置本机存储容器 (CN) | 安装的 CN 中 OpenShift 群集，然后使它成为存储。 如果已禁用 Azure 提供程序将为默认值 |
| 配置群集日志记录 | 将 EFK 日志记录功能安装到群集。  基础结构的大小适当地向主机 EFK pod 节点 |
| 为群集配置指标 | 到 OpenShift 群集安装 Hawkular 指标。  基础结构的大小适当地向主机 Hawkular 指标 pod 节点 |
| 默认路由器子域 | 选择用于测试或生产环境中输入你自己的子域，自定义 nipio |
 
   ![其他边栏选项卡中的产品/服务](media/openshift-marketplace-self-managed/ocp-additionalmain.png)  
<br>

**其他设置的额外的参数**

| 输入的参数 | 参数说明 |
|-----------------------|-----------------|
| (CN)节点大小 | 接受默认节点大小，或选择**更改大小**以选择新的 VM 大小 |
| 输入你的自定义子域 | 要用于公开应用程序中通过 OpenShift 群集上的路由器使用的自定义路由域。  请确保创建适当的通配符 DNS 条目] |
 
   ![提供其他 cn 安装](media/openshift-marketplace-self-managed/ocp-additionalcnsall.png)  
<br>

**摘要**

在此阶段检查核心配额就足以部署为群集选择的 Vm 总数进行验证。  查看输入的所有参数。  如果可接受输入，请单击**确定**以继续。

   ![产品/服务摘要边栏选项卡](media/openshift-marketplace-self-managed/ocp-summary.png)  
<br>

**购买**

确认购买页上的联系人信息，然后单击**采购**接受的使用条款和启动 OpenShift 容器平台群集的部署。

   ![产品/服务购买边栏选项卡](media/openshift-marketplace-self-managed/ocp-purchase.png)  
<br>


## <a name="connect-to-the-openshift-cluster"></a>连接到 OpenShift 群集

部署完成后，从部署输出部分检索连接。 使用连接到 OpenShift 控制台中使用浏览器**OpenShift 控制台 URL**。 此外可以将 SSH 到堡垒主机。 在以下示例中，管理员用户名为 clusteradmin，守护主机的公共 IP DNS FQDN 为 bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com：

```bash
$ ssh clusteradmin@bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、OpenShift 群集和所有相关的资源，可以使用 [az group delete](/cli/azure/group) 命令将其删除。

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>后续步骤

- [部署后任务](./openshift-post-deployment.md)
- [在 Azure 中排查 OpenShift 部署问题](./openshift-troubleshooting.md)
- [OpenShift 容器平台入门](https://docs.openshift.com/container-platform)

