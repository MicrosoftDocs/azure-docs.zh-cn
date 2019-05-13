---
title: 在 Azure HDInsight 上 Apache Kafka 的自带密钥
description: 本文介绍如何使用你在 Azure Key Vault 中的密钥加密 Azure HDInsight 上的 Apache Kafka 中存储的数据。
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 6108bfd9e39b37507ec7e113bf2c489e890f0ca0
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233563"
---
# <a name="bring-your-own-key-for-apache-kafka-on-azure-hdinsight"></a>在 Azure HDInsight 上 Apache Kafka 的自带密钥

Azure HDInsight 包括为 Apache Kafka 提供创建自己的密钥 (BYOK) 支持。 借助此功能，你可以拥有和管理用于加密静态数据的密钥。

通过 Azure 存储服务加密 (SSE) 保护 HDInsight 中的所有托管磁盘。 默认情况下，这些磁盘上的数据使用 Microsoft 托管密钥进行加密。 如果启用 BYOK，则会为 HDInsight 提供加密密钥，以便通过 Azure Key Vault 使用该密钥并对其进行管理。

BYOK 加密是在群集创建期间处理的一步过程，无需额外费用。 需要做的就是将 HDInsight 注册为 Azure Key Vault 的托管标识，并在创建群集时添加加密密钥。

所有发送到 Kafka 群集的消息（包括由 Kafka 维护的副本）都使用对称数据加密密钥 (DEK) 进行加密。 使用密钥保管库中的密钥加密密钥 (KEK) 保护 DEK。 加密和解密过程完全由 Azure HDInsight 处理。 

可以使用 Azure 门户或 Azure CLI 安全地旋转密钥保管库中的密钥。 当密钥旋转时，HDInsight Kafka 群集会在几分钟内开始使用新密钥。 启用"软删除"密钥保护功能可防止勒索软件的方案和意外删除。 密钥保管库不支持此保护功能没有。

## <a name="get-started-with-byok"></a>BYOK 入门
若要创建 BYOK 启用 Kafka 群集，我们会完成以下步骤：
1. 创建 Azure 资源的管理的标识
2. 设置 Azure 密钥保管库和密钥
3. 使用 BYOK 启用创建 HDInsight Kafka 群集
4. 轮换加密密钥

## <a name="create-managed-identities-for-azure-resources"></a>创建 Azure 资源的管理的标识

   若要向密钥保管库进行身份验证，使用以下命令创建用户分配托管的标识[Azure 门户](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md)， [Azure PowerShell](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)， [Azure 资源管理器](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md)，或[Azure CLI](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)。 有关如何将托管在 Azure HDInsight 中的标识工作的详细信息，请参阅[托管在 Azure HDInsight 中的标识](../hdinsight-managed-identities.md)。 虽然托管标识和适用于 Kafka 的 BYOK 需要 Azure Active Directory，但不要求使用企业安全性套餐 (ESP)。 将托管标识资源 ID 添加到 Key Vault 访问策略时，请务必保存该 ID。

   ![在 Azure 门户中创建用户分配的托管标识](./media/apache-kafka-byok/user-managed-identity-portal.png)

## <a name="setup-the-key-vault-and-keys"></a>设置密钥保管库和密钥

   HDInsight 仅支持 Azure Key Vault。 如果拥有自己的密钥保管库，则可以将密钥导入 Azure Key Vault。 请记住，密钥必须具有"软删除"。 "软删除"功能是可通过 REST、.NET /C#，PowerShell 和 Azure CLI 接口。

   1. 若要创建新的密钥保管库，请按照 [Azure Key Vault](../../key-vault/key-vault-overview.md) 快速入门进行操作。 有关导入现有密钥的详细信息，请访问[关于密钥、机密和证书](../../key-vault/about-keys-secrets-and-certificates.md)。

   2. 使用密钥保管库上启用"软删除" [az keyvault update](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-update) cli 命令。
        ```Azure CLI az keyvault update --name <Key Vault Name> --enable-soft-delete
        ```

   3. Create keys

        a. To create a new key, select **Generate/Import** from the **Keys** menu under **Settings**.

        ![Generate a new key in Azure Key Vault](./media/apache-kafka-byok/kafka-create-new-key.png)

        b. Set **Options** to **Generate** and give the key a name.

        ![Generate a new key in Azure Key Vault](./media/apache-kafka-byok/kafka-create-a-key.png)

        c. Select the key you created from the list of keys.

        ![Azure Key Vault key list](./media/apache-kafka-byok/kafka-key-vault-key-list.png)

        d. When you use your own key for Kafka cluster encryption, you need to provide the key URI. Copy the **Key identifier** and save it somewhere until you're ready to create your cluster.

        ![Copy key identifier](./media/apache-kafka-byok/kafka-get-key-identifier.png)
   
    4. Add managed identity to the key vault access policy.

        a. Create a new Azure Key Vault access policy.

        ![Create new Azure Key Vault access policy](./media/apache-kafka-byok/add-key-vault-access-policy.png)

        b. Under **Select Principal**, choose the user-assigned managed identity you created.

        ![Set Select Principal for Azure Key Vault access policy](./media/apache-kafka-byok/add-key-vault-access-policy-select-principal.png)

        c. Set **Key Permissions** to **Get**, **Unwrap Key**, and **Wrap Key**.

        ![Set Key Permissions for Azure Key Vault access policy](./media/apache-kafka-byok/add-key-vault-access-policy-keys.png)

        d. Set **Secret Permissions** to **Get**, **Set**, and **Delete**.

        ![Set Key Permissions for Azure Key Vault access policy](./media/apache-kafka-byok/add-key-vault-access-policy-secrets.png)

        e. Click on **Save**. 

        ![Save Azure Key Vault access policy](./media/apache-kafka-byok/add-key-vault-access-policy-save.png)

