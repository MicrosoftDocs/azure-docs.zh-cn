---
title: 使用 Azure PowerShell 创建 SQL Server Windows VM | Microsoft Docs
description: 本教程介绍如何使用 Azure PowerShell 创建 Windows SQL Server 2017 虚拟机。
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: craigg
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 12/21/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 69789f11cc0ab3702ebbc905ce86f9992868e259
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64715466"
---
# <a name="quickstart-create-a-sql-server-windows-virtual-machine-with-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建 SQL Server Windows 虚拟机

本快速入门逐步讲解如何使用 Azure PowerShell 创建 SQL Server 虚拟机。

> [!TIP]
> - 本快速入门提供的路径适用于快速预配并连接到 SQL VM。 若要详细了解创建 SQL VM 所需的其他 Azure PowerShell 选项，请参阅 [SQL Server VM 预配指南（使用 Azure PowerShell）](virtual-machines-windows-ps-sql-create.md)。
> - 如果对 SQL Server 虚拟机有任何疑问，请参阅[常见问题解答](virtual-machines-windows-sql-server-iaas-faq.md)。

## <a id="subscription"></a> 获取 Azure 订阅

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。


## <a id="powershell"></a> 获取 Azure PowerShell

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

## <a name="configure-powershell"></a>配置 PowerShell

1. 打开 PowerShell，通过运行 **Connect-AzAccount** 命令建立对 Azure 帐户的访问。

   ```powershell
   Connect-AzAccount
   ```

1. 此时应会显示用于输入凭据的屏幕。 使用登录 Azure 门户时所用的相同电子邮件和密码。

## <a name="create-a-resource-group"></a>创建资源组

1. 使用唯一的资源组名称定义一个变量。 为了简化本快速入门的其余部分，其余命令将此名称作为基础用于其他资源名称。

   ```powershell
   $ResourceGroupName = "sqlvm1"
   ```

1. 为所有 VM 资源定义目标 Azure 区域的位置。

   ```powershell
   $Location = "East US"
   ```

1. 创建资源组。

   ```powershell
   New-AzResourceGroup -Name $ResourceGroupName -Location $Location
   ```

## <a name="configure-network-settings"></a>配置网络设置

1. 创建虚拟网络、子网和公共 IP 地址。 这些资源用来与虚拟机建立网络连接，以及连接到 Internet。

   ``` PowerShell
   $SubnetName = $ResourceGroupName + "subnet"
   $VnetName = $ResourceGroupName + "vnet"
   $PipName = $ResourceGroupName + $(Get-Random)

   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix 192.168.1.0/24

   # Create a virtual network
   $Vnet = New-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Location $Location `
      -Name $VnetName -AddressPrefix 192.168.0.0/16 -Subnet $SubnetConfig

   # Create a public IP address and specify a DNS name
   $Pip = New-AzPublicIpAddress -ResourceGroupName $ResourceGroupName -Location $Location `
      -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name $PipName
   ```

1. 创建网络安全组。 配置允许远程桌面 (RDP) 和 SQL Server 连接的规则。

   ```powershell
   # Rule to allow remote desktop (RDP)
   $NsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "RDPRule" -Protocol Tcp `
      -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow

   #Rule to allow SQL Server connections on port 1433
   $NsgRuleSQL = New-AzNetworkSecurityRuleConfig -Name "MSSQLRule"  -Protocol Tcp `
      -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * `
      -DestinationAddressPrefix * -DestinationPortRange 1433 -Access Allow

   # Create the network security group
   $NsgName = $ResourceGroupName + "nsg"
   $Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName `
      -Location $Location -Name $NsgName `
      -SecurityRules $NsgRuleRDP,$NsgRuleSQL
   ```

1. 创建网络接口。

   ```powershell
   $InterfaceName = $ResourceGroupName + "int"
   $Interface = New-AzNetworkInterface -Name $InterfaceName `
      -ResourceGroupName $ResourceGroupName -Location $Location `
      -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $Pip.Id `
      -NetworkSecurityGroupId $Nsg.Id
   ```

