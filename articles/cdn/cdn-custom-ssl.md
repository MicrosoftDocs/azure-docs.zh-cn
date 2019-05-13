---
title: 教程 - 在 Azure CDN 自定义域上配置 HTTPS | Microsoft Docs
description: 在本教程中，你将了解如何在 Azure CDN 终结点自定义域上启用和禁用 HTTPS。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 10337468-7015-4598-9586-0b66591d939b
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/18/2019
ms.author: magattus
ms.custom: mvc
ms.openlocfilehash: dc43e2ad2668a7d3a808e398857cbf1d28c9aa1c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65150852"
---
# <a name="tutorial-configure-https-on-an-azure-cdn-custom-domain"></a>教程：在 Azure CDN 自定义域中配置 HTTPS

本教程演示如何为与 Azure CDN 终结点关联的自定义域启用 HTTPS 协议。 通过在自定义域（例如 https:\//www.contoso.com）上使用 HTTPS 协议，可以确保敏感数据在通过 Internet 发送时可以通过 TLS/SSL 加密安全地进行分发。 Web 浏览器通过 HTTPS 连接到网站时，它会验证网站的安全证书并验证该证书是否是由合法的证书颁发机构颁发的。 此过程提供安全性并保护 Web 应用程序免受攻击。

默认情况下，Azure CDN 支持对 CDN 终结点主机名使用 HTTPS。 例如，如果创建 CDN 终结点（例如 https:\//contoso.azureedge.net），则会自动启用 HTTPS。  

自定义 HTTPS 功能的一些关键属性包括：

- 无需额外付费：证书获取或续订不收取费用，对于 HTTPS 流量不另外收费。 只需为从 CDN 出口的 GB 数付费。

- 简单启用：可从 [Azure 门户](https://portal.azure.com)进行一键式预配。 还可以使用 REST API 或其他开发人员工具启用该功能。

- 提供完整的证书管理：为你处理所有证书获取和管理。 证书在过期之前将自动进行设置并续订，这可消除由于证书过期而导致服务中断的风险。

本教程介绍如何执行以下操作：
> [!div class="checklist"]
> - 在自定义域上启用 HTTPS 协议。
> - 使用 CDN 托管的证书 
> - 使用自己的证书
> - 验证域
> - 在自定义域上禁用 HTTPS 协议。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)] 

在完成本教程中的步骤之前，必须先创建一个 CDN 配置文件，一个至少一个 CDN 终结点。 有关详细信息，请参阅[快速入门：创建 Azure CDN 配置文件和终结点](cdn-create-new-endpoint.md)。

此外，还必须在 CDN 终结点上关联一个 Azure CDN 自定义域。 有关详细信息，请参阅[教程：将自定义域添加到 Azure CDN 终结点](cdn-map-content-to-custom-domain.md)

---

## <a name="ssl-certificates"></a>SSL 证书数
若要启用 HTTPS 协议以在 Azure CDN 自定义域上安全传送内容，必须使用 SSL 证书。 可以选择是使用由 Azure CDN 托管的证书还是使用自己的证书。


# <a name="option-1-default-enable-https-with-a-cdn-managed-certificatetaboption-1-default-enable-https-with-a-cdn-managed-certificate"></a>[选项 1（默认）：使用 CDN 托管的证书启用 HTTPS](#tab/option-1-default-enable-https-with-a-cdn-managed-certificate)

使用 CDN 托管的证书时，只需单击几下即可启用 HTTPS 功能。 Azure CDN 可处理所有证书管理任务，如获取和续订。 启用此功能后，进程将立即启动。 如果自定义域已映射到 CDN 终结点，则不需要进一步操作。 Azure CDN 将自动执行步骤并完成请求。 但是，如果自定义域映射到其他位置，则必须使用电子邮件来验证域所有权。

若要在自定义域上启用 HTTPS，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.com)中，浏览到“Microsoft 的 Azure CDN 标准版”、“Akamai 的 Azure CDN 标准版”、“Verizon 的 Azure CDN 标准版”或“Verizon 的 Azure CDN 高级版”配置文件。

