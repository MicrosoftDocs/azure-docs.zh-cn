---
title: 停止使用托管的标识 VM 扩展并开始使用 Azure 实例元数据服务终结点
description: 逐步介绍了如何停止使用 VM 扩展并开始使用 Azure 实例元数据服务 (IMDS) 进行身份验证步骤。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/25/2018
ms.author: markvi
ms.openlocfilehash: 6ee8891eae108256875660cc3f2256b65703a1aa
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406788"
---
# <a name="how-to-stop-using-the-virtual-machine-managed-identities-extension-and-start-using-the-azure-instance-metadata-service"></a>如何停止使用的虚拟机托管标识扩展并开始使用 Azure 实例元数据服务

## <a name="virtual-machine-extension-for-managed-identities"></a>适用于管理的标识虚拟机扩展

适用于管理的标识的虚拟机扩展用于请求的虚拟机中托管的标识令牌。 工作流包括以下步骤：

1. 首先，在资源中的工作负荷调用本地终结点`http://localhost/oauth2/token`请求访问令牌。
2. 虚拟机扩展使用的托管标识的凭据从 Azure AD 请求访问令牌... 
3. 访问令牌返回给调用方，并可用于向支持 Azure AD 身份验证，如 Azure 密钥保管库或 Azure 存储服务进行身份验证。

由于下一节中所述的几个限制，托管的标识 VM 扩展已不再支持使用等效的终结点在 Azure 实例元数据服务 (IMDS)

### <a name="provision-the-extension"></a>预配扩展 

