---
title: 从 Azure 购买和配置 SSL 证书 - 应用服务 | Microsoft Docs
description: 了解如何购买应用服务证书并将其绑定到应用服务应用
services: app-service
documentationcenter: .net
author: cephalin
manager: jpconnoc
tags: buy-ssl-certificates
ms.assetid: cdb9719a-c8eb-47e5-817f-e15eaea1f5f8
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: apurvajo;cephalin
ms.custom: seodec18
ms.openlocfilehash: b561091eedb43e1c77f3c97951beeb92bfcf4751
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65202923"
---
# <a name="buy-and-configure-an-ssl-certificate-for-azure-app-service"></a>为 Azure 应用服务购买和配置 SSL 证书

本教程介绍如何通过在 [Azure 密钥保管库](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)中创建（购买）应用服务证书，然后将其绑定到应用服务应用来保护[应用服务应用](https://docs.microsoft.com/azure/app-service/)或[函数应用](https://docs.microsoft.com/azure/azure-functions/)。

> [!TIP]
> App Service 证书可用于任何 Azure 或非 Azure 服务，且不限于应用服务。 为此，需要创建应用服务证书的本地 PFX 副本，以便随时随地使用它。 有关详细信息，请参阅[创建应用服务证书的本地 PFX 副本](https://blogs.msdn.microsoft.com/benjaminperkins/2017/04/12/export-an-azure-app-service-certificate-pfx-powershell/)。
>

## <a name="prerequisites"></a>必备组件

按照本操作方法指南操作：

- [创建应用服务应用](/azure/app-service/)
- [将域名映射到应用](app-service-web-tutorial-custom-domain.md)或[在 Azure 中购买并配置](manage-custom-dns-buy-domain.md)

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

## <a name="start-certificate-order"></a>启动证书申请

在<a href="https://portal.azure.com/#create/Microsoft.SSL" target="_blank">应用服务证书创建页</a>启动应用服务证书申请。

![证书创建](./media/app-service-web-purchase-ssl-web-site/createssl.png)

使用下表来帮助配置证书。 完成后，单击“创建”。

| 设置 | 描述 |
|-|-|
| 名称 | 应用服务证书证书的友好名称。 |
| 裸域主机名 | 如果在此处指定根域，则会获得一个证书，该证书*同时*对根域和 `www` 子域提供保护。 若要仅保护子域，请在此处指定子域的完全限定域名（例如，`mysubdomain.contoso.com`）。 |
| 订阅 | 托管 Web 应用的数据中心。 |
| 资源组 | 包含证书的资源组。 例如，可以使用新资源组，或选择与应用服务应用相同的资源组。 |
| 证书 SKU | 确定要创建的证书类型是标准证书还是[通配符证书](https://wikipedia.org/wiki/Wildcard_certificate)。 |
| 法律条款 | 单击以确认你同意法律条款。 从 GoDaddy 获取证书。 |

## <a name="store-in-azure-key-vault"></a>存储在 Azure Key Vault 中

证书购买过程完成后，还需完成其他一些步骤才可开始使用此证书。 

选择[应用服务证书](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders)页中的证书，然后单击“证书配置” > “步骤 1: 存储”。

![插入已准备好存储在 KV 中的图像](./media/app-service-web-purchase-ssl-web-site/ReadyKV.png)

[Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis) 是一项 Azure 服务，可帮助保护云应用程序和服务使用的加密密钥和机密。 它是为应用服务证书所选的存储。

在“Key Vault 状态”页，单击“Key Vault 存储库”以创建新的保管库或选择现有保管库。 如果选择创建新的保管库，请使用下表以帮助配置保管库，然后单击“创建”。 查看如何在同一订阅和资源组中创建新的 Key Vault。

| 设置 | 描述 |
|-|-|
| 名称 | 由字母数字字符和短划线组成的唯一名称。 |
| 资源组 | 建议选择与应用服务证书相同的资源组。 |
| Location | 选择与应用服务应用相同的位置。 |
| 定价层 | 有关信息，请参阅 [Azure Key Vault 定价详细信息](https://azure.microsoft.com/pricing/details/key-vault/)。 |
| 访问策略| 定义应用程序和对保管库资源允许的访问权限。 可以稍后配置，请按照[授予多个应用程序访问密钥保管库的权限](../key-vault/key-vault-group-permissions-for-apps.md)的步骤进行操作。 |
| 虚拟网络访问 | 限制为仅特定 Azure 虚拟网络具有保管库访问权限。 可以稍后配置，请按照[配置 Azure Key Vault 防火墙和虚拟网络](../key-vault/key-vault-network-security.md)的步骤进行操作 |

选择保管库后，关闭“Key Vault 存储库”页面。 “存储”选项应显示绿色选中标记，表示成功。 保持页面处于打开状态，执行下一步骤。

## <a name="verify-domain-ownership"></a>验证域所有权

在上一步中所用的同一“证书配置”页中，单击“步骤 2: 验证”。

![](./media/app-service-web-purchase-ssl-web-site/verify-domain.png)

选择“应用服务验证”。 由于已将域映射到 Web 应用（请参阅[先决条件](#prerequisites)），所以验证已经执行。 只需单击“验证”来完成此步骤。 单击“刷新”按钮，直到显示“证书为域已验证”消息。

> [!NOTE]
> 支持四种类型的域验证方法： 
> 
> - **应用服务验证** - 当域已映射到同一订阅中的应用服务应用时，这是最方便的选项。 它可利用应用服务应用已验证域所有权这一事实。
> - **域** - 验证[从 Azure 购买的应用服务域](manage-custom-dns-buy-domain.md)。 Azure 会自动为你添加验证 TXT 记录，并完成该过程。
> - **邮件** - 通过向域管理员发送电子邮件来验证域。 选择此选项时会提供相应说明。
> - **手动** - 使用 HTML 页（仅标准证书）或 DNS TXT 记录验证域。 选择此选项时会提供相应说明。

## <a name="bind-certificate-to-app"></a>将证书绑定到应用

在 [Azure 门户](https://portal.azure.com/)的左侧菜单中，选择“应用服务” > “\<your_ app>”。

在应用的左侧导航窗格中，选择“SSL 设置” > “私有证书 (.pfx)” > “导入应用服务证书”。

![插入导入证书的图像](./media/app-service-web-purchase-ssl-web-site/ImportCertificate.png)

选择刚刚购买的证书。

导入证书后，需要将其绑定到应用中已映射的域名。 选择“绑定” > “添加 SSL 绑定”。 

![插入导入证书的图像](./media/app-service-web-purchase-ssl-web-site/AddBinding.png)

使用下表帮助在“SSL 绑定”对话框中配置绑定，然后单击“添加绑定”。

| 设置 | 描述 |
|-|-|
| 主机名 | 要为其添加 SSL 绑定的域名。 |
| 私有证书指纹 | 要绑定的证书。 |
| SSL 类型 | <ul><li>**SNI SSL** - 可添加多个基于 SNI 的 SSL 绑定。 选择此选项可以使用多个 SSL 证书来保护同一 IP 地址上的多个域。 大多数新式浏览器（包括 Internet Explorer、Chrome、Firefox 和 Opera）都支持 SNI（可以在[服务器名称指示](https://wikipedia.org/wiki/Server_Name_Indication)中了解更全面的浏览器支持信息）。</li><li>基于 IP 的 SSL - 只能添加一个基于 IP 的 SSL 绑定。 选择此选项只能使用一个 SSL 证书来保护专用公共 IP 地址。 配置绑定后，请按照[重新映射 IP SSL 的 A 记录](app-service-web-tutorial-custom-ssl.md#remap-a-record-for-ip-ssl)中的步骤进行操作。 </li></ul> |

## <a name="verify-https-access"></a>验证 HTTPS 访问

使用 `HTTPS://<domain_name>`（而非 `HTTP://<domain_name>`）访问应用，以验证是否已正确配置证书。

## <a name="rekey-certificate"></a>为证书重新生成密钥

如果您认为您的证书的专用密钥已泄漏，可以重新生成你的证书。 选择中的证书[应用服务证书](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders)页上，然后选择**重新生成密钥并同步**从左侧导航栏中。

单击**重新生成密钥**启动进程。 此过程需要 1 - 10 分钟才能完成。

![插入重新生成 SSL 密钥的图像](./media/app-service-web-purchase-ssl-web-site/Rekey.png)

通过重新生成证书的密钥，将使用证书颁发机构颁发的新证书滚动更新现有证书。

重新生成密钥操作完成后，单击**同步**。同步操作会自动更新应用服务中的证书的主机名绑定，而不会导致任何停机时间缩到您的应用程序。

> [!NOTE]
> 如果您不单击**同步**，应用服务会自动在 48 小时内同步你的证书。

## <a name="renew-certificate"></a>续订证书

若要在任何时候启用证书自动续订，请选择[应用服务证书](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders)页面中的证书，然后单击左侧导航窗格的“自动续订设置”。

选择“开”，然后单击“保存”。 如果启用了自动续订，则证书会在到期前 60 天自动续订。

![自动续订证书](./media/app-service-web-purchase-ssl-web-site/auto-renew.png)

若要改为手动续订证书，请单击“手动续订”。 可以请求在到期前 60 天手动续订证书。

续订操作完成后，单击**同步**。同步操作会自动更新应用服务中的证书的主机名绑定，而不会导致任何停机时间缩到您的应用程序。

> [!NOTE]
> 如果您不单击**同步**，应用服务会自动在 48 小时内同步你的证书。

## <a name="automate-with-scripts"></a>使用脚本自动化

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom SSL certificate to a web app")]

## <a name="more-resources"></a>更多资源

* [实施 HTTPS](app-service-web-tutorial-custom-ssl.md#enforce-https)
* [实施 TLS 1.1/1.2](app-service-web-tutorial-custom-ssl.md#enforce-tls-versions)
* [在 Azure 应用服务的应用程序代码中使用 SSL 证书](app-service-web-ssl-cert-load.md)
* [常见问题解答：应用服务证书](https://blogs.msdn.microsoft.com/appserviceteam/2017/07/24/faq-app-service-certificates/)