## <a name="create-the-sql-vm"></a>创建 SQL VM

1. 定义登录到 VM 所需的凭据。 用户名为“azureadmin”。 确保在运行命令之前更改 \<密码>。

   ``` PowerShell
   # Define a credential object
   $SecurePassword = ConvertTo-SecureString '<password>' `
      -AsPlainText -Force
   $Cred = New-Object System.Management.Automation.PSCredential ("azureadmin", $securePassword)
   ```

1. 创建虚拟机配置对象，然后创建 VM。 以下命令创建基于 Windows Server 2016 的 SQL Server 2017 Developer Edition VM。

   ```powershell
   # Create a virtual machine configuration
   $VMName = $ResourceGroupName + "VM"
   $VMConfig = New-AzVMConfig -VMName $VMName -VMSize Standard_DS13_V2 |
      Set-AzVMOperatingSystem -Windows -ComputerName $VMName -Credential $Cred -ProvisionVMAgent -EnableAutoUpdate |
      Set-AzVMSourceImage -PublisherName "MicrosoftSQLServer" -Offer "SQL2017-WS2016" -Skus "SQLDEV" -Version "latest" |
      Add-AzVMNetworkInterface -Id $Interface.Id
   
   # Create the VM
   New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VMConfig
   ```

   > [!TIP]
   > 创建 VM 需要数分钟。

## <a name="install-the-sql-iaas-agent"></a>安装 SQL IaaS 代理

若要获取门户集成和 SQL VM 功能，必须安装 [SQL Server IaaS 代理扩展](virtual-machines-windows-sql-server-agent-extension.md)。 若要在新 VM 上安装该代理，请在创建 VM 后运行以下命令。

   ```powershell
   Set-AzVMSqlServerExtension -ResourceGroupName $ResourceGroupName -VMName $VMName -name "SQLIaasExtension" -version "1.2" -Location $Location
   ```

## <a name="remote-desktop-into-the-vm"></a>通过远程桌面连接到 VM

1. 使用以下命令检索新 VM 的公共 IP 地址。

   ```powershell
   Get-AzPublicIpAddress -ResourceGroupName $ResourceGroupName | Select IpAddress
   ```

1. 将返回的 IP 地址作为命令行参数传递给 **mstsc**，以便启动到新 VM 的远程桌面会话。

   ```
   mstsc /v:<publicIpAddress>
   ```

1. 系统提示输入凭据时，选择输入另一帐户的凭据。 输入带前置反斜杠的用户名（例如 `\azureadmin`），以及此前在本快速入门中设置的密码。

## <a name="connect-to-sql-server"></a>连接到 SQL Server

1. 登录到远程桌面会话以后，从开始菜单启动 **SQL Server Management Studio 2017**。

1. 在“连接到服务器”对话框中，保留默认设置。 服务器名称为 VM 名称。 身份验证设置为“Windows 身份验证”。 选择“连接”。

你现在已通过本地方式连接到 SQL Server。 若要进行远程连接，必须通过门户或手动[配置连接性](virtual-machines-windows-sql-connect.md)。

## <a name="clean-up-resources"></a>清理资源

如果不需要让 VM 继续运行，可以在不使用它时将它停止，以免产生不必要的费用。 以下命令可停止 VM，但保留它供将来使用。

```powershell
Stop-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
```

还可以使用 **Remove-AzResourceGroup** 命令永久删除与虚拟机关联的所有资源。 这样做也会永久删除虚拟机，因此请小心使用此命令。

## <a name="next-steps"></a>后续步骤

本快速入门使用 Azure PowerShell 创建了一个 SQL Server 2017 虚拟机。 若要详细了解如何将数据迁移到新的 SQL Server，请参阅以下文章。

> [!div class="nextstepaction"]
> [将数据库迁移到 SQL VM](virtual-machines-windows-migrate-sql.md)
