---
title: Azure 存储中的静态网站托管
description: Azure 存储静态网站托管提供经济高效、可缩放的解决方案用于托管新式 Web 应用程序。
services: storage
author: normesta
ms.service: storage
ms.topic: article
ms.author: normesta
ms.reviewer: seguler
ms.date: 04/29/2019
ms.subservice: blobs
ms.openlocfilehash: cd1fa71cb2a10c7e61f76bdd224ba6d0f039346f
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148475"
---
# <a name="static-website-hosting-in-azure-storage"></a>Azure 存储中的静态网站托管
使用 Azure 存储 GPv2 帐户可以直接通过名为 *$web* 的存储容器提供静态内容（HTML、CSS、JavaScript 和图像文件）。 利用 Azure 存储中的托管，可以使用无服务器体系结构，包括 [Azure Functions](/azure/azure-functions/functions-overview) 和其他 PaaS 服务。

与静态网站托管相比，依赖于服务器端代码的动态站点最适合使用 [Azure 应用服务](/azure/app-service/overview)来托管。

## <a name="how-does-it-work"></a>工作原理
在存储帐户中启用静态网站托管时，请选择默认文件的名称，并选择性地提供自定义 404 页面的路径。 启用该功能后，将创建名为 *$web* 的容器（如果不存在）。

*$web* 容器中的文件：

- 通过匿名访问请求来提供
- 仅通过对象读取操作来提供
- 区分大小写
- 可遵循以下模式在公共 Web 中提供：
    - `https://<ACCOUNT_NAME>.<ZONE_NAME>.web.core.windows.net/<FILE_NAME>`
- 可遵循以下模式通过 Blob 存储终结点来提供：
    - `https://<ACCOUNT_NAME>.blob.core.windows.net/$web/<FILE_NAME>`

使用 Blob 存储终结点上传文件。 例如，将文件上传到此位置：

```bash
https://contoso.blob.core.windows.net/$web/image.png
```

在如下位置的浏览器中提供：

```bash
https://contoso.z4.web.core.windows.net/image.png
```

如果未提供文件名，选定的默认文件名将在根目录和任何子目录中使用。 如果服务器返回了 404 错误，并且你未提供错误文档路径，则向用户返回默认的 404 页面。

> [!NOTE]
> 文件的默认公共访问级别是私有的。 因为文件通过匿名访问请求提供服务，将忽略此设置。 没有公共访问权限的所有文件，并且 RBAC 权限将被忽略。

## <a name="cdn-and-ssl-support"></a>CDN 和 SSL 支持

若要使你的静态网站文件通过自定义域和 HTTPS 上可用，请参阅[使用 Azure CDN 通过 HTTPS 访问包含自定义域的 blob](storage-https-custom-domain-cdn.md)。 在此过程中，需将 CDN 指向 Web 终结点而不是 Blob 终结点。 由于 CDN 配置不会立即执行，因此，可能需要等待几分钟才能看到内容。

当更新静态网站时，请确保通过清除 CDN 终结点清除缓存在 CDN 边缘服务器上的内容。 有关详细信息，请参阅[清除 Azure CDN 终结点](../../cdn/cdn-purge-endpoint.md)。

> [!NOTE]
> 通过帐户 web 终结点以本机方式支持 HTTPS。 通过 HTTPS 自定义域使用要求在此时间的 Azure CDN 使用。 
>
> 通过 HTTPS 公共帐户 web 终结点： `https://<ACCOUNT_NAME>.<ZONE_NAME>.web.core.windows.net/<FILE_NAME>`

## <a name="custom-domain-names"></a>自定义域名

可[为 Azure 存储帐户配置自定义域名](storage-custom-domain-name.md)，以便通过自定义域提供静态网站。 有关如何在 Azure 中托管域的深入信息，请参阅[在 Azure DNS 中托管域](../../dns/dns-delegate-domain-azure-dns.md)。

