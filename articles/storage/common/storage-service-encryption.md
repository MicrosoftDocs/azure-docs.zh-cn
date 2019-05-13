---
title: 静态数据的 azure 存储加密 |Microsoft Docs
description: Azure 存储会自动对其进行加密来保护数据之前将其保存到云。 Azure 存储中的所有数据进行加密和解密以透明方式使用 256 位 AES 加密，并为符合 FIPS 140-2。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/30/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 6eb7de7810ce23aed4031cca9f038da7149a6f9c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153091"
---
# <a name="azure-storage-encryption-for-data-at-rest"></a>静态数据的 azure 存储加密

Azure 存储自动加密数据时将其保存到云。 加密可保护你的数据，并帮助您以满足组织的安全性和符合性承诺。 在 Azure 存储中的数据进行加密，以透明方式使用 256 位解密[AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)，另一个的最强块加密法，为符合 FIPS 140-2。 Azure 存储加密是类似于在 Windows 上的 BitLocker 加密。

Azure 存储加密的所有新的和现有存储帐户启用，并且不能禁用。 默认情况下保护数据，因为不需要修改代码或应用程序以充分利用 Azure 存储加密。 

存储帐户而不考虑其性能层 （标准或高级） 或部署模型 （Azure 资源管理器或经典） 进行加密。 所有 Azure 存储冗余选项都支持加密，并将加密的存储帐户的所有副本。 所有 Azure 存储资源进行都加密，包括 blob、 磁盘、 文件、 队列和表。 对象的所有元数据也会加密。

加密不会影响 Azure 存储性能。 没有为 Azure 存储加密无需额外付费。

有关基础 Azure 存储加密的加密模块的详细信息，请参阅[加密 API:下一代](https://docs.microsoft.com/windows/desktop/seccng/cng-portal)。

## <a name="key-management"></a>密钥管理

您可以依赖于 Microsoft 托管密钥进行加密的存储帐户，或使用你自己的密钥，与 Azure 密钥保管库一起加密。

### <a name="microsoft-managed-keys"></a>Microsoft 托管密钥

默认情况下，你的存储帐户使用 Microsoft 托管的加密密钥。 你可以查看加密设置中的存储帐户**加密**一部分[Azure 门户](https://portal.azure.com)下, 图中所示。

![查看帐户使用 Microsoft 托管密钥进行加密](media/storage-service-encryption/encryption-microsoft-managed-keys.png)

### <a name="customer-managed-keys"></a>客户托管密钥

你可以使用客户托管的密钥管理 Azure 存储加密。 客户托管密钥使更灵活地创建、 轮换、 禁用和撤消访问控制。 此外可以审核用于保护数据的加密密钥。 

使用 Azure 密钥保管库来管理你的密钥和审核密钥用法。 可以创建自己的密钥并将其存储在密钥保管库，或使用 Azure 密钥保管库 Api 来生成密钥。 存储帐户和 Key Vault 必须在同一个区域中，但可以在不同的订阅中。 有关 Azure 密钥保管库的详细信息，请参阅[什么是 Azure 密钥保管库？](../../key-vault/key-vault-overview.md)。

若要撤消对客户托管密钥的访问权限，请参阅[Azure 密钥保管库 PowerShell](https://docs.microsoft.com/powershell/module/azurerm.keyvault/)并[Azure Key Vault CLI](https://docs.microsoft.com/cli/azure/keyvault)。 有效地撤消访问权限阻止访问的存储帐户中的所有数据，因为加密密钥是由 Azure 存储不可访问。

若要了解如何将客户托管密钥用于 Azure 存储，请参阅以下文章之一：

- [配置为从 Azure 门户的 Azure 存储加密的客户托管密钥](storage-encryption-keys-portal.md)
- [配置为从 PowerShell 的 Azure 存储加密的客户托管密钥](storage-encryption-keys-powershell.md)
- [使用 Azure CLI 从 Azure 存储加密使用客户托管的密钥](storage-encryption-keys-cli.md)

> [!NOTE]  
> 不支持客户托管密钥[Azure 托管磁盘](../../virtual-machines/windows/managed-disks-overview.md)。

## <a name="azure-storage-encryption-versus-disk-encryption"></a>与磁盘加密的 azure 存储加密

使用 Azure 存储加密，所有 Azure 存储帐户和它们所包含的资源进行加密，包括备份 Azure 虚拟机磁盘的页 blob。 此外，可以使用加密 Azure 虚拟机磁盘[Azure 磁盘加密](../../security/azure-security-disk-encryption-overview.md)。 Azure 磁盘加密使用行业标准[BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview)在 Windows 上并[Dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt)提供与 Azure 密钥保管库集成的基于操作系统的加密解决方案在 Linux 上。

## <a name="next-steps"></a>后续步骤

- [什么是 Azure 密钥保管库？](../../key-vault/key-vault-overview.md)
