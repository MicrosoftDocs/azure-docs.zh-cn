---
title: 排查 Azure VM 上的 BitLocker 启动错误 | Microsoft Docs
description: 了解如何排查 Azure VM 中的 BitLocker 启动错误
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: cshepard
editor: v-jesits
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 03/25/2019
ms.author: genli
ms.openlocfilehash: 5ac40e3c98da613b53c06908bcde927cc2e81b08
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64684763"
---
# <a name="bitlocker-boot-errors-on-an-azure-vm"></a>Azure VM 上的 BitLocker 启动错误

 本文介绍在 Microsoft Azure 中启动 Windows 虚拟机 (VM) 时可能遇到的 BitLocker 错误。

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="symptom"></a>症状

 Windows VM 不启动。 检查[启动诊断](../windows/boot-diagnostics.md)窗口中的屏幕截图时，看到以下错误消息之一：

- 插入含 BitLocker 密钥的 USB 驱动程序

- 你被锁定！ 输入恢复密钥，以便再次开始操作（键盘布局：美式键盘）错误登录信息输入次数过多，因此，你的 PC 被锁定以保护你的隐私。 若要检索恢复密钥，请从另一电脑或移动设备转到 https://windows.microsoft.com/recoverykeyfaq。 如果需要，密钥 ID 为 XXXXXXX。 或者，可以重置电脑。

- 输入密码以解锁此驱动器 [ ] 按 Insert 键在键入时查看密码。
- 输入恢复密钥 从 USB 设备加载恢复密钥。

## <a name="cause"></a>原因

如果 VM 找不到用于解密加密磁盘的 BitLocker 恢复密钥 (BEK) 文件，则可能会出现此问题。

## <a name="solution"></a>解决方案

若要解决此问题，停止并解除分配 VM，然后重启。 此操作将强制 VM 从 Azure Key Vault 中检索 BEK 文件，然后将其放在加密磁盘上。 

如果此方法未能解决此问题，请执行以下步骤，手动还原 BEK 文件：

