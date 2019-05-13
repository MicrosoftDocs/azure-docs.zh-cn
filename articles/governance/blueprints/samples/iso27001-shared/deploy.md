---
title: 示例 - ISO 27001 共享服务蓝图 - 部署步骤
description: ISO 27001 共享服务蓝图示例的部署步骤。
author: DCtheGeek
ms.author: dacoulte
ms.date: 03/14/2019
ms.topic: sample
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: 88d5cfbbcb29cacc2e8c1c6a226367c5f23e8231
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64926297"
---
# <a name="deploy-the-iso-27001-shared-services-blueprint-sample"></a>部署 ISO 27001 共享服务蓝图示例

若要部署 Azure 蓝图 ISO 27001 共享服务蓝图示例，必须执行以下步骤：

> [!div class="checklist"]
> - 基于示例创建新的蓝图
> - 将示例副本标记为“已发布”
> - 将蓝图副本分配到现有的订阅

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free)。

## <a name="create-blueprint-from-sample"></a>基于示例创建蓝图

首先，通过使用示例作为起点在环境中创建新的蓝图，来实现蓝图示例。

1. 选择“所有服务”，然后在左窗格中搜索并选择“策略”。 在“策略”页上选择“蓝图”。

1. 在左侧的“开始”页中，选择“创建蓝图”下的“创建”按钮。

1. 在“其他示例”下找到“ISO 27001:共享服务”蓝图示例，然后选择“使用此示例”。

1. 输入该蓝图示例的“基本信息”：

   - **蓝图名称**：提供 ISO 27001 共享服务蓝图示例副本的名称。
   - **定义位置**：使用省略号并选择要将示例副本保存到的管理组。

1. 选择页面顶部的“项目”选项卡，或页面底部的“下一步:项目”。

1. 查看构成蓝图示例的项目列表。 许多项目包含稍后我们将要定义的参数。 查看完蓝图示例后，选择“保存草稿”。

## <a name="publish-the-sample-copy"></a>发布示例副本

现已在环境中创建蓝图示例的副本。 该副本在创建后处于“草稿”模式，必须先将其**发布**，然后才能分配和部署它。 可根据环境和需求自定义蓝图示例的副本，但这种修改可能会将该副本移出 ISO 27001 标准。

1. 选择“所有服务”，然后在左窗格中搜索并选择“策略”。 在“策略”页上选择“蓝图”。

1. 在左侧选择“蓝图定义”页。 使用筛选器找到蓝图示例的副本，然后选择它。

1. 选择页面顶部的“发布蓝图”。 在右侧的新窗格中，提供蓝图示例副本的**版本**。 以后做出修改时，此属性非常有用。 提供**更改注释**，例如，“基于 ISO 27001 蓝图示例发布的第一个版本”。 然后选择页面底部的“发布”。

## <a name="assign-the-sample-copy"></a>分配示例副本

成功**发布**蓝图示例的副本后，可将它分配到它所在的管理组中的某个订阅。 在此步骤中，需提供参数来使蓝图示例副本的每个部署保持唯一。

1. 选择“所有服务”，然后在左窗格中搜索并选择“策略”。 在“策略”页上选择“蓝图”。

1. 在左侧选择“蓝图定义”页。 使用筛选器找到蓝图示例的副本，然后选择它。

1. 选择蓝图定义页面顶部的“分配蓝图”。

