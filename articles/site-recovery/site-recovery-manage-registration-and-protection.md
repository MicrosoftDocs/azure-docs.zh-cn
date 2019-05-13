---
title: 删除服务器并禁用保护 | Microsoft 文档
description: 本文介绍如何从 Site Recovery 保管库中注销服务器，以及如何禁用虚拟机和物理服务器的保护。
author: rajani-janaki-ram
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: rajani-janaki-ram
ms.openlocfilehash: 1b4cd5bb020e73dc9045eb164ce49931f818f72d
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415491"
---
# <a name="remove-servers-and-disable-protection"></a>删除服务器并禁用保护

本文介绍了如何从恢复服务保管库中取消注册服务器，以及如何禁用对计算机的 Site Recovery 保护。


## <a name="unregister-a--configuration-server"></a>取消注册配置服务器

如果将 VMware VM 或 Windows/Linux 物理服务器复制到 Azure，则可从保管库中注销未连接的配置服务器，如下所示：

1. [禁用对虚拟机的保护](#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure)。
2. [取消关联或删除](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy)复制策略。
3. [删除配置服务器](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server)

## <a name="unregister-a-vmm-server"></a>取消注册 VMM 服务器

1. 停止在要删除的 VMM 服务器上复制云中的虚拟机。
2. 删除由需要删除的 VMM 服务器上的云使用的任何网络映射。 在“Site Recovery 基础结构” > “对于 System Center VMM” > “网络映射”中，右键单击网络映射 > “删除”。
3. 记下 VMM 服务器的 ID。
4. 取消复制策略与要删除的 VMM 服务器上的云的关联。  在“Site Recovery 基础结构” > “对于 System Center VMM” >  “复制策略”中，右键单击关联的策略。 右键单击云“取消关联”。
5. 删除 VMM 服务器或主动节点。 在“Site Recovery 基础结构” > “对于 System Center VMM” > “VMM 服务器”中，右键单击服务器 > “删除”。
6. 如果 VMM 服务器处于“已断开连接”状态，请对 VMM 服务器下载并运行[清理脚本](https://aka.ms/asr-cleanup-script-vmm)。 使用“以管理员身份运行”选项打开 PowerShell，以更改默认 (LocalMachine) 范围的执行策略。 在脚本中，指定要删除的 VMM 服务器的 ID。 脚本会从服务器中删除注册和云配对信息。
5. 在所有辅助 VMM 服务器上，运行清理脚本。
6. 在任何其他被动 VMM 群集节点（已安装提供程序）上运行清理脚本。
7. 手动卸载 VMM 服务器上的提供程序。 如果有一个群集，请从所有节点删除。
8. 若要将虚拟机复制到 Azure，需要在已删除的云中从 Hyper-V 主机中卸载 Microsoft 恢复服务代理。

## <a name="unregister-a-hyper-v-host-in-a-hyper-v-site"></a>取消注册 Hyper-V 站点中的 Hyper-V 主机

未由 VMM 托管的 Hyper-V 主机将收集到 Hyper-V 站点中。 在 Hyper-V 站点中删除主机，如下所示：

1. 禁用位于主机上的 Hyper-V VM 的复制。
2. 取消关联 Hyper-V 站点的策略。 在“Site Recovery 基础结构” > “对于 Hyper-V 站点” >  “复制策略”中，右键单击关联的策略。 右键单击站点 >“取消关联”。
3. 删除 Hyper-V 主机。 依次转到“Site Recovery 基础结构” > “对于 Hyper-V 站点” > “Hyper-V 主机”，右键单击服务器，再单击“删除”。
4. 从 Hyper-V 站点中删除所有主机后，将该站点删除。 依次转到“Site Recovery 基础结构” > “对于 Hyper-V 站点” > “Hyper-V 站点”，右键单击站点，再单击“删除”。
5. 如果 Hyper-V 主机处于“已断开连接”状态，请对已删除的每个 Hyper-V 主机运行以下脚本。 该脚本清理服务器上的设置，并从保管库中取消注册该服务器。


```powershell
        pushd .
        try
        {
            $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
            $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
            $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
            $isAdmin=$principal.IsInRole($administrators)
            if (!$isAdmin)
            {
                "Please run the script as an administrator in elevated mode."
                $choice = Read-Host
                return;
            }

            $error.Clear()
            "This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
            $choice =  Read-Host

            if (!($choice -eq 'Y' -or $choice -eq 'y'))
            {
            "Stopping cleanup."
            return;
            }

            $serviceName = "dra"
            $service = Get-Service -Name $serviceName
            if ($service.Status -eq "Running")
            {
                "Stopping the Azure Site Recovery service..."
                net stop $serviceName
            }

            $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
            $registrationPath = $asrHivePath + '\Registration'
            $proxySettingsPath = $asrHivePath + '\ProxySettings'
            $draIdvalue = 'DraID'
            $idMgmtCloudContainerId='IdMgmtCloudContainerId'


            if (Test-Path $asrHivePath)
            {
                if (Test-Path $registrationPath)
                {
                    "Removing registration related registry keys."
                    Remove-Item -Recurse -Path $registrationPath
                }

                if (Test-Path $proxySettingsPath)
                {
                    "Removing proxy settings"
                    Remove-Item -Recurse -Path $proxySettingsPath
                }

                $regNode = Get-ItemProperty -Path $asrHivePath
                if($regNode.DraID -ne $null)
                {
                    "Removing DraId"
                    Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
                }
                if($regNode.IdMgmtCloudContainerId -ne $null)
                {
                    "Removing IdMgmtCloudContainerId"
                    Remove-ItemProperty -Path $asrHivePath -Name $idMgmtCloudContainerId
                }
                "Registry keys removed."
            }

            # First retrieve all the certificates to be deleted
            $ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
            # Open a cert store object
            $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
            $store.Open('ReadWrite')
            # Delete the certs
            "Removing all related certificates"
            foreach ($cert in $ASRcerts)
            {
                $store.Remove($cert)
            }
        }catch
        {
            [system.exception]
            Write-Host "Error occurred" -ForegroundColor "Red"
            $error[0]
            Write-Host "FAILED" -ForegroundColor "Red"
        }
        popd
```


## <a name="disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure"></a>禁用对 VMware VM 或物理服务器（VMware 到 Azure）的保护

1. 依次转到“受保护的项” > “复制的项”，右键单击计算机，再单击“禁用复制”。
2. 在“禁用复制”页中，选择下列选项之一：
    - **禁用复制并删除(推荐)** - 此选项从 Azure Site Recovery 中删除复制的项，并停止复制计算机。 此外，还将清理配置服务器上的复制配置，并停止对这个受保护的服务器收取 Site Recovery 费用。 请注意，此选项仅在配置服务器处于连接状态时使用。
    - **删除** - 只有在源环境已删除或无法访问（未连接）时，才应使用此选项。 此选项会从 Azure Site Recovery 中删除复制的项（停止计费）。 不过，并不会清理配置服务器上的复制配置。 

> [!NOTE]
> 这两个选项都不会从受保护的服务器中卸载移动服务，需要手动卸载。 如果打算使用相同的配置服务器重新保护服务器，可以跳过卸载移动服务这一步。

## <a name="disable-protection-for-a-azure-vm-azure-to-azure"></a>禁用对 Azure VM (Azure 到 Azure) 的保护

-  依次转到“受保护的项” > “复制的项”，右键单击计算机，再单击“禁用复制”。
> [!NOTE]
> 不会从受保护的服务器卸载移动服务，需要手动将其卸载。 如果您计划重新保护服务器，则可以跳过卸载移动服务。

## <a name="disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure"></a>禁用对 Hyper-V 虚拟机（Hyper-V 到 Azure）的保护

> [!NOTE]
> 要在没有 VMM 服务器的情况下将 Hyper-V VM 复制到 Azure，请执行此过程。 若要使用 **System Center VMM 到 Azure** 方案复制虚拟机，请按照“禁用对使用 System Center VMM 到 Azure 方案复制的 Hyper-V 虚拟机的保护”中的说明操作

1. 依次转到“受保护的项” > “复制的项”，右键单击计算机，再单击“禁用复制”。
2. 在“禁用复制”中，可以选择下列选项：
   - **禁用复制并删除(推荐)** - 此选项从 Azure Site Recovery 中删除复制的项，并停止复制计算机。 此外，还将清理本地虚拟机上的复制配置，并停止对这个受保护的服务器收取 Site Recovery 费用。
   - **删除** - 只有在源环境已删除或无法访问（未连接）时，才应使用此选项。 此选项会从 Azure Site Recovery 中删除复制的项（停止计费）。 不过，并不会清理本地虚拟机上的复制配置。 

     > [!NOTE]
     > 如果选择“删除”选项，请运行下面的一组脚本，清理本地 Hyper-V 服务器上的复制设置。
1. 在源 Hyper-V 主机服务器上，取消复制虚拟机。 将 SQLVM1 替换为虚拟机名称，并通过管理 PowerShell 运行以下脚本

```powershell
    $vmName = "SQLVM1"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario"></a>禁用对使用 System Center VMM 到 Azure 方案复制到 Azure 的 Hyper-V 虚拟机的保护

1. 依次转到“受保护的项” > “复制的项”，右键单击计算机，再单击“禁用复制”。
2. 在“禁用复制”中，选择下列选项之一：

   - **禁用复制并删除(推荐)** - 此选项从 Azure Site Recovery 中删除复制的项，并停止复制计算机。 此外，还将清理本地虚拟机上的复制配置，并停止对这个受保护的服务器收取 Site Recovery 费用。
   - **删除** - 只有在源环境已删除或无法访问（未连接）时，才应使用此选项。 此选项会从 Azure Site Recovery 中删除复制的项（停止计费）。 不过，并不会清理本地虚拟机上的复制配置。 

     > [!NOTE]
     > 如果选择“删除”选项，请运行下面的脚本，清理本地 VMM 服务器上的复制设置。
3. 从源 VMM 服务器上的 VMM 控制台使用 PowerShell（需要管理员权限）运行此脚本。 将占位符 SQLVM1 替换为虚拟机名称。

        $vm = get-scvirtualmachine -Name "SQLVM1"
        Set-SCVirtualMachine -VM $vm -ClearDRProtection
4. 上述步骤清理 VMM 服务器上的复制设置。 若要停止运行在 Hyper-V 主机服务器上的虚拟机的复制，请运行以下脚本。 将 SQLVM1 替换为虚拟机的名称，将 host01.contoso.com 替换为 Hyper-V 主机服务器的名称。

```powershell
    $vmName = "SQLVM1"
    $hostName  = "host01.contoso.com"
    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
    $replicationService.RemoveReplicationRelationship($vm.__PATH)
```

## <a name="disable-protection-for-a-hyper-v-virtual-machine-replicating-to-secondary-vmm-server-using-the-system-center-vmm-to-vmm-scenario"></a>禁用对使用 System Center VMM 到 VMM 方案复制到辅助 VMM 服务器的 Hyper-V 虚拟机的保护

1. 依次转到“受保护的项” > “复制的项”，右键单击计算机，再单击“禁用复制”。
2. 在“禁用复制”中，选择下列选项之一：

   - **禁用复制并删除(推荐)** - 此选项从 Azure Site Recovery 中删除复制的项，并停止复制计算机。 此外，还将清理本地虚拟机上的复制配置，并停止对这个受保护的服务器收取 Site Recovery 费用。
   - **删除** - 只有在源环境已删除或无法访问（未连接）时，才应使用此选项。 此选项会从 Azure Site Recovery 中删除复制的项（停止计费）。 不过，并不会清理本地虚拟机上的复制配置。 运行下面的一组脚本，清理本地虚拟机上的复制设置。
     > [!NOTE]
     > 如果选择“删除”选项，请运行下面的脚本，清理本地 VMM 服务器上的复制设置。

3. 从源 VMM 服务器上的 VMM 控制台使用 PowerShell（需要管理员权限）运行此脚本。 将占位符 SQLVM1 替换为虚拟机名称。

        $vm = get-scvirtualmachine -Name "SQLVM1"
        Set-SCVirtualMachine -VM $vm -ClearDRProtection
4. 在辅助 VMM 服务器上，运行下面的脚本，清理辅助虚拟机的设置：

        $vm = get-scvirtualmachine -Name "SQLVM1"
        Remove-SCVirtualMachine -VM $vm -Force
5. 在辅助 VMM 服务器上刷新 Hyper-V 主机服务器上的虚拟机，以便在 VMM 控制台中重新检测辅助 VM。
6. 上述步骤清理 VMM 服务器上的复制设置。 若要停止虚拟机的复制，请在主 VM 和辅助 VM 上运行以下脚本。 将 SQLVM1 替换为虚拟机名称。

        Remove-VMReplication –VMName “SQLVM1”




