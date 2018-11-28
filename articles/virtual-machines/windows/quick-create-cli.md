---
title: 快速入门 - 使用 Azure PowerShell 创建 Windows 虚拟机| Microsoft Docs
description: 本快速入门介绍了如何使用 Azure PowerShell 创建 Windows 虚拟机
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 04/24/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 4ca8d42c1b2eece82fa31283b0df0d450e2f5afc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978873"
---
# <a name="quickstart-create-a-windows-virtual-machine-with-the-azure-cli"></a>快速入门：使用 Azure CLI 创建 Windows 虚拟机

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 这个快速入门展示了如何使用 Azure CLI 在 Azure 中部署运行 Windows Server 2016 的虚拟机 (VM)。 若要查看虚拟机的运行情况，可以通过 RDP 登录到该虚拟机并安装 IIS Web 服务器。

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本快速入门要求运行 Azure CLI 2.0.30 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](/cli/azure/group#az_group_create) 命令创建资源组。 Azure 资源组是部署和管理 Azure 资源的逻辑容器。 以下示例在 eastus 位置创建名为“我的资源组”的资源组：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [az vm create](/cli/azure/vm#az_vm_create) 创建虚拟机。 以下示例创建一个名为 *我的虚拟机* 的虚拟机。 此示例使用 *azure用户* 作为管理用户名，使用 *myPassword12* 作为密码。 更新这些值，使其适用于环境。 连接到虚拟机时需要这些值。

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword12
```

创建虚拟机和支持资源需要几分钟时间。 以下示例输出显示虚拟机创建操作已成功。

```azurecli-interactive
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

记下虚拟机输出中自己的 `公共IP地址`。 在后续步骤中，将使用此地址访问虚拟机。

## <a name="open-port-80-for-web-traffic"></a>为 Web 流量打开端口 80

默认情况下，在 Azure 中创建 Windows虚拟机时仅会打开 RDP 连接。 请使用 [az vm open-port](/cli/azure/vm#az_vm_open_port) 打开 TCP 端口 80 以供 IIS Web 服务器使用：

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

使用以下命令从本地计算机创建远程桌面会话。 将 IP 地址替换为您的虚拟机的公用 IP 地址。 出现提示时，输入创建 虚拟机时使用的凭据：

```powershell
mstsc /v:publicIpAddress
```

## <a name="install-web-server"></a>安装 Web 服务器

若要查看虚拟机的运行情况，请安装 IIS Web 服务器。 在虚拟机中打开 PowerShell 提示符并运行以下命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成后，关闭到虚拟机 的 RDP 连接。

## <a name="view-the-web-server-in-action"></a>查看Web 服务器的运行情况

IIS 已安装，并且现在已从 Internet 打开虚拟机上的端口 80 - 可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页面。 使用上一步中获取的虚拟机的公用 IP 地址。 以下示例展示了默认 IIS 网站：

![IIS 默认站点](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>清理资源

如果不再需要，可以删除资源组、虚拟机和所有相关的资源，可以使用 [az group delete](/cli/azure/group#az_group_delete) 命令将其删除：

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>下一个步骤

在本快速入门中，您部署了简单的虚拟机，为web流量打开了一个网络端口，并安装了一个基本 Web 服务器。 若要详细了解 Azure 虚拟机，请继续学习 Windows虚拟机的教程。

> [!div class="nextstepaction"]
> [Azure Windows 虚拟机教程](./tutorial-manage-vm.md)