1. 提供蓝图分配的参数值：

   - 基础

     - **订阅**：在蓝图示例副本所保存到的管理组中选择一个或多个订阅。 如果选择多个订阅，将使用输入的参数为每个订阅创建一个分配。
     - **分配名称**：系统会根据蓝图的名称预先填充该名称。
       请根据需要更改该名称，或保留原样。
     - **位置**：选择要在其中创建托管标识的区域。 Azure 蓝图使用此托管标识在分配的蓝图中部署所有项目。 若要了解详细信息，请参阅 [Azure 资源的托管标识](../../../../active-directory/managed-identities-azure-resources/overview.md)。
     - **蓝图定义版本**：选择蓝图示例副本的**已发布**版本。

   - 锁分配

     选择环境的蓝图锁定设置。 有关更多信息，请参阅[蓝图资源锁定](../../concepts/resource-locking.md)。

   - 托管标识

     保留默认的系统分配的托管标识选项。

   - 蓝图参数

     蓝图定义中的许多项目使用本部分定义的参数来提供一致性。

     - **组织名称**：输入组织的短名称。 此属性主要用于为资源命名。
     - **共享服务子网地址前缀**：提供用于将部署的资源联网到一起的 CIDR 表示法值。
     - **共享服务位置**：确定要将项目部署到的位置。 并非所有服务都可在所有位置使用。 部署此类服务的项目会针对该项目要部署到的位置提供一个参数选项。
     - **允许的位置(策略:ISO 27001 的蓝图计划)**：该值指示资源组和资源的允许位置。
     - **VM 代理的 Log Analytics 工作区(策略:ISO 27001 的蓝图计划)**：指定工作区的资源 ID。 此参数使用 `concat` 函数来构造资源 ID。

   - 项目参数

     在本部分定义的参数将应用到定义了这些参数的项目。 这些参数属于[动态参数](../../concepts/parameters.md#dynamic-parameters) ，因为它们是在分配蓝图期间定义的。 有关完整列表或项目参数及其说明，请参阅[项目参数表](#artifact-parameters-table) 。

1. 输入所有参数后，选择页面底部的“分配”。 随后将创建蓝图分配，并开始部署项目。 部署过程大约需要一小时。 若要检查部署状态，请打开蓝图分配。

> [!WARNING]
> Azure 蓝图服务和内置蓝图示例是**免费的**。 Azure 资源[按产品定价](https://azure.microsoft.com/pricing/) 。 使用[定价计算器](https://azure.microsoft.com/pricing/calculator/) 可以估算运行此蓝图示例部署的资源所需的成本。

## <a name="artifact-parameters-table"></a>项目参数表

下表提供了蓝图项目参数的列表：

|项目名称|项目类型|参数名称|说明|
|-|-|-|-|
|\[预览\]：为 Linux VM 规模集(VMSS)部署 Log Analytics 代理|策略分配|可选：支持将 Linux OS 添加到范围的 VM 映像列表|（可选）默认值为 _["none"]_。|
|\[预览\]：为 Linux VM 部署 Log Analytics 代理|策略分配|可选：支持将 Linux OS 添加到范围的 VM 映像列表|（可选）默认值为 _["none"]_。|
|\[预览\]：为 Windows VM 规模集(VMSS)部署 Log Analytics 代理|策略分配|可选：支持将 Windows OS 添加到范围的 VM 映像列表|（可选）默认值为 _["none"]_。|
|\[预览\]：为 Windows VM 部署 Log Analytics 代理|策略分配|可选：支持将 Windows OS 添加到范围的 VM 映像列表|（可选）默认值为 _["none"]_。|
|允许的资源类型|策略分配|允许的资源类型|允许部署的资源类型列表。 此列表包括共享服务中部署的所有资源类型。|
|允许的存储帐户 SKU|策略分配|允许的存储 SKU|允许的诊断日志存储帐户 SKU 列表。 默认值为 _["Standard_LRS"]_。|
|允许的虚拟机 SKU|策略分配|允许部署的虚拟机 SKU 列表。 默认值为 _["Standard_DS1_v2", "Standard_DS2_v2"]_。|
|ISO 27001 的蓝图计划|策略分配|用于审核诊断日志的资源类型|用于审核是否未启用诊断日志设置的资源类型列表。 [Azure Monitor 诊断日志架构](../../../../azure-monitor/platform/diagnostic-logs-schema.md#supported-log-categories-per-resource-type)中提供了可接受的值。|
|Log Analytics 资源组|资源组|名称|**已锁定** - 将**组织名称**与 `-sharedsvsc-log-rg` 相连接可使资源组名称保持唯一。|
|Log Analytics 资源组|资源组|位置|**已锁定** - 使用蓝图参数。|
|Log Analytics 模板|资源管理器模板|服务层|设置 Log Analytics 工作区的层。 默认值为 _PerNode_。|
|Log Analytics 模板|资源管理器模板|日志保留期(以天为单位)|日志保留期（以天为单位）。 默认值为 _365_。|
|Log Analytics 模板|资源管理器模板|位置|用于创建 Log Analytics 工作区的区域。 默认值为“美国西部 2”。|
|网络资源组|资源组|名称|**已锁定** - 将**组织名称**与 `-sharedsvcs-net-rg` 相连接可使资源组名称保持唯一。|
|网络资源组|资源组|位置|**已锁定** - 使用蓝图参数。|
|Azure 防火墙模板|资源管理器模板|Azure 防火墙专用 IP|配置 [Azure 防火墙](../../../../firewall/overview.md)的专用 IP。 此值也用作共享服务子网中的默认路由表。 应是“Azure 防火墙子网地址前缀”中定义的 CIDR 表示法的一部分。 默认值为 _10.0.4.4_。|
|Azure 防火墙模板|资源管理器模板|日志保留期(以天为单位)|日志保留期（以天为单位）。 默认值为 _365_。|
|网络安全组模板|资源管理器模板|日志保留期(以天为单位)|日志保留期（以天为单位）。 默认值为 _365_。|
|虚拟网络和路由表模板|资源管理器模板|虚拟网络地址前缀|虚拟网络的 CIDR 表示法。 默认值为 _10.0.0.0/16_。|
|虚拟网络和路由表模板|资源管理器模板|启用虚拟网络 DDoS 防护|为虚拟网络配置 DDoS 防护。 默认值为 _true_。|
|虚拟网络和路由表模板|资源管理器模板|共享服务子网地址前缀|共享服务子网的 CIDR 表示法。 默认值为 _10.0.0.0/24_。|
|虚拟网络和路由表模板|资源管理器模板|外围网络子网地址前缀|外围网络子网的 CIDR 表示法。 默认值为 _10.0.1.0/24_。|
|虚拟网络和路由表模板|资源管理器模板|应用程序网关子网地址前缀|应用程序网关子网的 CIDR 表示法。 默认值为 _10.0.2.0/24_。|
|虚拟网络和路由表模板|资源管理器模板|虚拟网络网关子网地址前缀|虚拟网络网关子网的 CIDR 表示法。 默认值为 _10.0.3.0/24_。|
|虚拟网络和路由表模板|资源管理器模板|Azure 防火墙子网地址前缀|[Azure 防火墙](../../../../firewall/overview.md)子网的 CIDR 表示法。 应包含“Azure 防火墙专用 IP”参数。|
|密钥保管库资源组|资源组|名称|**已锁定** - 将**组织名称**与 `-sharedsvcs-kv-rg` 相连接可使资源组名称保持唯一。|
|密钥保管库资源组|资源组|位置|**已锁定** - 使用蓝图参数。|
|Key Vault 模板|资源管理器模板|Jumpbox 管理员用户名|Jumpbox 的用户名。 必须与 **Jumpbox 模板**中的相同属性值相匹配。 默认值为 _jb-admin-user_。|
|Key Vault 模板|资源管理器模板|Jumpbox 管理员 SSH 密钥或密码|Jumpbox 上的帐户的密钥或密码。 必须与 **Jumpbox 模板**中的相同属性值相匹配。 无默认值，且不能留空。|
|Key Vault 模板|资源管理器模板|域管理员用户名|用于访问 Active Directory VM 以及将其他 VM 加入域的用户名。 必须与 **Active Directory 域服务模板**中的“域管理员用户”属性值相匹配。 默认值为 _domain-admin-user_。|
|Key Vault 模板|资源管理器模板|域管理员密码|域管理员用户的密码。 无默认值，且不能留空。|
|Key Vault 模板|资源管理器模板|AAD 对象 ID|需要访问 Key Vault 实例的帐户的 AAD 对象标识符。 无默认值，且不能留空。 若要在 Azure 门户中查找此值，请在“服务”下搜索并选择“用户”。 使用“名称”框筛选帐户名，并选择该帐户。 在“用户配置文件”页上，选择“对象 ID”旁边的“单击以复制”图标。  |
|Key Vault 模板|资源管理器模板|日志保留期(以天为单位)|日志保留期（以天为单位）。 默认值为 _365_。|
|Key Vault 模板|资源管理器模板|Key Vault SKU|指定创建的 Key Vault 的 SKU。 默认值为“高级”。|
|Jumpbox 资源组|资源组|名称|**已锁定** - 将**组织名称**与 `-sharedsvcs-jb-rg` 相连接可使资源组名称保持唯一。|
|Jumpbox 资源组|资源组|位置|**已锁定** - 使用蓝图参数。|
|Jumpbox 模板|资源管理器模板|Jumpbox 管理员用户名|用于访问 Jumpbox VM 的用户名。 必须与 **Key Vault 模板**中的相同属性值相匹配。 默认值为 _jb-admin-user_。|
|Jumpbox 模板|资源管理器模板|Jumpbox 管理员密码(Key Vault 资源 ID)|Key Vault 的资源 ID。 请使用 "/subscriptions/{subscriptionId}/resourceGroups/{orgName}-sharedsvcs-kv-rg/providers/Microsoft.KeyVault/vaults/{orgName}-sharedsvcs-kv"，并将 `{subscriptionId}` 替换为你的订阅 ID，将 `{orgName}` 替换为“组织名称”蓝图参数。|
|Jumpbox 模板|资源管理器模板|Jumpbox 管理员密码(Key Vault 机密名称)|Jumpbox 管理员的用户名。必须与 **Key Vault 模板**中的“Jumpbox 管理员用户名”属性值相匹配。|
|Jumpbox 模板|资源管理器模板|Jumpbox 操作系统|确定 Jumpbox VM 的操作系统。 默认值为 _Windows_。|
|Active Directory 域服务资源组|资源组|名称|**已锁定** - 将**组织名称**与 `-sharedsvcs-adds-rg` 相连接可使资源组名称保持唯一。|
|Active Directory 域服务资源组|资源组|位置|**已锁定** - 使用蓝图参数。|
|Active Directory 域服务模板|资源管理器模板|域管理员用户名|ADDS Jumpbox 的用户名。 必须与 **Key Vault 模板**中的相同属性值相匹配。 默认值为 _adds-admin-user_。|
|Active Directory 域服务模板|资源管理器模板|域管理员密码(Key Vault 资源 ID)|Key Vault 的资源 ID。 请使用 "/subscriptions/{subscriptionId}/resourceGroups/{orgName}-sharedsvcs-kv-rg/providers/Microsoft.KeyVault/vaults/{orgName}-sharedsvcs-kv"，并将 `{subscriptionId}` 替换为你的订阅 ID，将 `{orgName}` 替换为“组织名称”蓝图参数。|
|Active Directory 域服务模板|资源管理器模板|域管理员密码(Key Vault 机密名称)|域管理员的用户名。必须与 **Key Vault 模板**中的“域管理员用户名”属性值相匹配。|
|Active Directory 域服务模板|资源管理器模板|域名|示例创建的 Active Directory 的名称。 默认值为 _contoso.com_。|
|Active Directory 域服务模板|资源管理器模板|域管理员用户|用于 AD 管理员帐户以及将设备加入 AD 域的用户名。 必须与 **Key Vault 模板**中的“AD 管理员用户名”属性值相匹配。 默认值为 _domain-admin-user_。|
|Active Directory 域服务模板|资源管理器模板|域管理员密码|设置用于存储密码的 Key Vault 详细信息。 无默认值，且不能留空。|

## <a name="next-steps"></a>后续步骤

了解 ISO 27001 共享服务蓝图示例的部署步骤后，请访问以下文章来了解体系结构和控制映射：

> [!div class="nextstepaction"]
> [ISO 27001 共享服务蓝图 - 概述](./index.md)
> [ISO 27001 共享服务蓝图 - 控制映射](./control-mapping.md)

有关蓝图和如何使用这些蓝图的更多文章：

- 了解[蓝图生命周期](../../concepts/lifecycle.md)。
- 了解如何使用[静态和动态参数](../../concepts/parameters.md)。
- 了解如何自定义[蓝图排序顺序](../../concepts/sequencing-order.md)。
- 了解如何利用[蓝图资源锁定](../../concepts/resource-locking.md)。
- 了解如何[更新现有分配](../../how-to/update-existing-assignments.md)。
