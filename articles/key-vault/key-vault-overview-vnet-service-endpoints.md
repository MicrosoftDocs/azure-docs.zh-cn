---
title: Azure 密钥保管库的虚拟网络服务终结点 - Azure 密钥保管库 | Microsoft Docs
description: Key Vault 的虚拟网络服务终结点概述
services: key-vault
author: amitbapat
ms.author: ambapat
manager: barbkess
ms.date: 01/02/2019
ms.service: key-vault
ms.topic: conceptual
ms.openlocfilehash: d003e851dcfa77b18a67450946a71e4a13ce495c
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64700975"
---
# <a name="virtual-network-service-endpoints-for-azure-key-vault"></a>Azure Key Vault 的虚拟网络服务终结点

通过 Azure Key Vault 的虚拟网络服务终结点可将访问限制为指定虚拟网络。 此外，还可通过这些终结点将访问限制为一系列 IPv4（Internet 协议版本 4）地址范围。 任何从外部连接到 Key Vault 的用户都无法访问这些资源。

此限制有一个重要的例外情况。 若用户已选择允许受信任的 Microsoft 服务访问，则会允许来自这些服务的连接通过防火墙。 这些服务包括 Office 365 Exchange Online、Office 365 SharePoint Online、Azure 计算、Azure 资源管理器和 Azure 备份等。 此类用户仍需提供有效的 Azure Active Directory 令牌，并且必须具有执行所请求的操作的权限（配置为访问策略）。 有关详细信息，请参阅[虚拟网络服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md)。

## <a name="usage-scenarios"></a>使用方案

可以将 [Key Vault 防火墙和虚拟网络](key-vault-network-security.md)配置为默认拒绝访问来自所有网络的流量（包括 Internet 流量）。 可以向来自特定 Azure 虚拟网络和公共 Internet IP 地址范围的流量授予访问权限，为应用程序构建安全的网络边界。