## Create HDInsight cluster

   You're now ready to create a new HDInsight cluster. BYOK can only be applied to new clusters during cluster creation. Encryption can't be removed from BYOK clusters, and BYOK can't be added to existing clusters.

   ![Kafka disk encryption in Azure portal](./media/apache-kafka-byok/apache-kafka-byok-portal.png)

   During cluster creation, provide the full key URL, including the key version. For example, `https://contoso-kv.vault.azure.net/keys/kafkaClusterKey/46ab702136bc4b229f8b10e8c2997fa4`. You also need to assign the managed identity to the cluster and provide the key URI.

## Rotating the Encryption key
   There might be scenarios where you might want to change the encryption keys used by the Kafka cluster after it has been created. This can be easily via the portal. For this operation, the cluster must have access to both the current key and the intended new key, otherwise the rotate key operation will fail.

   To rotate the key, you must have the full url of the new key (See Step 3 of [Setup the Key Vault and Keys](#setup-the-key-vault-and-keys)). Once you have that, go to the Kafka cluster properties section in the portal and click on **Change Key** under **Disk Encryption Key URL**. Enter in the new key url and submit to rotate the key.

   ![Kafka rotate disk encryption key](./media/apache-kafka-byok/kafka-change-key.png)

## FAQ for BYOK to Apache Kafka

**How does the Kafka cluster access my key vault?**

   Associate a managed identity with the HDInsight Kafka cluster during cluster creation. This managed identity can be created before or during cluster creation. You also need to grant the managed identity access to the key vault where the key is stored.

**Is this feature available for all Kafka clusters on HDInsight?**

   BYOK encryption is only possible for Kafka 1.1 and above clusters.

**Can I have different keys for different topics/partitions?**

   No, all managed disks in the cluster are encrypted by the same key.

**What happens if the cluster loses access to the key vault or the key?**
   If the cluster loses access to the key, warnings will be shown in the Apache Ambari portal. In this state, the **Change Key** operation will fail. Once key access is restored, Ambari warnings will go away and operations such as key rotation can be successfully performed.

   ![Kafka key access Ambari alert](./media/apache-kafka-byok/kafka-byok-ambari-alert.png)

**How can I recover the cluster if the keys are deleted?**

   Since only “Soft Delete” enabled keys are supported, if the keys are recovered in the key vault, the cluster should regain access to the keys. To recover an Azure Key Vault key, see [Undo-AzKeyVaultKeyRemoval](/powershell/module/az.keyvault/Undo-AzKeyVaultKeyRemoval) or [az-keyvault-key-recover](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-recover).

**Can I have producer/consumer applications working with a BYOK cluster and a non-BYOK cluster simultaneously?**

   Yes. The use of BYOK is transparent to producer/consumer applications. Encryption happens at the OS layer. No changes need to be made to existing producer/consumer Kafka applications.

**Are OS disks/Resource disks also encrypted?**

   No. OS disks and Resource disks are not encrypted.

**If a cluster is scaled up, will the new brokers support BYOK seamlessly?**

   Yes. The cluster needs access to the key in the key vault during scale up. The same key is used to encrypt all managed disks in the cluster.

**Is BYOK available in my location?**

   Kafka BYOK is available in all public clouds.

## Next steps

* For more information about Azure Key Vault, see [What is Azure Key Vault](../../key-vault/key-vault-whatis.md)?
* To get started with Azure Key Vault, see [Getting Started with Azure Key Vault](../../key-vault/key-vault-overview.md).