2. 在 CDN 终结点列表中，选择包含自定义域的终结点。

    ![终结点列表](./media/cdn-custom-ssl/cdn-select-custom-domain-endpoint.png)

    此时会显示“终结点”页。

3. 在自定义域列表中，选择要为其启用 HTTPS 的自定义域。

    ![自定义域列表](./media/cdn-custom-ssl/cdn-custom-domain.png)

    此时将显示“自定义域”页。

4. 在证书管理类型下，选择“CDN 托管”。

5. 选择“打开”，启用 HTTPS。

    ![自定义域 HTTPS 状态](./media/cdn-custom-ssl/cdn-select-cdn-managed-certificate.png)

6. 继续[验证域](#validate-the-domain)。


# <a name="option-2-enable-https-with-your-own-certificatetaboption-2-enable-https-with-your-own-certificate"></a>[选项 2：使用自己的证书启用 HTTPS](#tab/option-2-enable-https-with-your-own-certificate)

> [!IMPORTANT]
> 此选项仅适用于来自 Microsoft 的 Azure CDN 和来自 Verizon 的 Azure CDN 配置文件。 
>
 
可以使用自己的证书启用 HTTPS 功能。 此过程通过与 Azure Key Vault 的集成完成，后者允许你安全地存储证书。 Azure CDN 使用此安全机制来获得你的证书，并且需要一些额外的步骤。 创建 SSL 证书时，必须使用允许的证书颁发机构 (CA) 创建。 否则，如果使用不允许的 CA，你的请求将被拒绝。 有关允许的 CA 的列表，请参阅[允许在 Azure CDN 上启用自定义 HTTPS 的证书颁发机构](cdn-troubleshoot-allowed-ca.md)。对于来自 Verizon 的 Azure CDN，可接受任何有效 CA。 

### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>准备 Azure Key Vault 帐户和证书
 
1. Azure Key Vault：在要启用自定义 HTTPS 的 Azure CDN 配置文件和 CDN 终结点的同一订阅下，必须具有正在运行的 Azure Key Vault 帐户。 创建 Azure Key Vault 帐户（如果还没有帐户）。
 
2. Azure Key Vault 证书：如果已有证书，可以将其直接上传到 Azure Key Vault 帐户，或者，可以直接通过 Azure Key Vault，从 Azure Key Vault 集成的合作伙伴 CA 之一创建新的证书。 

### <a name="register-azure-cdn"></a>注册 Azure CDN

通过 PowerShell 将 Azure CDN 注册为 Azure Active Directory 中的应用。

1. 根据需要在本地计算机上安装 [Azure PowerShell](/powershell/azure/install-az-ps)。

2. 在 PowerShell 中运行以下命令：

     `New-AzADServicePrincipal -ApplicationId "205478c0-bd83-4e1b-a9d6-db63a3e1e1c8"`

    ![在 PowerShell 中注册 Azure CDN](./media/cdn-custom-ssl/cdn-register-powershell.png)
              

### <a name="grant-azure-cdn-access-to-your-key-vault"></a>向 Azure CDN 授予 Key Vault 的访问权限
 
在 Azure Key Vault 帐户中为 Azure CDN 授予证书（机密）的访问权限。

1. 在 Key Vault 帐户的“设置”下，选择“访问策略”，然后选择“添加新策略”以创建新策略。

    ![创建新的访问策略](./media/cdn-custom-ssl/cdn-new-access-policy.png)

2. 在“选择主体”中，搜索 **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8**，并选择 **Microsoft.Azure.Cdn**。 单击“选择”。

    ![访问策略设置](./media/cdn-custom-ssl/cdn-access-policy-settings.png)

3. 在“机密权限”中，选择“获取”以允许 CDN 执行这些权限来获取并列出证书。 

4. 选择“确定”。 

    Azure CDN 现在可以访问此 Key Vault 和存储在其中的证书（机密）。
 
### <a name="select-the-certificate-for-azure-cdn-to-deploy"></a>选择要部署的 Azure CDN 证书
 
1. 返回 Azure CDN 门户，选择想要启用自定义 HTTPS 的配置文件和 CDN 终结点。 

2. 在自定义域列表中，选择要为其启用 HTTPS 的自定义域。

    此时将显示“自定义域”页。

3. 在证书管理类型下，选择“使用我自己的证书”。 

    ![配置证书](./media/cdn-custom-ssl/cdn-configure-your-certificate.png)

4. 选择 Key Vault，证书（机密）和证书版本。

    Azure CDN 会列出下列信息： 
    - 订阅 ID 的 Key Vault 帐户。 
    - 所选 Key Vault 下的证书（机密）。 
    - 可用证书版本。 
 
5. 选择“打开”，启用 HTTPS。
  
6. 使用自己的证书时，不需要对域进行验证。 转至[等待传播](#wait-for-propagation)。

---

## <a name="validate-the-domain"></a>验证域

如果已使用一个自定义域且该自定义域通过 CNAME 记录映射到自定义终结点，或使用的是自己的证书，请转至  
[自定义域已映射到 CDN 终结点](#custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record)。 否则，如果终结点的 CNAME 记录条目不再存在或者它包含 cdnverify 子域，请转至[自定义域未映射到 CDN 终结点](#custom-domain-is-not-mapped-to-your-cdn-endpoint)。

### <a name="custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record"></a>自定义域已通过 CNAME 记录映射到 CDN 终结点

将自定义域添加到终结点时，会在域注册机构的 DNS 表中创建一条 CNAME 记录，以将其映射到 CDN 终结点主机名。 如果该 CNAME 记录仍然存在，并且不包含 cdnverify 子域，则 DigiCert CA 将使用它来自动验证自定义域的所有权。 

如果使用的是自己的证书，则不需要对域进行验证。

CNAME 记录应采用以下格式，其中 *Name* 是自定义域名，*Value* 是 CDN 终结点主机名：

| 名称            | 类型  | 值                 |
|-----------------|-------|-----------------------|
| <www.contoso.com> | CNAME | contoso.azureedge.net |

有关 CNAME 记录的详细信息，请参阅[创建 CNAME DNS 记录](https://docs.microsoft.com/azure/cdn/cdn-map-content-to-custom-domain)。

如果 CNAME 记录采用正确的格式，DigiCert 会自动验证自定义域名，并为域名创建专用的证书。 DigitCert 不会向你发送验证电子邮件，并且你无需批准请求。 该证书会在一年内有效，并会在过期前自动续订。 转至[等待传播](#wait-for-propagation)。 

自动验证通常要花费几分钟时间。 如果在一小时内未看到域完成验证，请创建一个支持票证。

>[!NOTE]
>如果通过 DNS 提供商获得证书颁发机构授权 (CAA) 记录，则必须包含 DigiCert 作为一个有效的 CA。 CAA 记录允许域名所有者通过自己的 DNS 提供商指定哪些 CA 有权为其域名颁发证书。 如果某个 CA 收到具有 CAA 记录的域证书订单，但该 CA 未被列为授权的颁发者，则禁止向该域或子域颁发证书。 有关管理 CAA 记录的信息，请参阅[管理 CAA 记录](https://support.dnsimple.com/articles/manage-caa-record/)。 有关 CAA 记录工具，请参阅 [CAA 记录帮助器](https://sslmate.com/caa/)。

### <a name="custom-domain-is-not-mapped-to-your-cdn-endpoint"></a>自定义域未映射到 CDN 终结点

如果终结点的 CNAME 记录条目不再存在，或者它包含 cdnverify 子域，请按照此步骤中的其余说明进行操作。

>[!NOTE]
>如果使用 **Akamai 的 Azure CDN** 配置文件，则目前不能对自定义域所有权进行电子邮件验证。 此功能目前尚未完成。 

在自定义域上提交启用 HTTPS 的请求后，DigiCert CA 会根据域的 [WHOIS](http://whois.domaintools.com/) 注册者信息，通过联系域的注册者来验证域的所有权。 通过 WHOIS 注册中列出的电子邮件地址（默认）或电话号码进行联系。 必须先完成域验证，才能在自定义域上激活 HTTPS。 可在 6 个工作日内批准域。 自动取消 6 个工作日内未批准的请求。 

![WHOIS 记录](./media/cdn-custom-ssl/whois-record.png)

DigiCert 还会将验证电子邮件发送到其他电子邮件地址。 如果 WHOIS 注册信息属于隐私，请验证是否可直接从以下任一地址进行审批：

admin@&lt;your-domain-name.com&gt;  
administrator@&lt;your-domain-name.com&gt;  
webmaster@&lt;your-domain-name.com&gt;  
hostmaster@&lt;your-domain-name.com&gt;  
postmaster@&lt;your-domain-name.com&gt;  

应会在几分钟内收到如以下所示的电子邮件，要求你批准请求。 如果使用垃圾邮件筛选，请将 admin@digicert.com 添加到允许列表。 如果未在 24 小时内收到电子邮件，请与 Microsoft 支持部门联系。
    
![域验证电子邮件](./media/cdn-custom-ssl/domain-validation-email.png)

单击批准链接时，会定向到以下在线批准表： 
    
![域验证表单](./media/cdn-custom-ssl/domain-validation-form.png)

按表中的说明操作；有两种验证选项：

- 可批准通过同一根域（例如 consoto.com）的同一帐户下的所有将来订单。 如果你打算为同一根域添加其他自定义域，建议使用此方法。

- 可以只批准该请求中使用的特定主机名。 后续请求将需要其他批准。

批准后，DigiCert 会针对自定义域名完成证书创建。 该证书会在一年内有效，并会在过期前自动续订。

## <a name="wait-for-propagation"></a>等待传播

验证域名后，将需要长达 6-8 小时才能使自定义域 HTTPS 功能激活。 此过程完成后，Azure 门户中的自定义 HTTPS 状态会设置为“已启用”，且自定义域对话框中的四个操作步骤会标记为完成。 自定义域现可使用 HTTPS。

![启用 HTTPS 对话框](./media/cdn-custom-ssl/cdn-enable-custom-ssl-complete.png)

### <a name="operation-progress"></a>操作进度

下表显示启用 HTTPS 时出现的操作进度。 启用 HTTPS 后，自定义域对话框中将出现四个操作步骤。 每个步骤变为活动状态时，其下将随之显示更多子步骤详细信息。 并非所有这些子步骤都会执行。 步骤成功完成后，它旁边会显示一个绿色的复选标记。 

| 操作步骤 | 操作子步骤详细信息 | 
| --- | --- |
| 1 提交请求 | 提交请求 |
| | 正在提交 HTTPS 请求。 |
| | 已成功提交 HTTPS 请求。 |
| 2 域验证 | 如果域是映射到 CDN 终结点的 CNAME，则会自动验证域。 否则，将会向域的注册记录中列出的电子邮件（WHOIS 注册者）发送一个验证请求。 请尽快验证域。 |
| | 已成功验证域所有权。 |
| | 域所有权验证请求已过期（很可能是客户在 6 天内未响应）。 将不会在域中启用 HTTPS。 * |
| | 客户已拒绝域所有权验证请求。 将不会在域中启用 HTTPS。 * |
| 3 证书预配 | 证书颁发机构当前正在颁发在你的域中启用 HTTPS 所需的证书。 |
| | 证书已颁发，当前正将证书部署到 CDN 网络。 最多可能需要 6 小时才能完成此操作。 |
| | 已成功将证书部署到 CDN 网络。 |
| 4 完成 | 已成功在域中启用 HTTPS。 |

\* 除非出现错误，否则不会显示此消息。 

如果提交请求之前出现错误，则会显示以下错误消息：

<code>
We encountered an unexpected error while processing your HTTPS request. Please try again and contact support if the issue persists.
</code>



## <a name="clean-up-resources---disable-https"></a>清理资源 - 禁用 HTTPS

在前面的步骤中，你在自定义域上启用了 HTTPS 协议。 如果不再希望为自定义域使用 HTTPS，可以通过执行下列步骤来禁用 HTTPS：

### <a name="disable-the-https-feature"></a>禁用 HTTPS 功能 

1. 在 [Azure 门户](https://portal.azure.com)中，浏览到“Microsoft 的 Azure CDN 标准版”、“Verizon 的 Azure CDN 标准版”或“Verizon 的 Azure CDN 高级版”配置文件。

2. 在终结点的列表中，单击包含自定义域的终结点。

3. 单击要禁用 HTTPS 的自定义域。

    ![自定义域列表](./media/cdn-custom-ssl/cdn-custom-domain-HTTPS-enabled.png)

4. 单击“禁用”**上**禁用 HTTPS，然后单击“应用”。

    ![“自定义 HTTPS”对话框](./media/cdn-custom-ssl/cdn-disable-custom-ssl.png)

### <a name="wait-for-propagation"></a>等待传播

禁用自定义域 HTTPS 功能后，最多可能需要 6-8 小时才会生效。 此过程完成后，Azure 门户中的自定义 HTTPS 状态会设置为“已禁用”，且自定义域对话框中的三个操作步骤会标记为完成。 自定义域不再能够使用 HTTPS。

![禁用 HTTPS 对话框](./media/cdn-custom-ssl/cdn-disable-custom-ssl-complete.png)

#### <a name="operation-progress"></a>操作进度

下表显示在禁用 HTTPS 时发生的操作进程。 禁用 HTTPS 后，自定义域对话框中将出现三个操作步骤。 每个步骤变为活动状态时，其他详细信息将显示在相应步骤下。 步骤成功完成后，它旁边会显示一个绿色的复选标记。 

| 操作进度 | 操作详细信息 | 
| --- | --- |
| 1 提交请求 | 提交请求 |
| 2 证书取消预配 | 删除证书 |
| 3 完成 | 已删除证书 |

## <a name="frequently-asked-questions"></a>常见问题

1. 谁是证书提供者？使用哪种类型的证书？

    对于 **Verizon 的 Azure CDN** 和 **Microsoft 的 Azure CDN**，Digicert 提供的专用/单个证书将用于自定义域。 

2. 使用基于 IP 的 TLS/SSL 还是 SNI TLS/SSL？

    **Verizon 的 Azure CDN** 和 **Microsoft 的 Azure CDN Standard** 都使用 SNI TLS/SSL。

3. 如果我未收到 DigiCert 发来的域验证电子邮件，怎么办？

    如果自定义域的 CNAME 条目直接指向终结点主机名（并且你未使用 cdnverify 子域名称），则你不会收到域验证电子邮件。 验证会自动进行。 否则，如果你没有 CNAME 条目，并且在 24 小时内未收到电子邮件，请联系 Microsoft 支持部门。

4. 使用 SAN 证书是否没有使用专用证书安全？
    
    SAN 证书遵循与专用证书相同的加密和安全标准。 所有颁发的 SSL 证书都使用 SHA-256 来增强服务器安全性。

5. 我是否需要通过我的 DNS 提供商获得证书颁发机构授权记录？

    否，当前不需要证书颁发机构授权记录。 但是，如果你确实有一个，则必须包含 DigiCert 作为一个有效的 CA。

6. 从 2018 年 6 月 20 日开始，Verizon 的 Azure CDN 默认使用专用证书和 SNI TLS/SSL。使用“使用者可选名称”(SAN) 证书和基于 IP 的 TLS/SSL 的现有自定义域会发生什么情况？

    如果 Microsoft 经分析后发现，只是向应用程序发出了仅 SNI 客户端请求，则现有的域将在未来几个月逐渐迁移到单个证书。 如果 Microsoft 检测到向应用程序发出了一些非 SNI 客户端请求，则域将保留在 SAN 证书和基于 IP 的 TLS/SSL 中。 在任何情况下，对服务或客户端请求的支持都不会中断，不管这些请求是否为 SNI。


## <a name="next-steps"></a>后续步骤

本教程介绍了以下操作：

> [!div class="checklist"]
> - 在自定义域上启用 HTTPS 协议。
> - 使用 CDN 托管的证书 
> - 使用自己的证书
> - 验证域。
> - 在自定义域上禁用 HTTPS 协议。

继续学习下一教程，了解如何在 CDN 终结点上配置缓存。

> [!div class="nextstepaction"]
> [教程：设置 Azure CDN 缓存规则](cdn-caching-rules-tutorial.md)

