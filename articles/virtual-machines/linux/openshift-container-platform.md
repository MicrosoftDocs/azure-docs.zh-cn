---
title: 在 Azure 中部署 OpenShift 容器平台 | Microsoft Docs
description: 在 Azure 中部署 OpenShift 容器平台。
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
ms.date: 04/18/2019
ms.author: haroldw
ms.openlocfilehash: 664099322bef3ac85d980fbe5e43dcc49cba862b
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411564"
---
# <a name="deploy-openshift-container-platform-in-azure"></a>在 Azure 中部署 OpenShift 容器平台

可以使用多种方法之一在 Azure 中部署 OpenShift 容器平台：

- 可以手动部署必要的 Azure 基础结构组件，然后按照 [OpenShift 容器平台文档](https://docs.openshift.com/container-platform)操作。
- 也可使用现有的[资源管理器模板](https://github.com/Microsoft/openshift-container-platform/)，该模板可简化 OpenShift 容器平台群集的部署流程。
- 另一种做法是使用 [Azure 市场产品](https://azuremarketplace.microsoft.com/marketplace/apps/redhat.openshift-container-platform?tab=Overview)。

所有选项均需要 Red Hat 订阅。 在部署期间，会将 Red Hat Enterprise Linux 实例注册到 Red Hat 订阅并附加到包含 OpenShift 容器平台权利的池 ID。
确保提供有效的 Red Hat 订阅管理员 (RHSM) 用户名、密码和池 ID。 可以使用激活密钥、组织 ID 和池 ID。 可通过登录 https://access.redhat.com 验证此信息。


## <a name="deploy-using-the-openshift-container-platform-resource-manager-template"></a>使用 OpenShift 容器平台资源管理器模板进行部署

### <a name="private-clusters"></a>专用群集

部署 OpenShift 的专用群集群集需要多个只是不具有公共 IP 关联到主负载均衡器 （web 控制台） 或基础结构负载均衡器 （路由器）。  专用群集通常使用自定义 DNS 服务器 （不是默认 Azure DNS）、 自定义域名 （例如 contoso.com) 和预定义的虚拟网络。  对于专用群集，需要提前使用所有相应的子网和 DNS 服务器设置配置虚拟网络。  然后，使用**existingMasterSubnetReference**， **existingInfraSubnetReference**， **existingCnsSubnetReference**，和**existingNodeSubnetReference**指定由群集使用的现有子网。

如果选择了专用的母版 (**masterClusterType**= 私有)，需要指定的静态专用 IP **masterPrivateClusterIp**。  此 IP 将分配给主负载均衡器的前端。  IP 必须在主机子网且未使用的 CIDR。  **masterClusterDnsType**必须设置为"自定义"和 DNS 名称必须为提供的主机**masterClusterDns**。  DNS 名称必须映射到静态专用 IP 和将用于访问主节点上的控制台。

如果选择专用路由器 (**routerClusterType**= 私有)，需要指定的静态专用 IP **routerPrivateClusterIp**。  此 IP 将分配到前端的基础结构负载均衡器。  IP 地址必须为 CIDR 中基础结构的子网且未被使用。  **routingSubDomainType**必须设置为"自定义"和通配符 DNS 名称中，路由必须提供用于对**routingSubDomain**。  

如果选择了专用的主机和专用路由器，在自定义域名，还必须为输入**domainName**

成功部署后，堡垒节点是可以使用 ssh 到公共 ip 的唯一节点。  即使主节点配置进行公共访问，它们不公开的 ssh 访问。

若要使用资源管理器模板进行部署，需要使用参数文件提供输入参数。 若要进一步自定义部署，请将 GitHub 存储库进行分支并更改相应的项。

一些常见的自定义选项包括但不限于：

- 守护 VM 大小（azuredeploy.json 中的变量）
- 命名约定（azuredeploy.json 中的变量）
- OpenShift 群集详细信息，通过主机文件修改 (deployOpenShift.sh)

### <a name="configure-the-parameters-file"></a>配置参数文件

[OpenShift 容器平台模板](https://github.com/Microsoft/openshift-container-platform)提供适用于不同 OpenShift 容器平台版本的多个分支。  根据需求，可以直接从存储库进行部署，也可以将存储库分叉，并在部署之前对模板或脚本进行自定义更改。

为 `aadClientId` 参数使用前面创建的服务主体中的 `appId` 值。

以下示例演示如何使用全部所需的输入创建名为 azuredeploy.parameters.json 的参数文件。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "_artifactsLocation": {
            "value": "https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master"
        },
        "location": {
            "value": "eastus"
        },
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_D4s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_D4s_v3"
        },
        "cnsVmSize": {
            "value": "Standard_E4s_v3"
        },
        "osImageType": {
            "value": "defaultgallery"
        },
        "marketplaceOsImage": {
            "value": {
                "publisher": "RedHat",
                "offer": "RHEL",
                "sku": "7-RAW",
                "version": "latest"
            }
        },
        "storageKind": {
            "value": "changeme"
        },
        "openshiftClusterPrefix": {
            "value": "changeme"
        },
        "minorVersion": {
            "value": "69"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 3
        },
        "nodeInstanceCount": {
            "value": 3
        },
        "cnsInstanceCount": {
            "value": 3
        },
        "osDiskSize": {
            "value": 64
        },
        "dataDiskSize": {
            "value": 64
        },
        "cnsGlusterDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "changeme"
        },
        "enableMetrics": {
            "value": "false"
        },
        "enableLogging": {
            "value": "false"
        },
        "enableCNS": {
            "value": "false"
        },
        "rhsmUsernameOrOrgId": {
            "value": "changeme"
        },
        "rhsmPoolId": {
            "value": "changeme"
        },
        "rhsmBrokerPoolId": {
            "value": "changeme"
        },
        "sshPublicKey": {
            "value": "GEN-SSH-PUB-KEY"
        },
        "keyVaultSubscriptionId": {
            "value": "255a325e-8276-4ada-af8f-33af5658eb34"
        },
        "keyVaultResourceGroup": {
            "value": "changeme"
        },
        "keyVaultName": {
            "value": "changeme"
        },
        "enableAzure": {
            "value": "true"
        },
        "aadClientId": {
            "value": "changeme"
        },
        "domainName": {
            "value": "contoso.com"
        },
        "masterClusterDnsType": {
            "value": "default"
        },
        "masterClusterDns": {
            "value": "console.contoso.com"
        },
        "routingSubDomainType": {
            "value": "nipio"
        },
        "routingSubDomain": {
            "value": "apps.contoso.com"
        },
        "virtualNetworkNewOrExisting": {
            "value": "new"
        },
        "virtualNetworkName": {
            "value": "changeme"
        },
        "addressPrefixes": {
            "value": "10.0.0.0/14"
        },
        "masterSubnetName": {
            "value": "changeme"
        },
        "masterSubnetPrefix": {
            "value": "10.1.0.0/16"
        },
        "infraSubnetName": {
            "value": "changeme"
        },
        "infraSubnetPrefix": {
            "value": "10.2.0.0/16"
        },
        "nodeSubnetName": {
            "value": "changeme"
        },
        "nodeSubnetPrefix": {
            "value": "10.3.0.0/16"
        },
        "existingMasterSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/mastersubnet"
        },
        "existingInfraSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/infrasubnet"
        },
        "existingCnsSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/cnssubnet"
        },
        "existingNodeSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/nodesubnet"
        },
        "masterClusterType": {
            "value": "public"
        },
        "masterPrivateClusterIp": {
            "value": "10.1.0.200"
        },
        "routerClusterType": {
            "value": "public"
        },
        "routerPrivateClusterIp": {
            "value": "10.2.0.200"
        },
        "routingCertType": {
            "value": "selfsigned"
        },
        "masterCertType": {
            "value": "selfsigned"
        }
    }
}
```

请将参数替换为你的特定信息。

不同的版本可能具有不同的参数，因此，请验证所用分支的必要参数。

### <a name="azuredeployparametersjson-file-explained"></a>azuredeploy。Parameters.json 文件解释

| 属性 | 描述 | 有效选项 | 默认值 |
|----------|-------------|---------------|---------------|
| `_artifactsLocation`  | 项目 （json、 脚本等） 的 URL |  |  https:\//raw.githubusercontent.com/Microsoft/openshift-container-platform/master  |
| `location` | 要将资源部署到 azure 区域 |  |  |
| `masterVmSize` | 主 VM 的大小。 从选择一种允许在 azuredeploy.json 文件中列出的 VM 大小 |  | Standard_E2s_v3 |
| `infraVmSize` | 大小的基础结构 VM。 从选择一种允许在 azuredeploy.json 文件中列出的 VM 大小 |  | Standard_D4s_v3 |
| `nodeVmSize` | 应用程序节点 VM 的大小。 从选择一种允许在 azuredeploy.json 文件中列出的 VM 大小 |  | Standard_D4s_v3 |
| `cnsVmSize` | 容器 (CN) 的本机存储节点 VM 的大小。 从选择一种允许在 azuredeploy.json 文件中列出的 VM 大小 |  | Standard_E4s_v3 |
| `osImageType` | 要使用的 RHEL 映像。 defaultgallery:按需;marketplace： 第三方映像 | defaultgallery <br> 市场 | defaultgallery |
| `marketplaceOsImage` | 如果`osImageType`marketplace，则为 publisher、 产品/服务、 sku、 版本 marketplace 产品/服务的输入适当的值。 此参数是一个对象类型 |  |  |
| `storageKind` | 要使用存储的类型  | 托管式<br> 非托管 | 托管式 |
| `openshiftClusterPrefix` | 群集用于配置的所有节点的主机名的前缀。  介于 1 到 20 个字符之间 |  | mycluster |
| `minoVersion` | OpenShift 容器平台 3.11 部署次要版本 |  | 69 |
| `masterInstanceCount` | 要部署的主机节点的数目 | 1, 3, 5 | 3 |
| `infraInstanceCount` | 数字的基础结构要部署节点 | 1, 2, 3 | 3 |
| `nodeInstanceCount` | 要部署的节点数 | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 | 2 |
| `cnsInstanceCount` | 若要部署的 CN 节点数 | 3, 4 | 3 |
| `osDiskSize` | （以 gb 为单位） VM 的 OS 磁盘的大小 | 64, 128, 256, 512, 1024, 2048 | 64 |
| `dataDiskSize` | 数据磁盘附加到节点 （以 gb 为单位） 的 Docker 卷的大小 | 32, 64, 128, 256, 512, 1024, 2048 | 64 |
| `cnsGlusterDiskSize` | 数据磁盘附加到 CNS 节点以便使用由 glusterfs （以 gb 为单位的大小 | 32, 64, 128, 256, 512, 1024, 2048 | 128 |
| `adminUsername` | OS (VM) 登录名和初始 OpenShift 用户的管理员用户名 |  | ocpadmin |
| `enableMetrics` | 启用指标。 指标则需要更多资源为基础结构 VM 因此选择适当大小 | true <br> false | false |
| `enableLogging` | 启用日志记录。 elasticsearch pod 需要 8 GB RAM 因此选择适当大小的基础结构 VM | true <br> false | false |
| `enableCNS` | 启用容器本机存储 | true <br> false | false |
| `rhsmUsernameOrOrgId` | Red Hat 订阅管理员用户名或组织 ID |  |  |
| `rhsmPoolId` | 包含计算节点的 OpenShift 授权的 Red Hat 订阅管理器池 ID |  |  |
| `rhsmBrokerPoolId` | 包含节点的 OpenShift 授权主服务器和基础结构的 Red Hat 订阅管理器池 ID。 如果不具有不同的池 Id，输入与 rhsmPoolId 相同的池 ID |  |
| `sshPublicKey` | 在 SSH 公钥复制此处 |  |  |
| `keyVaultSubscriptionId` | 包含密钥保管库的订阅的订阅 ID |  |  |
| `keyVaultResourceGroup` | 包含密钥保管库的资源组的名称 |  |  |
| `keyVaultName` | 您创建的密钥保管库的名称 |  |  |
| `enableAzure` | 启用 Azure 云提供程序 | true <br> false | true |
| `aadClientId` | 也称为应用程序 ID 为服务主体的 azure Active Directory 客户端 ID |  |  |
| `domainName` | 若要使用 （如果适用） 的自定义域名的名称。 设置为"none"如果不是部署的完全专用群集 |  | 无 |
| `masterClusterDnsType` | OpenShift web 控制台的域类型。 default 将基础结构使用的主 DNS 标签的公共 IP。 自定义允许你定义你自己的名称 | 默认 <br> 自定义 | 默认 |
| `masterClusterDns` | 要用于访问 OpenShift web 控制台中，如果选择了自定义的自定义 DNS 名称 `masterClusterDnsType` |  | console.contoso.com |
| `routingSubDomainType` | 如果设置为 nipio`routingSubDomain`将使用 nip.io。  如果你有自己想要使用的路由的域，请使用自定义 | nipio <br> 自定义 | nipio |
| `routingSubDomain` | 你想要用于路由如果选择了自定义作为通配符 DNS 名称 `routingSubDomainType` |  | apps.contoso.com |
| `virtualNetworkNewOrExisting` | 选择是否要使用现有的虚拟网络或创建新的虚拟网络 | 现有 <br> 新资源 | 新资源 |
| `virtualNetworkResourceGroupName` | 如果为 new 选择新的虚拟网络的资源组的名称 `virtualNetworkNewOrExisting` |  | resourceGroup().name |
| `virtualNetworkName` | 如果为 new 选择创建新虚拟网络的名称 `virtualNetworkNewOrExisting` |  | openshiftvnet |
| `addressPrefixes` | 新的虚拟网络的地址前缀 |  | 10.0.0.0/14 |
| `masterSubnetName` | 主子网的名称 |  | mastersubnet |
| `masterSubnetPrefix` | 用于主子网的 CIDR 需要的 addressPrefix 子集 |  | 10.1.0.0/16 |
| `infraSubnetName` | 名称的基础结构的子网 |  | infrasubnet |
| `infraSubnetPrefix` | 使用 CIDR 子网-必须是子集 addressPrefix 基础结构 |  | 10.2.0.0/16 |
| `nodeSubnetName` | 节点的子网的名称 |  | nodesubnet |
| `nodeSubnetPrefix` | 用于节点的子网的 CIDR 需要的 addressPrefix 子集 |  | 10.3.0.0/16 |
| `existingMasterSubnetReference` | 对现有子网用于主节点的完整引用。 不需要创建新的 vNet / 子网 |  |  |
| `existingInfraSubnetReference` | 完整的现有子网对引用的基础结构节点。 不需要创建新的 vNet / 子网 |  |  |
| `existingCnsSubnetReference` | 对现有子网的 CN 节点的完整引用。 不需要创建新的 vNet / 子网 |  |  |
| `existingNodeSubnetReference` | 对计算节点的现有子网的完整引用。 不需要创建新的 vNet / 子网 |  |  |
| `masterClusterType` | 指定是否在群集使用专用或公用的主节点。 如果选择专用，则不会向公共 IP 通过 Internet 公开的主节点。 相反，它将使用中指定的专用 IP `masterPrivateClusterIp` | 公共 <br> 私有 | 公共 |
| `masterPrivateClusterIp` | 如果选择了专用的主节点，然后将专用 IP 地址必须指定用于通过主节点的内部负载均衡器。 此静态 IP 必须在主机子网和尚不存在正在使用的 CIDR 块。 如果选择了公共的主节点，不会使用此值，但是仍必须指定 |  | 10.1.0.200 |
| `routerClusterType` | 指定是否在群集使用专用或公共基础结构节点。 如果选择专用，则基础结构节点不会暴露在 Internet 通过公共 IP。 相反，它将使用中指定的专用 IP `routerPrivateClusterIp` | 公共 <br> 私有 | 公共 |
| `routerPrivateClusterIp` | 如果是私有的基础结构选择节点，则必须为指定的专用 IP 地址通过使用内部负载均衡器基础结构节点。 此静态 IP 必须在主机子网和尚不存在正在使用的 CIDR 块。 如果公钥基础结构选择节点，不会使用此值，但是仍必须指定 |  | 10.2.0.200 |
| `routingCertType` | 自定义证书用于路由域或默认自签名的证书-按照中的说明进行操作**自定义证书**部分 | selfsigned <br> 自定义 | selfsigned |
| `masterCertType` | 自定义证书用于主机的域或默认自签名的证书-按照中的说明进行操作**自定义证书**部分 | selfsigned <br> 自定义 | selfsigned |

<br>

### <a name="deploy-using-azure-cli"></a>使用 Azure CLI 进行部署

> [!NOTE] 
> 以下命令需要 Azure CLI 2.0.8 或更高版本。 可以使用 `az --version` 命令检查 CLI 版本。 若要更新 CLI 版本，请参阅[安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latesti)。

以下示例使用名为 myOpenShiftCluster 的部署将 OpenShift 群集和所有相关资源部署到名为 openshiftrg 的资源组中。 它直接从 GitHub 存储库引用模板并使用名为 azuredeploy.parameters.json 的本地参数文件。

```azurecli 
az group deployment create -g openshiftrg --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

该部署需要至少 60 分钟才能完成，根据部署的节点和配置选项的总数。 部署完成时，终端上会输出 OpenShift 控制台的守护主机 DNS FQDN 和 URL。

```json
{
  "Bastion DNS FQDN": "bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com",
  "OpenShift Console URL": "http://openshiftlb.eastus.cloudapp.azure.com/console"
}
```

如果不想在命令行上等着部署完成，请将 `--no-wait` 添加为组部署的选项之一。 可以在 Azure 门户中资源组的部署部分检索部署的输出。

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
