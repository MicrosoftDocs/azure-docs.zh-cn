---
title: 教程：使用 Azure CDN 通过 SSL 在静态网站上启用自定义域 - Azure 存储
description: 了解如何为静态网站托管功能配置自定义域。
services: storage
author: normesta
ms.service: storage
ms.topic: tutorial
ms.date: 12/07/2018
ms.author: normesta
ms.reviewer: seguler
ms.custom: seodec18
ms.openlocfilehash: 4b074c9dee93ba44659b0321ae3eee7fbea1c61f
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65145056"
---
# <a name="tutorial-use-azure-cdn-to-enable-a-custom-domain-with-ssl-for-a-static-website"></a>教程：使用 Azure CDN 通过 SSL 为静态网站启用自定义域

本教程是一个系列中的第二部分。 本教程介绍如何通过 SSL 为静态网站启用自定义域终结点。 

本教程介绍如何使用 [Azure CDN](../../cdn/cdn-overview.md) 为静态网站配置自定义域终结点。 使用 Azure CDN，可以一次性预配自定义 SSL 证书、使用自定义域以及配置自定义重写规则。 配置 Azure CDN 需额外付费，但用户从世界上的任何位置访问你的网站时，其延迟会始终很低。 Azure CDN 还提供使用你自己的证书进行 SSL 加密的功能。 有关 Azure CDN 定价的详细信息，请参阅 [Azure CDN 定价](https://azure.microsoft.com/pricing/details/cdn/)。

本系列教程的第二部分将介绍如何：

> [!div class="checklist"]
> * 在静态网站终结点上创建 CDN 终结点
> * 启用自定义域和 SSL

## <a name="prerequisites"></a>先决条件

开始学习本教程之前，请完成第一部分：[教程：在 Blob 存储上托管静态网站](storage-blob-static-website-host.md)。 

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com/)即可开始操作。

## <a name="create-a-cdn-endpoint-on-the-static-website-endpoint"></a>在静态网站终结点上创建 CDN 终结点

1. 在 Web 浏览器中打开 [Azure 门户](https://portal.azure.com/)。 
1. 找到存储帐户并显示帐户概览。
1. 在“Blob 服务”菜单下选择“Azure CDN”，以便配置 Azure CDN。
1. 在“新建终结点”部分，填充创建新的 CDN 终结点所需的字段。
1. 输入终结点名称，例如 *mystaticwebsiteCDN*。
1. 输入网站域作为 CDN 终结点的主机名。
1. 对于源主机名，请输入静态网站终结点。 若要查找静态网站终结点，请导航到存储帐户的“静态网站”部分，然后复制该终结点。 
1. 在浏览器中导航到 *mywebsitecdn.azureedge.net*，测试 CDN 终结点。

## <a name="enable-custom-domain-and-ssl"></a>启用自定义域和 SSL

1. 在域名提供商处创建一个 CNAME 记录，以便将自定义域重定向到 CND 终结点。 *www* 子域的 CNAME 记录应该如下所示：

    ![指定子域 www 的 CNAME 记录](media/storage-blob-static-website-custom-domain/subdomain-cname-record.png)

1. 在 Azure 门户中单击新创建的终结点，以便配置自定义域和 SSL 证书。
1. 选择“添加自定义域”，输入域名，然后单击“添加”。
1. 选择新建的自定义域映射，以便预配 SSL 证书。
1. 将“自定义域 HTTPS”设置为“启用”。 选择“CDN 托管”，让 Azure CDN 托管 SSL 证书。 单击“ **保存**”。
1. 访问网站 URL，对网站进行测试。

## <a name="next-steps"></a>后续步骤

本教程的第二部分介绍了如何通过 SSL 在 Azure CDN 中为静态网站配置自定义域。

请访问以下链接，详细了解如何在 Azure 存储上进行静态网站托管。

> [!div class="nextstepaction"]
> [详细了解静态网站](storage-blob-static-website.md)
