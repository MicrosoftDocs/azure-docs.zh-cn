---
title: 配置为从 Azure 门户的 Azure 存储加密的客户托管密钥
description: 了解如何使用 Azure 门户配置用于 Azure 存储加密的客户托管密钥。 客户托管密钥，可以创建、 轮换、 禁用和撤消访问控制。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/16/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: baabc5a8e1d063cb51a3edea3a7218591e85aa1a
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154152"
---
# <a name="configure-customer-managed-keys-for-azure-storage-encryption-from-the-azure-portal"></a>配置为从 Azure 门户的 Azure 存储加密的客户托管密钥

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

本文介绍如何使用客户托管密钥使用配置密钥保管库[Azure 门户](https://portal.azure.com/)。 若要了解如何创建密钥保管库使用 Azure 门户，请参阅[快速入门：设置和使用 Azure 门户的 Azure Key Vault 中检索机密](../../key-vault/quick-create-portal.md)。 


> [!IMPORTANT]
> 使用 Azure 存储加密使用客户托管密钥需要密钥保管库，有两个所需的属性配置，**软删除**并**不清除**。 在 Azure 门户中创建新的 key vault 时，默认情况下启用这些属性。 但是，如果你需要启用现有的密钥保管库上的这些属性，您必须使用 PowerShell 或 Azure CLI。

## <a name="enable-customer-managed-keys"></a>启用使用客户托管密钥

若要启用使用客户托管密钥在 Azure 门户中的，执行以下步骤：

1. 导航到存储帐户。
1. 在存储帐户的“设置”边栏选项卡上，单击“加密”。 选择“使用自己的密钥”选项，如下图所示。

    ![显示加密选项的门户屏幕截图](./media/storage-encryption-keys-portal/ssecmk1.png)

## <a name="specify-a-key"></a>指定的密钥

启用客户托管密钥后，将有机会来指定要将存储帐户相关联的密钥。

### <a name="specify-a-key-as-a-uri"></a>将密钥指定为 URI

若要将密钥指定为一个 URI，请按照下列步骤：

1. 若要在 Azure 门户中查找密钥 URI，导航到密钥保管库，并选择**密钥**设置。 选择所需的密钥，然后单击要查看其设置的键。 复制的值**密钥标识符**字段，提供 URI。

    ![屏幕截图显示密钥保管库密钥 URI](media/storage-encryption-keys-portal/key-uri-portal.png)

1. 在中**加密**设置为存储帐户，选择**输入密钥 URI**选项。
1. 在“密钥 URI”字段中，指定 URI。

   ![屏幕截图显示如何输入密钥 URI](./media/storage-encryption-keys-portal/ssecmk2.png)

### <a name="specify-a-key-from-a-key-vault"></a>从 Key Vault 指定密钥

若要指定从 key vault 的密钥，首先请确保您有包含密钥的密钥保管库。 若要指定从 key vault 的密钥，请按照下列步骤：

1. 选择“从 Key Vault 中选择”选项。
2. 选择包含要使用的密钥的 Key Vault。
3. 从 Key Vault 中选择密钥。

   ![显示客户托管密钥选项的屏幕截图](./media/storage-encryption-keys-portal/ssecmk3.png)

## <a name="update-the-key-version"></a>更新密钥的版本

创建密钥的新版本时，你将需要更新存储帐户以使用新版本。 执行以下步骤:

1. 导航到你的存储帐户并显示**加密**设置。
1. 指定新密钥版本的 URI。 或者，可以选择密钥保管库和再次要更新的版本的密钥。

## <a name="next-steps"></a>后续步骤

- [静态数据的 azure 存储加密](storage-service-encryption.md)
- [什么是 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)？
