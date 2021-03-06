---
title: 常见问题解答 - 适用于 Linux VM 的 Azure 磁盘加密
description: 本文提供有关适用于 Linux IaaS VM 的 Microsoft Azure 磁盘加密的常见问题解答。
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 06/05/2019
ms.custom: seodec18
ms.openlocfilehash: 9d62195759de2d43e863c1cb2c4c3ef911bb52c6
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498517"
---
# <a name="azure-disk-encryption-for-linux-virtual-machines-faq"></a>适用于 Linux 虚拟机的 Azure 磁盘加密常见问题解答

本文提供有关适用于 Linux 虚拟机 (VM) 的 Azure 磁盘加密的常见问题解答 (FAQ)。 有关此服务的详细信息，请参阅 [Azure 磁盘加密概述](disk-encryption-overview.md)。

## <a name="what-is-azure-disk-encryption-for-linux-vms"></a>什么是适用于 Linux VM 的 Azure 磁盘加密？

适用于 Linux VM 的 Azure 磁盘加密使用 Linux 的 dm-crypt 功能为 OS 磁盘* 和数据磁盘提供全磁盘加密。 此外，它还在使用 [EncryptFormatAll 功能](disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms)时提供临时磁盘加密。 加密内容从 VM 流向存储后端。 因此，使用客户托管密钥提供端对端加密。
 
