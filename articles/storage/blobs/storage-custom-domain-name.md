---
title: 为 Azure 存储帐户配置自定义域名 | Microsoft Docs
description: 使用 Azure 门户将自己的规范名称 (CNAME) 映射到 Azure 存储帐户中的 Blob 存储或 Web 终结点。
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.date: 06/26/2018
ms.author: normesta
ms.reviewer: seguler
ms.subservice: blobs
ms.openlocfilehash: 4f6776a5f15cf391f3a65aceb6e9e783d87a2078
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148930"
---
# <a name="configure-a-custom-domain-name-for-your-azure-storage-account"></a>为 Azure 存储帐户配置自定义域名

可以配置自定义域以便访问 Azure 存储帐户中的 Blob 数据。 Azure Blob 存储的默认终结点是 *\<storage-account-name>.blob.core.windows.net*。 此外可以使用生成的 web 终结点的一部分[静态网站功能](storage-blob-static-website.md)。 如果你将自定义域和子域，如*www\.contoso.com*，到你的存储帐户的 blob 或 web 终结点，你的用户可以使用该域访问存储帐户中的 blob 数据。

> [!IMPORTANT]
> Azure 存储尚不支持以本机方式对自定义域使用 HTTPS。 目前可以[使用 Azure CDN 通过 HTTPS 访问包含自定义域的 Blob](storage-https-custom-domain-cdn.md)。
> 
> 
> [!NOTE]
> 暂仅支持为每个存储帐户配置一个自定义域名。 无法将自定义域名映射到 Web 和 Blob 服务终结点。
> 
> [!NOTE]
> 映射仅用于子域 (例如 www\.contoso.com)。 如果你想要 web 终结点提供对根级域 (例如 contoso.com)，则必须为[的自定义域使用 Azure CDN](storage-https-custom-domain-cdn.md)

下表显示了 *mystorageaccount* 存储帐户中的 Blob 数据的一些示例 URL。 为存储帐户注册的自定义子域是 *www\.contoso.com*：

| 资源类型 | 默认 URL | 自定义域 URL |
| --- | --- | --- |
| 存储帐户 | http://mystorageaccount.blob.core.windows.net | http://www.contoso.com |
| Blob |http://mystorageaccount.blob.core.windows.net/mycontainer/myblob | http://www.contoso.com/mycontainer/myblob |
| 根容器 | http://mystorageaccount.blob.core.windows.net/myblob 或 http://mystorageaccount.blob.core.windows.net/$root/myblob| http://www.contoso.com/myblob 或 http://www.contoso.com/$root/myblob |
| Web |  http://mystorageaccount.[zone].web.core.windows.net/$web/[indexdoc] 或 http://mystorageaccount.[zone].web.core.windows.net/[indexdoc] 或 http://mystorageaccount.[zone].web.core.windows.net/$web 或 http://mystorageaccount.[zone].web.core.windows.net/ | http://www.contoso.com/$web 或 http://www.contoso.com/ 或 http://www.contoso.com/$web/[indexdoc] 或 http://www.contoso.com/[indexdoc] |

> [!NOTE]  
> 如以下部分中所示，Blob 服务终结点的所有示例同样适用于 Web 服务终结点。

## <a name="direct-vs-intermediary-cname-mapping"></a>直接 CNAME 映射与中间 CNAME 映射

可通过下述两种方法之一将带有子域（例如 www\.contoso.com）前缀的自定义域指向存储帐户的 Blob 终结点： 
* 使用直接 CNAME 映射。
* 使用 *asverify* 中间子域。

### <a name="direct-cname-mapping"></a>直接 CNAME 映射

第一种方法是创建一个规范名称 (CNAME) 记录。此记录可以将自定义域和子域直接映射到 Blob 终结点。这种方法最简单。 CNAME 记录是一种域名系统 (DNS) 功能，用于将源域映射到目标域。 在示例中，源域是你自己的自定义域和子域（例如 *www\.contoso.com*）。 目标域是 Blob 服务终结点（例如 *mystorageaccount.blob.core.windows.net*）。

“注册自定义域”部分介绍了直接方法。

### <a name="intermediary-mapping-with-asverify"></a>使用 asverify 进行中间映射

第二种方法也使用 CNAME 记录。 但是，为了避免停机，此方法先采用了可由 Azure 识别的特殊子域 *asverify*。

