---
title: 如何使用 Packer 创建 Linux Azure VM 映像 | Microsoft Docs
description: 了解如何使用 Packer 在 Azure 中创建 Linux 虚拟机映像
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/07/2019
ms.author: cynthn
ms.openlocfilehash: c0ec2616d8bdcf3cfd6d649f12e9bfceea33690a
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467749"
---
# <a name="how-to-use-packer-to-create-linux-virtual-machine-images-in-azure"></a>如何使用 Packer 在 Azure 中创建 Linux 虚拟机映像
Azure 中的每个虚拟机 (VM) 都创建至定义 Linux 分发和 OS 版本的映像。 映像可包括预安装的应用程序和配置。 Azure 市场为最常见的分发和应用程序环境提供许多第一和第三方映像，或者也可创建满足自身需求的自定义映像。 本文详细介绍了如何使用开源工具 [Packer](https://www.packer.io/) 在 Azure 中定义和生成自定义映像。

> [!NOTE]
> Azure 现在提供一种服务，Azure 映像生成器 （预览版） 的定义和创建自定义映像。 Azure 映像生成器基于 Packer，因此您甚至可以与之使用现有的 Packer shell 预配程序脚本。 若要开始使用 Azure 映像生成器，请参阅[使用 Azure 映像生成器创建 Linux VM](image-builder.md)。


## <a name="create-azure-resource-group"></a>创建 Azure 资源组
在生成过程中，Packer 会在生成源 VM 时创建临时 Azure 资源。 要捕获该源 VM 用作映像，必须定义资源组。 Packer 生成过程的输出存储在此资源组中。

使用 [az group create](/cli/azure/group) 创建资源组。 以下示例在 eastus 位置创建名为 myResourceGroup 的资源组：

```azurecli
az group create -n myResourceGroup -l eastus
```


## <a name="create-azure-credentials"></a>创建 Azure 凭据
Packer 使用服务主体向 Azure 进行身份验证。 Azure 服务主体是可用于应用、服务和 Packer 等自动化工具的安全标识。 控制和定义服务主体可在 Azure 中执行哪些操作的权限。

使用 [az ad sp create-for-rbac](/cli/azure/ad/sp) 创建服务主体并输出 Packer 需要的凭据：

```azurecli
az ad sp create-for-rbac --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"
```

前述命令的输出示例如下所示：

```azurecli
{
    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
```

若要向 Azure 进行身份验证，还需使用 [az account show](/cli/azure/account) 获取 Azure 订阅 ID：

```azurecli
az account show --query "{ subscription_id: id }"
```

在下一步中使用这两个命令的输出。


## <a name="define-packer-template"></a>定义 Packer 模板
若要生成映像，需创建一个模板作为 JSON 文件。 在模板中，定义执行实际生成过程的生成器和配置器。 Packer 具有[用于 Azure 的配置器](https://www.packer.io/docs/builders/azure.html)，可用于定义 Azure 资源，如在前面创建的服务主体凭据。

创建名为 ubuntu.json 的文件并粘贴以下内容。 为以下内容输入自己的值：

| 参数                           | 获取位置 |
|-------------------------------------|----------------------------------------------------|
| *client_id*                         | `az ad sp` 创建命令的第一行输出 - appId |
| client_secret                     | `az ad sp` 创建命令的第二行输出 - password |
| tenant_id                         | `az ad sp` 创建命令的第三行输出 - tenant |
| subscription_id                   | `az account show` 命令的输出 |
| managed_image_resource_group_name | 在第一步中创建的资源组的名称 |
| managed_image_name                | 创建的托管磁盘映像的名称 |


```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",

    "managed_image_resource_group_name": "myResourceGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Linux",
    "image_publisher": "Canonical",
    "image_offer": "UbuntuServer",
    "image_sku": "16.04-LTS",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "location": "East US",
    "vm_size": "Standard_DS2_v2"
  }],
  "provisioners": [{
    "execute_command": "chmod +x {{ .Path }}; {{ .Vars }} sudo -E sh '{{ .Path }}'",
    "inline": [
      "apt-get update",
      "apt-get upgrade -y",
      "apt-get -y install nginx",

      "/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync"
    ],
    "inline_shebang": "/bin/sh -x",
    "type": "shell"
  }]
}
```

此模板生成 Ubuntu 16.04 LTS 映像，安装 NGINX，然后取消设置 VM。

> [!NOTE]
> 如果将此模板扩展以设置用户凭据，请将取消设置 Azure 代理的配置程序命令调整为读取 `-deprovision`，而非 `deprovision+user`。
> `+user` 标志从源 VM 中删除所有用户帐户。


## <a name="build-packer-image"></a>生成 Packer 映像
如果尚未在本地计算机上安装 Packer，[请按照 Packer 安装说明进行安装](https://www.packer.io/docs/install/index.html)。

通过指定 Packer 模板文件生成映像，如下所示：

```bash
./packer build ubuntu.json
```

前述命令的输出示例如下所示：

```bash
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> dept : Engineering
==> azure-arm:  ->> task : Image deployment
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.218.147’
==> azure-arm: Waiting for SSH to become available...
==> azure-arm: Connected to SSH!
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-shell868574263
    azure-arm: WARNING! The waagent service will be stopped.
    azure-arm: WARNING! Cached DHCP leases will be deleted.
    azure-arm: WARNING! root password will be disabled. You will not be able to login as root.
    azure-arm: WARNING! /etc/resolvconf/resolv.conf.d/tail and /etc/resolvconf/resolv.conf.d/original will be deleted.
    azure-arm: WARNING! packer account and entire home directory will be deleted.
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-swtxmqm7ly/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Compute Name              : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

Packer 生成 VM、运行配置程序以及清理部署需要几分钟时间。


## <a name="create-vm-from-azure-image"></a>从 Azure 映像创建 VM
现在可使用 [az vm create](/cli/azure/vm) 从映像创建 VM。 指定使用 `--image` 参数创建的映像。 以下示例从 myPackerImage 创建一个名为 myVM 的 VM，并生成 SSH 密钥（如果它们尚不存在）：

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image myPackerImage \
    --admin-username azureuser \
    --generate-ssh-keys
```

如果你希望在与 Packer 映像不同的资源组或区域中创建虚拟机，请指定映像 ID 而不是映像名称。 可以使用 [az image show](/cli/azure/image#az-image-show) 获取映像 ID。

创建 VM 需要几分钟时间。 创建 VM 后，请记下 Azure CLI 显示的 `publicIpAddress`。 此地址将用于通过 Web 浏览器访问 NGINX 站点。

若要使 VM 能使用 Web 流量，请通过 [az vm open-port](/cli/azure/vm) 从 Internet 打开端口 80：

```azurecli
az vm open-port \
    --resource-group myResourceGroup \
    --name myVM \
    --port 80
```

## <a name="test-vm-and-nginx"></a>测试 VM 和 NGINX
现可打开 Web 浏览器，并在地址栏中输入 `http://publicIpAddress`。 在 VM 创建过程中提供自己的公共 IP 地址。 默认 NGINX 页如以下示例所示：

![NGINX 默认站点](./media/build-image-with-packer/nginx.png) 


## <a name="next-steps"></a>后续步骤
此外可以使用与现有 Packer 预配程序脚本[Azure 映像生成器](image-builder.md)。