请参阅[支持的 VM 和操作系统](disk-encryption-overview.md#supported-vms-and-operating-systems)。

## <a name="where-is-azure-disk-encryption-in-general-availability-ga"></a>哪里有正式发布版 (GA) 的 Azure 磁盘加密？

适用于 Linux VM 的 Azure 磁盘加密已在所有 Azure 公共区域正式发布。

## <a name="what-user-experiences-are-available-with-azure-disk-encryption"></a>Azure 磁盘加密提供哪些用户体验？

Azure 磁盘加密正式版支持 Azure 资源管理器模板、Azure PowerShell 和 Azure CLI。 不同的用户体验提供了灵活性。 可以通过三个不同的选项为 VM 启用磁盘加密。 有关 Azure 磁盘加密中提供的用户体验详细信息和分步指南，请参阅[适用于 Linux 的 Azure 磁盘加密方案](disk-encryption-linux.md)。

## <a name="how-much-does-azure-disk-encryption-cost"></a>Azure 磁盘加密如何收费？

使用 Azure 磁盘加密来加密 VM 磁盘是免费的，但使用与 Azure Key Vault 相关联的内容则会产生费用。 有关 Azure Key Vault 成本的详细信息，请参阅 [Key Vault 定价](https://azure.microsoft.com/pricing/details/key-vault/)页面。

## <a name="how-can-i-start-using-azure-disk-encryption"></a>如何开始使用 Azure 磁盘加密？

若要开始，请参阅 [Azure 磁盘加密概述](disk-encryption-overview.md)。

## <a name="what-vm-sizes-and-operating-systems-support-azure-disk-encryption"></a>哪些 VM 大小和操作系统支持 Azure 磁盘加密？

[Azure 磁盘加密概述](disk-encryption-overview.md)一文列出了支持 Azure 磁盘加密的 [VM 大小](disk-encryption-overview.md#supported-vms)和 [VM 操作系统](disk-encryption-overview.md#supported-operating-systems)。

## <a name="can-i-encrypt-both-boot-and-data-volumes-with-azure-disk-encryption"></a>是否可以使用 Azure 磁盘加密来加密引导卷和数据卷？

是的，可以同时加密引导卷和数据卷，也可以在不先加密 OS 卷的情况下加密数据卷。 

加密 OS 卷之后，不支持在 OS 卷上禁用加密。 如果 Linux VM 位于规模集中，则只能加密数据卷。

## <a name="can-i-encrypt-an-unmounted-volume-with-azure-disk-encryption"></a>我可以使用 Azure 磁盘加密来加密未装入的卷吗？

不可以，Azure 磁盘加密只加密已装入的卷。

## <a name="what-is-storage-server-side-encryption"></a>什么是存储服务器端加密？

存储服务器端加密会在 Azure 存储中加密 Azure 托管磁盘。 默认情况下，托管磁盘使用平台托管密钥通过服务器端加密进行加密（从 2017 年 6 月 10 日开始）。 指定一个由客户托管的密钥，即可实现对使用自己的密钥加密托管磁盘的管理。 有关详细信息，请参阅：[Azure 托管磁盘的服务器端加密](../disk-encryption.md)。
 
## <a name="how-is-azure-disk-encryption-different-from-storage-server-side-encryption-with-customer-managed-key-and-when-should-i-use-each-solution"></a>Azure 磁盘加密与使用客户托管的密钥的存储服务器端加密有何不同，分别应于何时使用这两种解决方案？

Azure 磁盘加密使用客户托管的密钥提供对 OS 磁盘、数据磁盘和临时磁盘的端对端加密。
- 如果你的要求包括对上述各项加密和端到端加密，请使用 Azure 磁盘加密。 
- 如果你的要求是使用客户托管的密钥仅对静态数据加密，请采用[使用客户托管密钥的服务器端加密](../disk-encryption.md)。 不能既使用 Azure 磁盘加密又使用采用了客户托管密钥的存储服务器端加密来加密磁盘。 
- 如果你的 Linux 发行版未列在[支持 Azure 磁盘加密的操作系统](disk-encryption-overview.md#supported-operating-systems)下，或者使用的是在 [Windows 不支持的方案](disk-encryption-linux.md#unsupported-scenarios)中调出的方案，请考虑使用[客户托管密钥的服务器端加密](../disk-encryption.md)。
- 如果组织的策略允许你使用 Azure 托管密钥加密静态内容，则无需执行任何操作，因为系统默认加密这些内容。 对于托管磁盘而言，默认在服务器端加密中使用平台托管密钥来加密存储内的内容。 该密钥是由 Azure 存储服务托管的。 



## <a name="how-do-i-rotate-secrets-or-encryption-keys"></a>我如何轮换机密或加密密钥？

若要轮换机密，只需调用你一开始在启用磁盘加密时使用的命令并指定另一 Key Vault 即可。 若要轮换密钥加密密钥，只需调用你一开始在启用磁盘加密时使用的命令并指定新的密钥加密方法即可。 

>[!WARNING]
> - 如果之前是通过指定 Azure AD 凭据使用 [Azure 磁盘加密与 Azure AD 应用](disk-encryption-linux-aad.md)选项来加密此 VM，则必须继续使用此选项来加密 VM。 无法在此加密的 VM 上使用 Azure 磁盘加密，因为不支持此方案，这意味着尚不支持为此加密的 VM 实施 AAD 应用程序切换操作。

## <a name="how-do-i-add-or-remove-a-key-encryption-key-if-i-didnt-originally-use-one"></a>如何在一开始并没有使用密钥加密密钥的情况下添加或删除该密钥？

若要添加密钥加密密钥，请再次调用 enable 命令，传递密钥加密密钥参数。 若要删除密钥加密密钥，请在没有密钥加密密钥参数的情况下再次调用 enable 命令。

## <a name="does-azure-disk-encryption-allow-you-to-bring-your-own-key-byok"></a>Azure 磁盘加密是否支持自带秘钥 (BYOK)？

是的，可以提供自己的密钥加密密钥。 这些密钥在 Azure Key Vault（Azure 磁盘加密的密钥存储）中受保护。 有关密钥加密密钥支持方案的详细信息，请参阅[创建和配置用于 Azure 磁盘加密的 Key Vault](disk-encryption-key-vault.md)。

## <a name="can-i-use-an-azure-created-key-encryption-key"></a>是否可以使用 Azure 创建的密钥加密密钥？

是的，可以使用 Azure Key Vault 来生成密钥加密密钥供 Azure 磁盘加密使用。 这些密钥在 Azure Key Vault（Azure 磁盘加密的密钥存储）中受保护。 有关密钥加密密钥的详细信息，请参阅[创建和配置用于 Azure 磁盘加密的 Key Vault](disk-encryption-key-vault.md)。

## <a name="can-i-use-an-on-premises-key-management-service-or-hsm-to-safeguard-the-encryption-keys"></a>是否可以使用本地密钥管理服务或 HSM 来保护加密密钥？

无法使用本地密钥管理服务或 HSM 来配合 Azure 磁盘加密保护加密密钥。 只能使用 Azure Key Vault 服务来保护加密密钥。 有关密钥加密密钥支持方案的详细信息，请参阅[创建和配置用于 Azure 磁盘加密的 Key Vault](disk-encryption-key-vault.md)。

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption"></a>配置 Azure 磁盘加密的先决条件是什么？

Azure 磁盘加密具有先决条件。 若要创建新的 Key Vault 或设置现有 Key Vault 进行磁盘加密访问，以启用加密并保护机密和密钥，请参阅[创建和配置用于 Azure 磁盘加密的 Key Vault](disk-encryption-key-vault.md)一文。 有关密钥加密密钥支持方案的详细信息，请参阅[创建和配置用于 Azure 磁盘加密的 Key Vault](disk-encryption-key-vault.md)。

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption-with-an-azure-ad-app-previous-release"></a>使用 Azure AD 应用（早期版本）配置 Azure 磁盘加密的先决条件是什么？

Azure 磁盘加密具有先决条件。 请参阅[使用 Azure AD 的 Azure 磁盘加密](disk-encryption-linux-aad.md)内容，创建 Azure Active Directory 应用程序、创建新的 Key Vault 或设置现有 Key Vault 进行磁盘加密访问，以启用加密并保护机密和密钥。 有关密钥加密密钥支持方案的详细信息，请参阅[创建和配置可将 Azure 磁盘加密和 Azure AD 配合使用的 Key Vault](disk-encryption-key-vault-aad.md)。

## <a name="is-azure-disk-encryption-using-an-azure-ad-app-previous-release-still-supported"></a>是否仍然支持使用 Azure AD 应用（早期版本）进行 Azure 磁盘加密？
是的。 仍然支持使用 Azure AD 应用进行磁盘加密。 不过，当加密新的 VM 时，建议使用新方法而不是使用 Azure AD 应用进行加密。 

## <a name="can-i-migrate-vms-that-were-encrypted-with-an-azure-ad-app-to-encryption-without-an-azure-ad-app"></a>是否可以在不使用 Azure AD 应用的情况下将通过 Azure AD 应用加密的 VM 迁移到此加密？
  当前，对于通过 Azure AD 应用加密的计算机，没有直接迁移路径可用来在不使用 Azure AD 应用的情况下迁移到此加密。 此外，也没有直接路径用来将未使用 Azure AD 应用的加密迁移到使用 AD 应用的加密。 

## <a name="what-version-of-azure-powershell-does-azure-disk-encryption-support"></a>Azure 磁盘加密支持哪些 Azure PowerShell 版本？

使用最新版的 Azure PowerShell SDK 来配置 Azure 磁盘加密。 下载最新版本的 [Azure PowerShell](https://github.com/Azure/azure-powershell/releases)。 Azure SDK 版本 1.1.0 不支持 Azure 磁盘加密。

> [!NOTE]
> Linux Azure 磁盘加密预览扩展“Microsoft.OSTCExtension.AzureDiskEncryptionForLinux”已弃用。 发布的该扩展适用于 Azure 磁盘加密预览版。 不应将预览版扩展用于测试或生产性部署。

> 使用 Azure 资源管理器 (ARM) 之类的部署方案时，需要部署适用于 Linux VM 的 Azure 磁盘加密扩展，以便在 Linux IaaS VM 上启用加密，因此必须使用 Azure 磁盘加密生产版支持的扩展“Microsoft.Azure.Security.AzureDiskEncryptionForLinux”。

## <a name="can-i-apply-azure-disk-encryption-on-my-custom-linux-image"></a>是否可对自定义 Linux 映像应用 Azure 磁盘加密？

不能对自定义 Linux 映像应用 Azure 磁盘加密。 仅支持上述受支持分发版的 Linux 库映像。 目前不支持自定义 Linux 映像。

## <a name="can-i-apply-updates-to-a-linux-red-hat-vm-that-uses-the-yum-update"></a>是否可以向使用 yum 更新的 Linux Red Hat VM 应用更新？

是的，可以在 Red Hat Linux VM 上执行 yum 更新。  有关详细信息，请参阅[隔离网络上的 Azure 磁盘加密](disk-encryption-isolated-network.md)。

## <a name="what-is-the-recommended-azure-disk-encryption-workflow-for-linux"></a>对于 Linux，应使用哪种 Azure 磁盘加密工作流？

为在 Linux 上获得最佳结果，建议使用以下工作流：
* 从与所需的 OS 发行版和版本相对应的未修改存储库映像启动
* 备份要加密的任何已装载的驱动器。  使用此备份，在失败时能够进行恢复，例如当 VM 在加密完成前重启时。
* 加密（可能需要花费几小时甚至几天，具体取决于 VM 特征和所附加的任何数据磁盘的大小）
* 根据需要自定义软件，并将其添加到映像。

如果此工作流不可用，可在平台存储帐户层使用[存储服务加密](../../storage/common/storage-service-encryption.md) (SSE)，作为通过 dm-crypt 实现完整磁盘加密的一个替代方法。

## <a name="what-is-the-disk-bek-volume-or-mntazure_bek_disk"></a>磁盘“Bek 卷”或“/mnt/azure_bek_disk”是什么？

“Bek 卷”是一个本地数据卷，可以安全地存储用于已加密 Azure VM 的加密密钥。
> [!NOTE]
> 请勿删除或编辑此磁盘中的任何内容。 请勿卸载磁盘，因为 IaaS VM 上的任何加密操作都需要有加密密钥才能执行。


## <a name="what-encryption-method-does-azure-disk-encryption-use"></a>Azure 磁盘加密使用何种加密方法？

Azure 磁盘加密可将 aes-xts-plain64 的 decrypt 默认方法和 256 位卷主密钥配合使用。

## <a name="if-i-use-encryptformatall-and-specify-all-volume-types-will-it-erase-the-data-on-the-data-drives-that-we-already-encrypted"></a>如果我使用 EncryptFormatAll 并指定了所有卷类型，它是否会擦除我们已加密的数据驱动器上的数据？
否，不会擦除已使用 Azure 磁盘加密进行了加密的数据驱动器上的数据。 与 EncryptFormatAll 不重新加密 OS 驱动器类似，它也不会重新加密已加密的数据驱动器。 有关详细信息，请参阅 [EncryptFormatAll 条件](disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms)。        

## <a name="is-xfs-filesystem-supported"></a>是否支持 XFS 文件系统？
支持加密 XFS OS 磁盘。

仅当使用 EncryptFormatAll 参数时，才支持加密 XFS 数据磁盘。 该操作将重格式化卷，并清除卷中所有数据。 有关详细信息，请参阅 [EncryptFormatAll 条件](disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms)。

## <a name="can-i-backup-and-restore-an-encrypted-vm"></a>我可以备份和还原已加密的 VM 吗？ 

Azure 备份提供一个机制，可以用来备份和还原同一订阅与区域中的已加密 VM。  相关说明，请参阅[通过 Azure 备份来备份和还原加密的虚拟机](../../backup/backup-azure-vms-encryption.md)。  目前不支持将已加密的 VM 还原到另一区域。  

## <a name="where-can-i-go-to-ask-questions-or-provide-feedback"></a>可以在何处提问或提供反馈？

你可以在 [Microsoft Q&A 的 Azure 磁盘加密问题页面](/answers/topics/azure-disk-encryption.html)提问或提供反馈。

## <a name="next-steps"></a>后续步骤
本文档详细描述了有关 Azure 磁盘加密的最常见问题。 有关此服务的详细信息，请参阅以下文章：

- [Azure 磁盘加密概述](disk-encryption-overview.md)
- [在 Azure 安全中心应用磁盘加密](../../security-center/asset-inventory.md)
- [Azure 静态数据加密](../../security/fundamentals/encryption-atrest.md)