## <a name="pricing"></a>定价
启用静态网站托管是免费。 客户收取地利用的 blob 存储和操作成本。 如需详细了解 Azure Blob 存储价格，请参阅 [Azure Blob 存储定价页](https://azure.microsoft.com/pricing/details/storage/blobs/)。

## <a name="quickstart"></a>快速入门

### <a name="azure-portal"></a>Azure 门户
首先打开 Azure 门户 [https://portal.azure.com](https://portal.azure.com)，然后在 GPv2 存储帐户中运行以下步骤：

1. 单击“设置”
2. 单击“静态网站”
3. 输入索引文档名称。 （常用值为 *index.html*）
4. （可选）输入自定义 404 页面的错误文档路径。 （常用值为 *404.html*）

![](media/storage-blob-static-website/storage-blob-static-website-portal-config.PNG)

接下来，通过 Azure 门户将资产上传到 *$web* 容器，或使用 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)上传整个目录。 请确保包含的文件与启用该功能时所选的索引文档名称匹配。

最后，转到 Web 终结点来测试网站。

### <a name="azure-cli"></a>Azure CLI
安装存储预览扩展：

```azurecli-interactive
az extension add --name storage-preview
```
如果存在多个订阅，请将 CLI 设置为要启用的 GPv2 存储帐户的订阅：

```azurecli-interactive
az account set --subscription <SUBSCRIPTION_ID>
```
启用该功能。 请务必将所有占位符值（包括括号）替换为自己的值：

```azurecli-interactive
az storage blob service-properties update --account-name <ACCOUNT_NAME> --static-website --404-document <ERROR_DOCUMENT_NAME> --index-document <INDEX_DOCUMENT_NAME>
```
查询 Web 终结点 URL：

```azurecli-interactive
az storage account show -n <ACCOUNT_NAME> -g <RESOURCE_GROUP> --query "primaryEndpoints.web" --output tsv
```

将对象从源目录上传到 $web 容器。 请确保正确地转义命令中对 $web 容器的引用。 例如，如果在 Azure 门户中使用 CloudShell 中的 Azure CLI，请转义 $web 容器，如下所示：

```azurecli-interactive
az storage blob upload-batch -s <SOURCE_PATH> -d \$web --account-name <ACCOUNT_NAME>
```

## <a name="deployment"></a>部署

可用于将内容部署到存储容器的方法包括：

- [AzCopy](../common/storage-use-azcopy.md)
- [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)
- [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/)
- [Visual Studio Code 扩展](https://code.visualstudio.com/tutorials/static-website/getting-started)

在所有情况下，请确保将文件复制到 *$web* 容器。

## <a name="metrics"></a>度量值

若要在静态网站页面上启用指标，请单击“设置” > “监视” > “指标”。

将通过挂接到不同的指标 API 来生成指标数据。 门户只会显示在给定时间范围内使用的 API 成员，以便重点关注可返回数据的成员。 为确保能够选择所需的 API 成员，第一步是展开时间范围。

单击时间范围按钮，选择“过去 24 小时”，然后单击“应用”

![Azure 存储静态网站指标 - 时间范围](./media/storage-blob-static-website/storage-blob-static-website-metrics-time-range.png)

接下来，从“命名空间”下拉列表中选择“Blob”。

![Azure 存储静态网站指标 - 命名空间](./media/storage-blob-static-website/storage-blob-static-website-metrics-namespace.png)

然后选择“传出”指标。

![Azure 存储静态网站指标 - 指标](./media/storage-blob-static-website/storage-blob-static-website-metrics-metric.png)

从“聚合”选择器中选择“总和”。

![Azure 存储静态网站指标 - 聚合](./media/storage-blob-static-website/storage-blob-static-website-metrics-aggregation.png)

接下来，单击“添加筛选器”按钮，并从“属性”选择器中选择“API 名称”。

![Azure 存储静态网站指标 - API 名称](./media/storage-blob-static-website/storage-blob-static-website-metrics-api-name.png)

最后，在“值”选择器中选中“GetWebContent”旁边的复选框，以填充指标报告。

![Azure 存储静态网站指标 - GetWebContent](./media/storage-blob-static-website/storage-blob-static-website-metrics-getwebcontent.png)

启用后，指标仪表板会报告有关 $web 容器中的文件的流量统计信息。

## <a name="faq"></a>常见问题解答

**静态网站功能是否适用于所有存储帐户类型？**  
否，静态网站托管仅适用于 GPv2 标准存储帐户。

**新 Web 终结点是否支持存储 VNET 和防火墙规则？**  
是，新 Web 终结点遵循为存储帐户配置的 VNET 和防火墙规则。

**Web 终结点是区分大小写？**  
是的，Web 终结点区分大小写，就像 Blob 终结点一样。

**是 web 终结点可通过 HTTP 和 HTTPS 访问？**
是的可通过 HTTP 和 HTTPS 访问 web 终结点。 但是，如果存储帐户配置为通过 HTTPS 需要安全传输，用户必须使用 HTTPS 终结点。 有关详细信息，请参阅[需要在 Azure 存储中的安全传输](../common/storage-require-secure-transfer.md)。

## <a name="next-steps"></a>后续步骤
* [使用 Azure CDN 通过 HTTPS 访问包含自定义域的 Blob](storage-https-custom-domain-cdn.md)
* [为 blob 或 Web 终结点配置自定义域名](storage-custom-domain-name.md)
* [Azure Functions](/azure/azure-functions/functions-overview)
* [Azure 应用服务](/azure/app-service/overview)
* [生成首个无服务器 Web 应用程序](https://docs.microsoft.com/azure/functions/tutorial-static-website-serverless-api-with-database)
* [教程：在 Azure DNS 中托管域](../../dns/dns-delegate-domain-azure-dns.md)