在配置虚拟机或虚拟机规模集具有托管的标识时，您可以根据需要选择预配 Azure 资源 VM 扩展使用的托管的标识`-Type`参数上的[集 AzVMExtension](https://docs.microsoft.com/powershell/module/az.compute/set-azvmextension) cmdlet。 您可以传递`ManagedIdentityExtensionForWindows`或`ManagedIdentityExtensionForLinux`，具体取决于虚拟机的类型并将其使用命名`-Name`参数。 `-Settings` 参数指定 OAuth 令牌终结点用于令牌获取的端口：

```powershell
   $settings = @{ "port" = 50342 }
   Set-AzVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
```

此外可以使用 Azure 资源管理器部署模板以预配 VM 扩展添加到以下 JSON`resources`到模板的部分 (使用`ManagedIdentityExtensionForLinux`的 Linux 版本的名称和类型的元素)。

    ```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForWindows')]",
        "apiVersion": "2018-06-01",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Microsoft.ManagedIdentity",
            "type": "ManagedIdentityExtensionForWindows",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "port": 50342
            }
        }
    }
    ```
    
    
如果您正在使用虚拟机规模集，也可以预配托管的标识的扩展插件使用 Azure 资源虚拟机规模集[添加 AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) cmdlet。 您可以传递`ManagedIdentityExtensionForWindows`或`ManagedIdentityExtensionForLinux`，具体取决于虚拟机规模集的类型设置，并将其命名使用`-Name`参数。 `-Settings` 参数指定 OAuth 令牌终结点用于令牌获取的端口：

   ```powershell
   $setting = @{ "port" = 50342 }
   $vmss = Get-AzVmss
   Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Setting $settings 
   ```
若要预配虚拟机规模集扩展 Azure 资源管理器部署模板后，将添加到以下 JSON`extensionpProfile`到模板的部分 (使用`ManagedIdentityExtensionForLinux`的 Linux 版本的名称和类型的元素)。

    ```json
    "extensionProfile": {
        "extensions": [
            {
                "name": "ManagedIdentityWindowsExtension",
                "properties": {
                    "publisher": "Microsoft.ManagedIdentity",
                    "type": "ManagedIdentityExtensionForWindows",
                    "typeHandlerVersion": "1.0",
                    "autoUpgradeMinorVersion": true,
                    "settings": {
                        "port": 50342
                    },
                    "protectedSettings": {}
                }
            }
    ```

由于 DNS 查找失败，预配虚拟机扩展可能会失败。 如果发生这种情况，重新启动虚拟机，然后重试。 

### <a name="remove-the-extension"></a>删除扩展 
若要删除该扩展，请使用`-n ManagedIdentityExtensionForWindows`或`-n ManagedIdentityExtensionForLinux`开关 （具体取决于虚拟机的类型） 和[az vm 扩展删除](https://docs.microsoft.com/cli/azure/vm/)，或[az vmss 扩展删除](https://docs.microsoft.com/cli/azure/vmss)为虚拟机规模设置使用 Azure CLI 或`Remove-AzVMExtension`powershell:

```azurecli-interactive
az vm identity --resource-group myResourceGroup --vm-name myVm -n ManagedIdentityExtensionForWindows
```

```azurecli-interactive
az vmss extension delete -n ManagedIdentityExtensionForWindows -g myResourceGroup -vmss-name myVMSS
```

```powershell
Remove-AzVMExtension -ResourceGroupName myResourceGroup -Name "ManagedIdentityExtensionForWindows" -VMName myVM
```

### <a name="acquire-a-token-using-the-virtual-machine-extension"></a>使用虚拟机扩展获取标记

下面是使用创建 VM 扩展终结点的 Azure 资源管理的标识的示例请求：

```
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| 元素 | 描述 |
| ------- | ----------- |
| `GET` | HTTP 谓词，指示想要从终结点检索数据。 在本例中，该数据为 OAuth 访问令牌。 | 
| `http://localhost:50342/oauth2/token` | Azure 资源的托管标识终结点，其中 50342 是可配置的默认端口。 |
| `resource` | 一个查询字符串参数，表示目标资源的应用 ID URI。 它也会显示在所颁发令牌的 `aud`（受众）声明中。 本示例请求一个用于访问 Azure 资源管理器的、应用 ID URI 为 https://management.azure.com/ 的令牌。 |
| `Metadata` | 一个 HTTP 请求标头字段，Azure 资源的托管标识需要使用该元素来缓解服务器端请求伪造 (SSRF) 攻击。 必须将此值设置为“true”（全小写）。|
| `object_id` | （可选）一个查询字符串参数，指示要将此令牌用于的托管标识的 object_id。 如果 VM 有用户分配的多个托管标识，则为必需的。|
| `client_id` | （可选）一个查询字符串参数，指示要将此令牌用于的托管标识的 client_id。 如果 VM 有用户分配的多个托管标识，则为必需的。|


示例响应：

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| 元素 | 描述 |
| ------- | ----------- |
| `access_token` | 请求的访问令牌。 调用受保护 REST API 时，该令牌将作为“持有者”令牌嵌入在 `Authorization` 请求标头字段中，使 API 能够对调用方进行身份验证。 | 
| `refresh_token` | 未由 Azure 资源的托管标识使用。 |
| `expires_in` | 访问令牌在过期之前保持有效的秒数，从颁发时间开始算起。 可在令牌的 `iat` 声明中找到颁发时间。 |
| `expires_on` | 访问令牌过期的时间范围。 该日期表示为自“1970-01-01T0:0:0Z UTC”开始的秒数（对应于令牌的 `exp` 声明）。 |
| `not_before` | 访问令牌生效且可被接受的时间范围。 该日期表示为自“1970-01-01T0:0:0Z UTC”开始的秒数（对应于令牌的 `nbf` 声明）。 |
| `resource` | 请求访问令牌时所针对的资源，与请求的 `resource` 查询字符串参数匹配。 |
| `token_type` | 令牌的类型，这是一个“持有者”访问令牌，意味着资源可向此令牌的持有者授予访问权限。 |


### <a name="troubleshoot-the-virtual-machine-extension"></a>虚拟机扩展进行故障排除 

#### <a name="restart-the-virtual-machine-extension-after-a-failure"></a>在发生故障后重新启动虚拟机扩展

在 Windows 和某些 Linux 版本中，如果该扩展停止，可使用以下 cmdlet 手动重启该扩展：

```powershell
Set-AzVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

其中： 
- Windows 的扩展名称和类型是： `ManagedIdentityExtensionForWindows`
- 扩展名称和适用于 Linux 的类型是： `ManagedIdentityExtensionForLinux`

#### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>尝试 Azure 资源托管标识扩展的架构导出功能时，“自动化脚本”失败

时虚拟机上启用了 Azure 资源的管理的标识，尝试将"自动化脚本"功能用于虚拟机或其资源组时，会显示以下错误：

![Azure 资源托管标识自动化脚本导出错误](./media/howto-migrate-vm-extension/automation-script-export-error.png)

管理 Azure 资源虚拟机扩展的标识当前不支持将其架构导出到资源组模板的功能。 因此，生成的模板不显示用于在资源上启用 Azure 资源托管标识的配置参数。 按照中的示例，可以手动添加这些部分[配置托管在使用模板在 Azure 虚拟机上的 Azure 资源的标识](qs-configure-template-windows-vm.md)。

当架构导出功能变为可用于管理 Azure 资源 （已计划为在 2019 年 1 月中不推荐使用） 的虚拟机扩展的标识时，它将被列入[导出资源组包含 VM 扩展的](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

## <a name="limitations-of-the-virtual-machine-extension"></a>虚拟机扩展的限制 

有几个主要使用虚拟机扩展限制。 

 * 最严重的局限性在于用于请求令牌的凭据存储在虚拟机上。 如果攻击者成功地破坏了虚拟机可以盗取的凭据。 
 * 此外，虚拟机扩展是仍不受支持的多个 Linux 发行版，与修改、 生成和测试每个这些分发版上的扩展的成本的巨大开发。 目前，支持仅以下 Linux 分发版： 
    * CoreOS Stable
    * CentOS 7.1 
    * Red Hat 7.2 
    * Ubuntu 15.04 
    * Ubuntu 16.04
 * 没有性能影响到部署虚拟机与管理的标识，因为虚拟机扩展还必须将其预配。 
 * 最后，虚拟机扩展可以仅支持具有每个虚拟机 32 用户分配托管的标识。 

## <a name="azure-instance-metadata-service"></a>Azure 实例元数据服务

[Azure 实例元数据服务 (IMDS)](/azure/virtual-machines/windows/instance-metadata-service)是提供有关正在运行的可用于管理和配置虚拟机的虚拟机实例的信息的 REST 终结点。 终结点位于已知不可路由的 IP 地址 (`169.254.169.254`) 可仅从虚拟机中访问。

有以下几个以请求令牌使用 Azure IMDS 优点。 

1. 服务是外部的虚拟机，因此使用托管标识的凭据将不再存在于虚拟机上。 相反，它们托管和保护 Azure 虚拟机的主机上。   
2. 在 Azure IaaS 上受支持的所有 Windows 和 Linux 操作系统系统可以都使用管理的标识。
3. 部署是速度更快、 更轻松，因为不再需要将其预配 VM 扩展。
4. 使用 IMDS 终结点，最多 1000年用户分配管理的标识可以分配给单个虚拟机。
5. 而不所使用的虚拟机扩展使用 IMDS 请求没有明显变化，因此是相当简单的端口上当前使用的虚拟机扩展的现有部署。

出于这些原因，Azure IMDS 服务将以请求令牌的成为方法后的虚拟机扩展已弃用。 


## <a name="next-steps"></a>后续步骤

* [如何使用 Azure 虚拟机上的 Azure 资源管理的标识获取访问令牌](how-to-use-vm-token.md)
* [Azure 实例元数据服务](https://docs.microsoft.com/azure/virtual-machines/windows/instance-metadata-service)