> [!NOTE]
> Key Vault 防火墙和虚拟网络规则仅适用于 Key Vault [数据平面](../key-vault/key-vault-secure-your-key-vault.md#data-plane-access-control)。 Key Vault 控制平面操作（例如创建、删除和修改操作，设置访问策略，设置防火墙和虚拟网络规则）不受防火墙和虚拟网络规则的影响。

下面是此服务终结点的一些用法示例：

* 使用 Key Vault 存储加密密钥、应用程序机密和证书，并希望阻止从公共 Internet 访问 Key Vault。
* 你希望限制访问 Key Vault，以便只有你的应用程序或指定的少部分主机才能连接到 Key Vault。
* 你有一个在 Azure 虚拟网络中运行的应用程序，并且此虚拟网络限制了所有的入站和出站流量。 应用程序仍需连接到 Key Vault，以获取机密或证书，或者使用加密密钥。

## <a name="configure-key-vault-firewalls-and-virtual-networks"></a>配置 Key Vault 防火墙和虚拟网络

以下是配置防火墙和虚拟网络所需的步骤。 无论使用的是 PowerShell、Azure CLI 还是 Azure 门户，上述步骤均适用。

1. 启用 [Key Vault 日志记录](key-vault-logging.md)以查看详细的访问日志。 当防火墙和虚拟网络规则阻止访问 Key Vault 时，此操作有助于进行诊断。 （此为可选步骤，但强烈建议你执行。）
2. 为目标虚拟网络和子网启用“Key Vault 的服务终结点”。
3. 为 Key Vault 设置防火墙和虚拟网络规则，仅限特定虚拟网络、子网和 IPv4 地址范围能够访问该 Key Vault。
4. 如果需要所有受信任的 Microsoft 服务都能够访问此 Key Vault，则启用该选项，允许“受信任的 Azure 服务”连接到 Key Vault。

有关详细信息，请参阅[配置 Azure 密钥保管库防火墙和虚拟网络](key-vault-network-security.md)。

> [!IMPORTANT]
> 防火墙规则生效后，只在用户请求来自允许的虚拟网络或 IPv4 地址范围时，才能执行 Key Vault [数据平面](../key-vault/key-vault-secure-your-key-vault.md#data-plane-access-control)操作。 从 Azure 门户访问 Key Vault 时，这同样适用。 虽然用户可从 Azure 门户浏览到 Key Vault，但如果其客户端计算机不在允许列表中，则可能无法列出密钥、机密或证书。 这也会影响其他 Azure 服务的 Key Vault 选取器。 如果防火墙规则阻止了用户的客户端计算机，则用户可以查看 Key Vault 列表，但不能查看列表密钥。


> [!NOTE]
> 注意以下配置限制：
> * 最多允许 127 条虚拟网络规则和 127 条 IPv4 规则。 
> * 不支持使用“/31”或“/32”前缀大小的小型地址范围。 改为使用单独的 IP 地址规则配置这些范围。
> * IP 网络规则仅适用于公共 IP 地址。 IP 规则不允许为专用网络保留的 IP 地址范围（如 RFC 1918 中所定义）。 专用网络包括以开头的地址**10。**， **172.16 31**，并**192.168。**。 
> * 目前仅支持 IPv4 地址。

## <a name="trusted-services"></a>受信服务

以下是允许访问 Key Vault 的受信服务列表（前提是启用了“允许受信任的服务”选项）。

|受信服务|使用方案|
| --- | --- |
|Azure 虚拟机部署服务|[将证书从客户托管的 Key Vault 部署到 VM](https://blogs.technet.microsoft.com/kv/2016/09/14/updated-deploy-certificates-to-vms-from-customer-managed-key-vault/)。|
|Azure 资源管理器模板部署服务|[在部署期间传递安全值](../azure-resource-manager/resource-manager-keyvault-parameter.md)。|
|Azure 磁盘加密卷加密服务|允许在虚拟机部署期间访问 BitLocker 密钥 (Windows VM) 或 DM 密码 (Linux VM) 和密钥加密密钥。 这将启用 [Azure 磁盘加密](../security/azure-security-disk-encryption.md)。|
|Azure 备份|允许使用 [Azure 备份](../backup/backup-introduction-to-azure-backup.md)在 Azure 虚拟机备份期间备份和还原相关密钥和机密。|
|Exchange Online 和 SharePoint Online|允许使用[客户密钥](https://support.office.com/article/Controlling-your-data-in-Office-365-using-Customer-Key-f2cd475a-e592-46cf-80a3-1bfb0fa17697)访问 Azure 存储服务加密的客户密钥。|
|Azure 信息保护|允许访问 [Azure 信息保护](https://docs.microsoft.com/azure/information-protection/what-is-information-protection)的租户密钥。|
|Azure 应用服务|[通过 Key Vault 部署 Azure Web 应用证书](https://blogs.msdn.microsoft.com/appserviceteam/2016/05/24/deploying-azure-web-app-certificate-through-key-vault/)。|
|Azure SQL 数据库|[使用 Azure SQL 数据库和数据仓库的“创建自己的密钥”支持进行透明数据加密](../sql-database/transparent-data-encryption-byok-azure-sql.md?view=sql-server-2017&viewFallbackFrom=azuresqldb-current)。|
|Azure 存储|[在 Azure Key Vault 中使用客户托管密钥进行存储服务加密](../storage/common/storage-service-encryption-customer-managed-keys.md)。|
|Azure Data Lake Store|[在 Azure Data Lake Store 中使用客户托管密钥进行数据加密](../data-lake-store/data-lake-store-encryption.md)。|
|Azure databricks|[基于 Apache Spark 的快速、简单、协作分析服务](../azure-databricks/what-is-azure-databricks.md)|



> [!NOTE]
> 必须设置相关 Key Vault 访问策略，才能允许相应的服务访问 Key Vault。

## <a name="next-steps"></a>后续步骤

* [保护密钥保管库](key-vault-secure-your-key-vault.md)
* [配置 Azure Key Vault 防火墙和虚拟网络](key-vault-network-security.md)
