---
title: Azure 存储的高级威胁防护
description: 配置 Azure 存储高级威胁防护以检测帐户活动中的异常情况，并通知用户可能有害的访问帐户尝试。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/03/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.openlocfilehash: ed263fec271801ac9c46bbc3125c71ca15b9a330
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65153977"
---
# <a name="advanced-threat-protection-for-azure-storage"></a>Azure 存储的高级威胁防护

Azure 存储高级威胁防护提供额外的一层安全智能，用于检测试图访问或利用存储帐户的异常或可能有害的企图。 借助此保护层，用户无需成为安全专家，也无需管理安全监视系统便可以解决威胁。 

出现在活动中的异常情况时触发安全警报。  这些安全警报集成在一起[Azure 安全中心](https://azure.microsoft.com/services/security-center/)，还将发送通过电子邮件到订阅管理员，具有可疑活动和如何调查和修正的威胁的建议的详细信息。

> [!NOTE]
> * 目前仅适用于 Blob 存储的 Azure 存储高级的威胁防护。
> * 有关定价详细信息，包括 30 天免费试用版，请参阅[Azure 安全中心定价页]( https://azure.microsoft.com/pricing/details/security-center/)。
> * Azure 存储功能的 ATP 目前不在 Azure 政府版和主权云区域中可用。

Azure 存储的高级的威胁防护引入诊断日志的读取、 写入和删除 Blob 存储进行威胁检测到的请求。 若要调查的警报从高级威胁防护，可以查看相关的存储活动使用存储分析日志记录。 有关详细信息，请参阅如何[配置存储分析日志记录](storage-monitor-storage-account.md#configure-logging)。

## <a name="set-up-advanced-threat-protection"></a>设置高级威胁防护 

### <a name="using-the-portal"></a>使用门户

1. 在 [https://portal.azure.com](https://portal.azure.com/) 中启动 Azure 门户。

2. 导航到想要保护的 Azure 存储帐户的配置页面。 在“设置”页面中，选择“高级威胁防护”。

3. 在“高级威胁防护”配置边栏选项卡中
    * **打开**高级*威胁防护*
    * 单击“保存”以保存新的或更新的高级威胁防护策略。 （图中的价格是例如目的。）

![打开 Azure 存储高级威胁防护](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-turn-on.png)

### <a name="using-azure-security-center"></a>使用 Azure 安全中心
当你订阅到标准层在 Azure 安全中心高级威胁防护设置你的存储帐户上。 有关详细信息请参阅[升级到安全中心标准层以增强安全性](https://docs.microsoft.com/azure/security-center/security-center-pricing)。 （图中的价格是例如目的。）

![在 ASC 中的标准层](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-pricing.png)

### <a name="using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板

使用 Azure 资源管理器模板部署具有高级威胁防护的 Azure 存储帐户已启用。
有关详细信息，请参阅[存储帐户的高级威胁防护](https://azure.microsoft.com/resources/templates/201-storage-advanced-threat-protection-create/)。

### <a name="using-azure-policy"></a>使用 Azure 策略

使用 Azure 策略以在特定订阅或资源组下的存储帐户启用高级威胁防护。

1. 再 Azure**策略-定义**页。

1. 搜索**存储帐户上部署高级威胁防护**策略。

     ![搜索策略](./media/storage-advanced-threat-protection/storage-atp-policy-definitions.png)
  
1. 选择 Azure 订阅或资源组。

    ![选择订阅或组](./media/storage-advanced-threat-protection/storage-atp-policy2.png)

1. 将策略分配。

    ![策略定义页面](./media/storage-advanced-threat-protection/storage-atp-policy1.png)

### <a name="using-rest-api"></a>使用 REST API
使用 Rest API 命令来创建、 更新或获取特定的存储帐户的高级威胁防护设置。

* [高级威胁防护-创建](https://docs.microsoft.com/rest/api/securitycenter/advancedthreatprotection/create)
* [高级威胁防护-获取](https://docs.microsoft.com/rest/api/securitycenter/advancedthreatprotection/get)

### <a name="using-azure-powershell"></a>使用 Azure PowerShell

使用以下 PowerShell cmdlet:

  * [启用高级的威胁防护](https://docs.microsoft.com/powershell/module/az.security/enable-azsecurityadvancedthreatprotection)
  * [获取高级威胁防护](https://docs.microsoft.com/powershell/module/az.security/get-azsecurityadvancedthreatprotection)
  * [禁用高级的威胁防护](https://docs.microsoft.com/powershell/module/az.security/disable-azsecurityadvancedthreatprotection)

## <a name="explore-security-anomalies"></a>浏览安全异常

存储活动出现异常时，用户会收到一封电子邮件通知，其中包含有关可疑安全事件的信息。 事件详细信息包括：

* 异常的性质
* 存储帐户名称
* 事件时间
* 存储类型
* 可能的原因 
* 调查步骤
* 修正步骤


电子邮件还包含有关可能原因的详细信息以及用于调查和缓解潜在威胁的建议操作。

![Azure 存储高级威胁防护警报电子邮件](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-alert-email.png)

可从 Azure 安全中心的[安全警报磁贴](../../security-center/security-center-managing-and-responding-alerts.md#managing-security-alerts)查看和管理当前安全警报。 单击特定警报可提供详细信息以及用于调查当前威胁和解决潜在威胁的操作。

![Azure 存储高级威胁防护警报电子邮件](./media/storage-advanced-threat-protection/storage-advanced-threat-protection-alert.png)

## <a name="protection-alerts"></a>保护警报

警报是因访问或攻击存储帐户的异常且可能有害的尝试而生成的。 这些事件可以触发以下警报：

### <a name="anomalous-access-pattern-alerts"></a>异常访问模式的警报

* **从异常位置访问**:在存储帐户的访问模式中的更改时，会触发此警报。 例如，当用户从异常地理位置访问存储帐户时。
可能的原因：
   * 攻击者已访问你的存储帐户
   * 合法用户已从新位置访问你的存储帐户
 
* **应用程序异常**:此警报表示异常的应用程序具有访问此存储帐户。 可能的原因：
   * 攻击者已访问使用新的应用程序在存储帐户。
   * 合法用户已使用新的应用程序/浏览器访问你的存储帐户。

* **匿名访问**:此警报表示存储帐户的访问模式中的更改。 例如，此帐户已被以匿名方式访问 （即，没有任何身份验证），这是意外相比对此帐户的最新的访问模式。
可能的原因：
   * 攻击者已利用了对容器的公共读取访问。
   * 合法用户或应用程序已使用到的容器的公共读取访问权限。

### <a name="anomalous-extractupload-alerts"></a>异常的提取/上传警报

* **数据泄露**:此警报表示已与此存储容器上最近的活动相比，提取的数据量非常大。 可能的原因：
   * 攻击者已从容器中提取大量数据。 (例如： 数据泄露/破坏、 未经授权的数据传输)
   * 合法用户或应用程序已从容器中提取不寻常的大量数据。 (例如： 维护活动)

* **意外的删除**:此警报表明一个或多个意外的删除操作发生在存储帐户中，相比在此帐户上最近的活动。 可能的原因：
   * 攻击者已从你的存储帐户中删除数据。
   * 合法用户执行了异常的删除。

* **将 Azure 云服务包上传**:此警报表明，已与此帐户上最近的活动相比以异常方式上载到存储帐户的 Azure 云服务包 （.cspkg 文件）。 可能的原因： 
   * 攻击者准备部署到 Azure 云服务从你的存储帐户的恶意代码。
   * 已为合法服务部署而准备了合法用户。

### <a name="suspicious-storage-activities-alerts"></a>可疑存储活动警报

* **访问权限更改**:此警报表示已以异常方式中的存储容器的访问权限已更改。 可能的原因： 
   * 攻击者已更改容器权限以降低其安全性。
   * 合法用户更改了容器的权限。

* **访问检查**:此警报表明，已与此帐户上最近的活动相比以异常方式检查存储帐户的访问权限。 可能的原因： 
   * 攻击者执行了未来攻击的侦测。
   * 合法用户的存储帐户上执行维护。

* **数据浏览**:此警报表示存储帐户中的容器或 blob 已枚举以异常方式，与此帐户上最近的活动相比。 可能的原因： 
   * 攻击者执行了未来攻击的侦测。
   * 合法用户或应用程序逻辑探讨了存储帐户中的数据。






## <a name="next-steps"></a>后续步骤

* 详细了解[登录 Azure 存储帐户](/rest/api/storageservices/About-Storage-Analytics-Logging)

* 了解有关 [Azure 安全中心](../../security-center/security-center-intro.md)的详细信息
