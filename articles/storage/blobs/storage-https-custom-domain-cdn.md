---
title: 在 Azure CDN 中使用自定义域通过 HTTPS 访问 Blob
description: 了解如何将 Azure CDN 与 Blob 存储进行集成以使用自定义域通过 HTTPS 访问 Blob
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 06/26/2018
ms.author: normesta
ms.reviewer: seguler
ms.subservice: blobs
ms.openlocfilehash: da3a6dcb0d125ac4666bc375e843c57cf12fb2fc
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148405"
---
# <a name="use-azure-cdn-to-access-blobs-with-custom-domains-over-https"></a>在 Azure CDN 中使用自定义域通过 HTTPS 访问 Blob

Azure 内容分发网络 (Azure CDN) 现在支持针对自定义域名使用 HTTPS。 借助 Azure CDN，可以使用自定义域名通过 HTTPS 访问 Blob。 为此，请在 Blob 或 Web 终结点上启用 Azure CDN，并将 CDN 映射到自定义域名。 完成这些步骤后，Azure 会简化为自定义域启用 HTTPS 的过程，它支持一键式访问和全面的证书管理。 一般 Azure CDN 定价不会提高。

在传输 Web 应用程序数据时，Azure CDN 可帮助保护隐私和数据完整性。 Azure CDN 使用 SSL 协议通过 HTTPS 提供流量，在通过 Internet 发送数据时可保持数据的加密状态。 将 HTTPS 与 Azure CDN 配合使用有助于防范 Web 应用程序受到攻击。

> [!NOTE]  
> 除了为自定义域名提供 SSL 支持外，Azure CDN 还可以帮助扩展应用程序以便在世界范围内交付高带宽内容。 有关详细信息，请参阅 [Azure CDN 概述](../../cdn/cdn-overview.md)。

## <a name="quickstart"></a>快速入门

若要为自定义 Blob 存储终结点启用 HTTPS，请执行以下操作：

1.  [将 Azure 存储帐户与 Azure CDN 相集成](../../cdn/cdn-create-a-storage-account-with-cdn.md)。  
    本文将引导你在 Azure 门户中创建存储帐户（如果尚未执行此操作）。

    > [!NOTE]  
    > 若要在 Azure 存储的静态网站支持预览期添加存储 Web 终结点，请在“源类型”下拉列表中选择“自定义源”。 在 Azure 门户中，需要通过 Azure CDN 配置文件（而不是直接在存储帐户中）这样做。

2.  [将 Azure CDN 内容映射到自定义域](../../cdn/cdn-map-content-to-custom-domain.md)。

3.  [在 Azure CDN 自定义域上启用 HTTPS](../../cdn/cdn-custom-ssl.md)。

## <a name="shared-access-signatures"></a>共享访问签名

默认情况下，Blob 存储终结点不允许进行匿名读取访问。 如果 Blob 存储终结点配置为不允许进行匿名读取访问，请在向自定义域发出的每个请求中提供一个[共享访问签名](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)令牌。 有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](storage-manage-access-to-resources.md)。

Azure CDN 不遵守添加到共享访问签名令牌的任何限制。 例如，所有共享访问签名令牌都会过期。 从 Azure CDN 边缘节点中清除内容之前，仍可以使用已过期的共享访问签名来访问该内容。 可以通过设置缓存响应标头来控制数据在 Azure CDN 上的缓存时间。 有关操作方法，请参阅[管理 Azure CDN 中 Azure 存储 Blob 的过期问题](../../cdn/cdn-manage-expiration-of-blob-content.md)。

如果为同一 Blob 终结点创建了两个或更多个共享访问签名 URL，则我们建议为 Azure CDN 启用查询字符串缓存。 此操作可确保 Azure 将每个 URL 视为唯一的实体。 有关详细信息，请参阅[使用查询字符串控制 Azure CDN 缓存行为](../../cdn/cdn-query-string.md)。

## <a name="http-to-https-redirection"></a>HTTP 到 HTTPS 重定向

可将 HTTP 流量重定向到 HTTPS。 这需要使用 Verizon 提供的 Azure CDN 高级产品/服务。 [使用 Azure CDN 规则引擎替代 HTTP 行为](../../cdn/cdn-rules-engine.md)，并应用以下规则：

![HTTP 到 HTTPS 重定向规则](./media/storage-https-custom-domain-cdn/redirect-to-https.png)

从下拉列表中选择的 *Cdn-endpoint-name* 引用你为 Azure CDN 终结点配置的名称。 *Origin-path* 引用源存储帐户中存储静态内容的路径。 若要在一个容器中托管所有静态内容，请将 *origin-path* 替换为相应容器的名称。

若要更深入地了解规则，请参阅 [Azure CDN 规则引擎功能](../../cdn/cdn-rules-engine-reference-features.md)。

## <a name="pricing-and-billing"></a>定价和计费

通过 Azure CDN 访问 Blob 时，需要按 [Blob 存储价格](https://azure.microsoft.com/pricing/details/storage/blobs/)为边缘节点与源（Blob 存储）之间的流量付费。 此外，需要按 [Azure CDN 价格](https://azure.microsoft.com/pricing/details/cdn/)为从边缘节点访问的数据付费。

例如，假设你通过 Azure CDN 访问美国西部的某个存储帐户。 如果英国的某人尝试通过 Azure CDN 访问该存储帐户中的某个 Blob，则 Azure 会首先在距离英国最近的边缘节点中检查该 Blob。 如果 Azure 找到了该 Blob，则会访问副本并使用 Azure CDN 定价，因为 Azure CDN 正在访问该副本。 如果 Azure 未找到该 Blob，则会将该 Blob 复制到边缘节点。 此操作会产生 Blob 存储定价中指定的出口和事务费用。 然后，Azure 会在边缘节点上访问该文件，这会产生 Azure CDN 计费。

在 [Azure CDN 定价页](https://azure.microsoft.com/pricing/details/cdn/)上，对自定义域名的 HTTPS 支持仅适用于来自 Verizon 标准版和高级版产品的 Azure CDN。

## <a name="next-steps"></a>后续步骤

* [为 Blob 存储终结点配置自定义域名](storage-custom-domain-name.md)
* [Azure 存储中的静态网站托管](storage-blob-static-website.md)