将自定义域映射到 Blob 终结点会导致在 [Azure 门户](https://portal.azure.com)中注册域时出现短暂的停机。 如果该域目前所支持的应用程序的服务级别协议 (SLA) 要求不能有停机时间，请使用 Azure *asverify* 子域作为中间注册步骤。 此步骤可确保用户能够在 DNS 映射期间访问域。

使用 asverify 子域注册自定义域中介绍了中间方法。

## <a name="register-a-custom-domain"></a>注册自定义域
如果符合以下陈述，则可以使用本部分所述的过程来注册域：
* 你不担心域暂时对用户不可用。
* 自定义域当前未托管应用程序。 

可以使用 Azure DNS 为 Azure Blob 存储配置自定义 DNS 名称。 有关详细信息，请参阅[使用 Azure DNS 为 Azure 服务提供自定义域设置](https://docs.microsoft.com/azure/dns/dns-custom-domain#blob-storage)。

如果自定义域目前在支持不能有任何停机时间的应用程序，请遵循使用 asverify 子域注册自定义域中的过程。

若要配置自定义域名，请在 DNS 中创建一个新的 CNAME 记录。 该 CNAME 记录指定了域名的别名。 本示例将自定义域的地址映射到存储帐户的 Blob 存储终结点。

通常可以在域注册机构的网站上管理域的 DNS 设置。 每个注册机构指定 CNAME 记录的方法类似但略有不同，但概念是相同的。 由于某些基本域注册程序包不提供 DNS 配置，因此可能需要首先升级域注册程序包，才能创建 CNAME 记录。

1. 在 [Azure 门户](https://portal.azure.com)中转到自己的存储帐户。

1. 在菜单窗格中的“Blob 服务”下，选择“自定义域”。  
   此时会打开“自定义域”窗格。

1. 登录到域注册机构的网站，并转到用于管理 DNS 的页面。  
   可在名为“域名”、“DNS”或“名称服务器管理”的部分中找到该页。

1. 找到用于管理 CNAME 的部分。  
   可能需要转到高级设置页，并查找“CNAME”、“别名”或“子域”。

1. 创建新的 CNAME 记录，输入 **www** 或 **photos** 等子域别名（子域是必需的，根域不受支持），然后提供主机名。  
   主机名是 Blob 服务终结点。 主机名格式为 *\<mystorageaccount>.blob.core.windows.net*，其中，*mystorageaccount* 是存储帐户的名称。 要使用的主机名显示在 [Azure 门户](https://portal.azure.com)的“自定义域”窗格的第 1 项中。 

1. 在“自定义域”窗格中的文本框内，输入自定义域的名称，包括子域。  
   例如，如果域是 *contoso.com*，子域别名是 *www*，请输入 **www\.contoso.com**。 如果子域是 *photos*，则请输入 **photos.contoso.com**。

1. 若要注册自定义域，请选择“保存”。  
   如果注册成功，则门户会通知存储帐户已成功更新。

新的 CNAME 记录通过 DNS 传播后，如果用户具有相应的权限，则他们可以使用自定义域查看 Blob 数据。

## <a name="register-a-custom-domain-by-using-the-asverify-subdomain"></a>使用 *asverify* 子域注册自定义域
如果自定义域目前支持的应用程序的 SLA 要求不能停机，请使用本部分所述的过程注册自定义域。 通过创建从 *asverify.\<subdomain>.\<customdomain>* 指向 *asverify.\<storageaccount>.blob.core.windows.net* 的 CNAME，可以预先向 Azure 注册域。 然后，可以创建从 *\<subdomain>.\<customdomain>* 指向 *\<storageaccount>.blob.core.windows.net* 的另一个 CNAME，发往自定义域的流量将定向到 Blob 终结点。

*asverify* 子域是 Azure 能够识别的一个特殊子域。 将 *asverify* 追加到自己的子域可使 Azure 识别自定义域，且无需修改该域的 DNS 记录。 修改该域的 DNS 记录后，它将映射到 Blob 终结点，并且没有故障时间。

1. 在 [Azure 门户](https://portal.azure.com)中转到自己的存储帐户。

1. 在菜单窗格中的“Blob 服务”下，选择“自定义域”。  
   此时会打开“自定义域”窗格。

1. 登录到 DNS 提供程序的网站，并转到用于管理 DNS 的页面。  
   可在名为“域名”、“DNS”或“名称服务器管理”的部分中找到该页。

1. 找到用于管理 CNAME 的部分。  
   可能需要转到高级设置页，并查找“CNAME”、“别名”或“子域”。

1. 创建新的 CNAME 记录，并且提供包含 *asverify* 子域的子域别名（例如 **asverify.www** 或 **asverify.photos**），然后提供主机名。  
   主机名是 Blob 服务终结点。 主机名格式为 *asverify.\<mystorageaccount>.blob.core.windows.net*，其中，*mystorageaccount* 是存储帐户的名称。 要使用的主机名显示在 [Azure 门户](https://portal.azure.com)的“自定义域”窗格的第 2 项中。

1. 在“自定义域”窗格中的文本框内，输入自定义域的名称，包括子域。  
   请不要包含 asverify 例如，如果域是 *contoso.com*，子域别名是 *www*，请输入 **www\.contoso.com**。 如果子域是 *photos*，则请输入 **photos.contoso.com**。

1. 选中“使用间接 CNAME 验证”复选框。

1. 若要注册自定义域，请选择“保存”。  
   如果注册成功，则门户会通知存储帐户已成功更新。 自定义域已由 Azure 验证，但发往域的流量尚未路由到存储帐户。

1. 返回到 DNS 提供程序的网站，创建将子域映射到 Blob 服务终结点的另一条 CNAME 记录。  
   例如，将子域指定为 *www* 或 *photos*（不含 *asverify*），将主机名指定为 *\<mystorageaccount>.blob.core.windows.net*（其中，*mystorageaccount* 是存储帐户名称）。 完成此步骤后，也就完成了自定义域的注册。

1. 最后，可以删除新建的包含 *asverify* 的 CNAME 记录，因为只在中间步骤中才需要用到它。

新的 CNAME 记录通过 DNS 传播后，如果用户具有相应的权限，则他们可以使用自定义域查看 Blob 数据。

## <a name="test-your-custom-domain"></a>测试自定义域

若要确认自定义域是否映射到了 Blob 服务终结点，请在存储帐户中的公共容器内创建一个 Blob。 然后在 Web 浏览器中，使用以下格式的 URI 来访问该 Blob：`http://<subdomain.customdomain>/<mycontainer>/<myblob>`

例如，若要访问 *photos.contoso.com* 自定义子域中的 *myforms* 容器内的 Web 窗体：可使用以下 URI：`http://photos.contoso.com/myforms/applicationform.htm`

## <a name="deregister-a-custom-domain"></a>取消注册自定义域

若要取消注册 Blob 存储终结点的自定义域，请使用以下过程之一。

### <a name="azure-portal"></a>Azure 门户

若要删除自定义域设置，请执行以下操作：

1. 在 [Azure 门户](https://portal.azure.com)中转到自己的存储帐户。

1. 在菜单窗格中的“Blob 服务”下，选择“自定义域”。  
   此时会打开“自定义域”窗格。

1. 清除包含自定义域名的文本框的内容。

1. 选择“保存”按钮。

成功删除自定义域后，会看到一条门户通知，指出存储帐户已成功更新。

### <a name="azure-cli"></a>Azure CLI

若要删除自定义域注册，请使用 [az storage account update](https://docs.microsoft.com/cli/azure/storage/account) CLI 命令，并为 `--custom-domain` 参数值指定空字符串 (`""`)。

* 命令格式：

  ```azurecli
  az storage account update \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --custom-domain ""
  ```

* 命令示例：

  ```azurecli
  az storage account update \
      --name mystorageaccount \
      --resource-group myresourcegroup \
      --custom-domain ""
  ```

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

若要删除自定义域注册，请使用 [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) PowerShell cmdlet，并为 `-CustomDomainName` 参数值指定空字符串 (`""`)。

* 命令格式：

  ```powershell
  Set-AzStorageAccount `
      -ResourceGroupName "<resource-group-name>" `
      -AccountName "<storage-account-name>" `
      -CustomDomainName ""
  ```

* 命令示例：

  ```powershell
  Set-AzStorageAccount `
      -ResourceGroupName "myresourcegroup" `
      -AccountName "mystorageaccount" `
      -CustomDomainName ""
  ```

## <a name="next-steps"></a>后续步骤
* [将自定义域映射到 Azure 内容分发网络 (CDN) 终结点](../../cdn/cdn-map-content-to-custom-domain.md)
* [使用 Azure CDN 通过 HTTPS 访问包含自定义域的 Blob](storage-https-custom-domain-cdn.md)
* [Azure Blob 存储中的静态网站托管（预览版）](storage-blob-static-website.md)