1. 拍摄受影响的 VM 的系统磁盘的快照作为备份。 有关详细信息，请参阅[拍摄磁盘快照](../windows/snapshot-copy-managed-disk.md)。
2. [将系统磁盘附加到由 BitLocker 加密的恢复 VM](troubleshoot-recovery-disks-portal-windows.md)。 如果要运行仅可在 BitLocker 加密的 VM 上使用的 [manage-bde](https://docs.microsoft.com/windows-server/administration/windows-commands/manage-bde) 命令，这是必需的。

    附加托管磁盘时，可能会收到“包含加密设置，因此不能用作数据磁盘”错误消息。 在此情况下，运行以下脚本，重试附加磁盘：

    ```Powershell
    $rgName = "myResourceGroup"
    $osDiskName = "ProblemOsDisk"

    New-AzDiskUpdateConfig -EncryptionSettingsEnabled $false |Update-AzDisk -diskName $osDiskName -ResourceGroupName $rgName

    $recoveryVMName = "myRecoveryVM" 
    $recoveryVMRG = "RecoveryVMRG" 
    $OSDisk = Get-AzDisk -ResourceGroupName $rgName -DiskName $osDiskName;

    $vm = get-AzVM -ResourceGroupName $recoveryVMRG -Name $recoveryVMName 

    Add-AzVMDataDisk -VM $vm -Name $osDiskName -ManagedDiskId $osDisk.Id -Caching None -Lun 3 -CreateOption Attach 

    Update-AzVM -VM $vm -ResourceGroupName $recoveryVMRG
    ```
     不能将托管磁盘附加到从 Blob 映像还原的 VM。

3. 附加磁盘后，对恢复 VM 进行远程桌面连接，以便可以运行某些 Azure PowerShell 脚本。 确保已在恢复 VM 上安装[最新版本的 Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)。

4. 打开提升的 Azure PowerShell 会话（以管理员身份运行）。 运行以下命令来登录到 Azure 订阅：

    ```Powershell
    Add-AzAccount -SubscriptionID [SubscriptionID]
    ```

5. 运行以下脚本来检查 BEK 文件的名称：

    ```powershell
    $vmName = "myVM"
    $vault = "myKeyVault"
    Get-AzureKeyVaultSecret -VaultName $vault | where {($_.Tags.MachineName -eq $vmName) -and ($_.ContentType -match 'BEK')} `
            | Sort-Object -Property Created `
            | ft  Created, `
                @{Label="Content Type";Expression={$_.ContentType}}, `
                @{Label ="Volume"; Expression = {$_.Tags.VolumeLetter}}, `
                @{Label ="DiskEncryptionKeyFileName"; Expression = {$_.Tags.DiskEncryptionKeyFileName}}
    ```

    下面是输出的示例。 找到附加磁盘的 BEK 文件名。 在此情况下，假定附加磁盘的驱动器号为 F，且 BEK 文件为 EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK。

    ```
    Created             Content Type Volume DiskEncryptionKeyFileName               
    -------             ------------ ------ -------------------------               
    4/5/2018 7:14:59 PM Wrapped BEK  C:\    B4B3E070-836C-4AF5-AC5B-66F6FDE6A971.BEK
    4/7/2018 7:21:16 PM Wrapped BEK  F:\    EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    4/7/2018 7:26:23 PM Wrapped BEK  G:\    70148178-6FAE-41EC-A05B-3431E6252539.BEK
    4/7/2018 7:26:26 PM Wrapped BEK  H:\    5745719F-4886-4940-9B51-C98AFABE5305.BEK
    ```

    如果看到两个重复的卷，具有较新时间戳的卷为恢复 VM 使用的当前 BEK 文件。

    如果“内容类型”值为“包装的 BEK”，请转到[密钥加密密钥 (KEK) 方案](#key-encryption-key-scenario)。

    获取驱动器的 BEK 文件名称后，须创建 secret-file-name.BEK 文件以解锁驱动器。 

6.  将 BEK 文件下载到恢复磁盘。 以下示例将 BEK 文件保存到 C:\BEK 文件夹。 运行脚本前，请确保 `C:\BEK\` 路径存在。

    ```powershell
    $vault = "myKeyVault"
    $bek = " EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK"
    $keyVaultSecret = Get-AzureKeyVaultSecret -VaultName $vault -Name $bek
    $bekSecretBase64 = $keyVaultSecret.SecretValueText
    $bekFileBytes = [Convert]::FromBase64String($bekSecretbase64)
    $path = "C:\BEK\DiskEncryptionKeyFileName.BEK"
    [System.IO.File]::WriteAllBytes($path,$bekFileBytes)
    ```

7.  若要使用 BEK 文件解锁附加磁盘，请运行以下命令：

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    ```
    在此示例中，附加的 OS 磁盘为驱动器 F。请确保使用正确的驱动器号。 

    - 如果使用 BEK 密钥成功解锁了磁盘。 可以认为 BItLocker 问题已解决。 

    - 如果使用 BEK 密钥未能解锁磁盘，则可以使用暂停保护，通过运行以下命令暂时关闭 BitLocker
    
        ```powershell
        manage-bde -protectors -disable F: -rc 0
        ```      
    - 如果要使用 dytem 磁盘重新生成 VM，必须完全解密驱动器。 为此，请运行以下命令：

        ```powershell
        manage-bde -off F:
        ```
8.  从恢复 VM 分离磁盘，然后将磁盘作为系统磁盘重新附加到受影响的 VM。 有关详细信息，请参阅[通过将 OS 磁盘附加到恢复 VM 来排查 Windows VM 相关问题](troubleshoot-recovery-disks-windows.md)。

### <a name="key-encryption-key-scenario"></a>密钥加密密钥方案

对于密钥加密密钥方案，请执行以下步骤：

1. 请确保登录的用户帐户需要“用户|密钥权限|加密操作|解包密钥”中 Key Vault 访问策略中的“解包”权限。
2. 将以下脚本保存到 .PS1 文件：

    ```powershell
    #Set the Parameters for the script
    param (
            [Parameter(Mandatory=$true)]
            [string] 
            $keyVaultName,
            [Parameter(Mandatory=$true)]
            [string] 
            $kekName,
            [Parameter(Mandatory=$true)]
            [string]
            $secretName,
            [Parameter(Mandatory=$true)]
            [string]
            $bekFilePath,
            [Parameter(Mandatory=$true)]
            [string] 
            $adTenant
            )
    # Load ADAL Assemblies. The following script assumes that the Azure PowerShell version you installed is 1.0.0. 
    $adal = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\1.0.0\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\1.0.0\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"
    [System.Reflection.Assembly]::LoadFrom($adal)
    [System.Reflection.Assembly]::LoadFrom($adalforms)

    # Set well-known client ID for AzurePowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2" 
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURI = "https://vault.azure.net"
    # Set Authority to Azure AD Tenant
    $authority = "https://login.windows.net/$adtenant"
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $authResult = $authContext.AcquireTokenAsync($resourceAppIdURI, $clientId, $redirectUri, $platformParameters).result
    # Generate auth header 
    $authHeader = $authResult.CreateAuthorizationHeader()
    # Set HTTP request headers to include Authorization header
    $headers = @{'x-ms-version'='2014-08-01';"Authorization" = $authHeader}

    ########################################################################################################################
    # 1. Retrieve wrapped BEK
    # 2. Make KeyVault REST API call to unwrap the BEK
    # 3. Convert the Base64Url string returned by KeyVault unwrap to Base64 string 
    # 4. Convert Base64 string to bytes and write to the BEK file
    ########################################################################################################################

    #Get wrapped BEK and place it in JSON object to send to KeyVault REST API
    $keyVaultSecret = Get-AzureKeyVaultSecret -VaultName $keyVaultName -Name $secretName
    $wrappedBekSecretBase64 = $keyVaultSecret.SecretValueText
    $jsonObject = @"
    {
    "alg": "RSA-OAEP",
    "value" : "$wrappedBekSecretBase64"
    }
    "@

    #Get KEK Url
    $kekUrl = (Get-AzureKeyVaultKey -VaultName $keyVaultName -Name $kekName).Key.Kid;
    $unwrapKeyRequestUrl = $kekUrl+ "/unwrapkey?api-version=2015-06-01";

    #Call KeyVault REST API to Unwrap 
    $result = Invoke-RestMethod -Method POST -Uri $unwrapKeyRequestUrl -Headers $headers -Body $jsonObject -ContentType "application/json" -Debug

    #Convert Base64Url string returned by KeyVault unwrap to Base64 string
    $base64UrlBek = $result.value;
    $base64Bek = $base64UrlBek.Replace('-', '+');
    $base64Bek = $base64Bek.Replace('_', '/');
    if($base64Bek.Length %4 -eq 2)
    {
        $base64Bek+= '==';
    }
    elseif($base64Bek.Length %4 -eq 3)
    {
        $base64Bek+= '=';
    }

    #Convert base64 string to bytes and write to BEK file
    $bekFileBytes = [System.Convert]::FromBase64String($base64Bek);
    [System.IO.File]::WriteAllBytes($bekFilePath,$bekFileBytes)
    ```
3. 设置参数。 该脚本处理 KEK 机密以创建 BEK 密钥，然后将其保存到恢复 VM 上的本地文件夹中。

4. 脚本开始时，将看到以下输出：

        GAC    Version        Location                                                                              
        ---    -------        --------                                                                              
        False  v4.0.30319     C:\Program Files\WindowsPowerShell\Modules\Az.Accounts\...
        False  v4.0.30319     C:\Program Files\WindowsPowerShell\Modules\Az.Accounts\...

    脚本完成后，将看到以下输出：

        VERBOSE: POST https://myvault.vault.azure.net/keys/rondomkey/<KEY-ID>/unwrapkey?api-
        version=2015-06-01 with -1-byte payload
        VERBOSE: received 360-byte response of content type application/json; charset=utf-8


5. 若要使用 BEK 文件解锁附加磁盘，请运行以下命令：

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    ```
    在此示例中，附加的 OS 磁盘为驱动器 F。请确保使用正确的驱动器号。 

    - 如果使用 BEK 密钥成功解锁了磁盘。 可以认为 BItLocker 问题已解决。 

    - 如果使用 BEK 密钥未能解锁磁盘，则可以使用暂停保护，通过运行以下命令暂时关闭 BitLocker
    
        ```powershell
        manage-bde -protectors -disable F: -rc 0
        ```      
    - 如果要使用 dytem 磁盘重新生成 VM，必须完全解密驱动器。 为此，请运行以下命令：

        ```powershell
        manage-bde -off F:
        ```

6. 从恢复 VM 分离磁盘，然后将磁盘作为系统磁盘重新附加到受影响的 VM。 有关详细信息，请参阅[通过将 OS 磁盘附加到恢复 VM 来排查 Windows VM 相关问题](troubleshoot-recovery-disks-windows.md)。
