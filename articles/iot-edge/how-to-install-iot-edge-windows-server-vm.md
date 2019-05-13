---
title: 在 Windows Server 虚拟机上运行 Azure IoT Edge |Microsoft Docs
description: Azure IoT Edge 的设置说明的 Windows Server Marketplace 虚拟机
author: gregman-msft
manager: arjmands
ms.reviewer: kgremban
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 03/29/2019
ms.author: gregman
ms.openlocfilehash: be7479d3f042d6e64428a07e0509907b78595200
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159775"
---
# <a name="run-azure-iot-edge-on-windows-server-virtual-machines"></a>在 Windows Server 虚拟机上运行 Azure IoT Edge
使用 Azure IoT Edge 运行时可将设备转变为 IoT Edge 设备。 该运行时可以部署在像 Raspberry Pi 一样小的设备上，也可以部署在像工业服务器一样大的设备上。 使用 IoT Edge 运行时配置设备后，即可开始从云中部署业务逻辑。

若要了解有关 IoT Edge 运行时如何工作以及包含哪些组件的详细信息，请参阅[了解 Azure IoT Edge 运行时及其体系结构](iot-edge-runtime.md)。

本文列出了运行 Windows Server 2019 虚拟机使用的 Azure IoT Edge 运行时的步骤[Windows Server](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.WindowsServer?tab=Overview) Azure Marketplace 产品/服务。 按照的说明[安装 Azure IoT Edge 运行时](how-to-install-iot-edge-windows.md)上以用于其他版本的 Windows。

> [!NOTE]
> Windows Server 上的 IoT Edge 运行时处于[公共预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="deploy-from-the-azure-marketplace"></a>从 Azure 市场部署
1.  导航到[Windows Server](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.WindowsServer?tab=Overview) Azure Marketplace 产品/服务或搜索"Windows Server" [Azure Marketplace](https://azuremarketplace.microsoft.com/)
2.  选择**获取立即** 
3.  在中**软件计划**，找到"Windows Server 2019 数据中心服务器核心使用容器"，然后选择**继续**下一步对话框上。
    * 此外可以使用这些说明，对于其他版本的 Windows Server 容器
4.  进入 Azure 门户后，选择“创建”并遵循向导部署 VM。 
    *   如果是首次尝试使用 VM，它是最简单，若要使用密码，并在公共入站的端口菜单中启用 RDP 和 SSH。 
    *   如果你有资源密集型工作负荷，应通过添加更多 CPU 和/或内存来升级虚拟机大小。
5.  部署虚拟机后，请将其配置为连接到 IoT 中心：
    1.  复制在 IoT 中心创建的 IoT Edge 设备中的设备连接字符串（如果你不熟悉此过程，可以遵循[从 Azure 门户注册新的 Azure IoT Edge 设备](how-to-register-device-portal.md)操作指南）
    1.  在 Azure 门户中选择新建的虚拟机资源，并打开“运行命令”选项
    1.  选择**RunPowerShellScript**选项
    1.  将此脚本复制到你的设备连接字符串的命令窗口： 
        ```powershell
        . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
        Install-IoTEdge -Manual -DeviceConnectionString '<connection-string>'
        ```
    1.  执行脚本来安装 Edge 运行时，通过选择设置你的连接字符串**运行**
    1.  后一分钟或两个，应看到一条消息，Edge 运行时已安装并成功预配。


## <a name="deploy-from-the-azure-portal"></a>从 Azure 门户进行部署
1. 从 Azure 门户中，搜索"Windows Server"并选择**Windows Server 2019 Datacenter**开始 VM 创建工作流。 
2. 从**选择软件计划**选择"Windows Server 2019 数据中心服务器核心使用容器"，然后选择**创建**
3. 完成上面的"部署从 Azure Marketplace"中的步骤 5 说明。

## <a name="deploy-from-azure-cli"></a>从 Azure CLI 部署
1. 如果在桌面上使用 Azure CLI，请先登录：

   ```azurecli-interactive
   az login
   ```
    
1. 如果你有多个订阅，请选择要使用的订阅：
   1. 列出订阅：
    
      ```azurecli-interactive
      az account list --output table
      ```
    
   1. 复制要使用的订阅的 SubscriptionID 字段
   1. 使用复制的 ID 运行以下命令：
    
      ```azurecli-interactive 
      az account set -s {SubscriptionId}
      ```
    
1. 创建新资源组（或者在后续步骤中指定现有的资源组）：

   ```azurecli-interactive
   az group create --name IoTEdgeResources --location westus2
   ```
    
1. 创建新虚拟机：

   ```azurecli-interactive
   az vm create -g IoTEdgeResources -n EdgeVM --image MicrosoftWindowsServer:WindowsServer:2019-Datacenter-Core-with-Containers:latest  --admin-username azureuser --generate-ssh-keys --size Standard_DS1_v2
   ```
   * 此命令将提示您输入密码，但可以将选项添加`--admin-password`来更轻松地在脚本中设置
   * Windows Server Core 映像有命令行支持仅通过远程桌面，因此如果您想要完整的桌面体验，则指定`MicrosoftWindowsServer:WindowsServer:2019-Datacenter-with-Containers:latest`作为映像

1. 设置设备连接字符串（如果你不熟悉此过程，可以遵循[使用 Azure CLI 注册新的 Azure IoT Edge 设备](how-to-register-device-cli.md)操作指南）：

   ```azurecli-interactive
   az vm run-command invoke -g IoTEdgeResources -n EdgeVM --command-id RunPowerShellScript --script ". {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `Install-IoTEdge -Manual -DeviceConnectionString '<connection-string>'"
   ```

## <a name="next-steps"></a>后续步骤

预配了安装运行时的 IoT Edge 设备后，现在可以[部署 IoT Edge 模块](how-to-deploy-modules-portal.md)。

如果遇到问题的正确安装 Edge 运行时，请查看[故障排除](troubleshoot.md)页。

若要将现有安装更新到最新版本的 IoT Edge，请参阅[更新 IoT Edge 安全守护程序和运行时](how-to-update-iot-edge.md)。

深入了解如何使用 Windows 虚拟机[Windows 虚拟机文档](https://docs.microsoft.com/azure/virtual-machines/windows/)。

如果你想以 ssh 方式连接到此 VM 在安装完成后，请按照[安装 Windows Server 的 OpenSSH](https://docs.microsoft.com/windows-server/administration/openssh/openssh_install_firstuse#installing-openssh-with-powershell)指导使用远程桌面或远程 powershell。
