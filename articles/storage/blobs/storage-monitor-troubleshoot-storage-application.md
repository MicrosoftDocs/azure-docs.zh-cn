---
title: 在 Azure 中监视云存储应用程序并排查其问题 | Microsoft Docs
description: 利用诊断工具、指标和警报来排查云应用程序问题和监视云应用程序。
services: storage
author: normesta
ms.service: storage
ms.topic: tutorial
ms.date: 07/20/2018
ms.author: normesta
ms.reviewer: fryu
ms.custom: mvc
ms.openlocfilehash: 1383ccd570e23d80343ccdfd586d08ee25ebbc8d
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148239"
---
# <a name="monitor-and-troubleshoot-a-cloud-storage-application"></a>监视云存储应用程序并排查其问题

本教程是一个系列的第 4 部分，也是该系列的最后一部分。 本教程介绍如何监视云存储应用程序并排查其问题。

该系列的第 4 部分中介绍了如何：

> [!div class="checklist"]
> * 启用日志记录和指标
> * 启用授权错误警报
> * 使用错误的 SAS 令牌运行测试流量
> * 下载和分析日志

[Azure 存储分析](../common/storage-analytics.md)为存储帐户提供日志记录和指标数据。 此数据提供对存储帐户运行状况的见解。 若要从 Azure 存储分析收集数据，可以配置日志记录、指标和警报。 此过程包括启用日志记录、配置指标和启用警报。

可从 Azure 门户中的“诊断”选项卡启用日志记录和指标。 存储日志记录可用于记录存储帐户中成功和失败请求的相关详细信息。 使用这些日志，可以查看针对 Azure 表、队列和 Blob 的读取、写入和删除操作的详细信息。 它们还可用于查看失败请求的原因，如超时、限制和授权错误。

## <a name="log-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com)

## <a name="turn-on-logging-and-metrics"></a>启用日志记录和指标

在左侧菜单中，依次选择“资源组”、“myResourceGroup”，然后在资源列表中选择存储帐户。

在“诊断设置(经典)”下，将“状态”设置为“开”。 确保“Blob 属性”下的所有选项均已启用。

完成后，单击“保存”

![“诊断”窗格](media/storage-monitor-troubleshoot-storage-application/enable-diagnostics.png)

## <a name="enable-alerts"></a>启用警报

警报可提供一种方法，根据违反阈值的指标向管理员发送电子邮件或触发 Webhook。 此示例启用 `SASClientOtherError` 指标的警报。

### <a name="navigate-to-the-storage-account-in-the-azure-portal"></a>导航到 Azure 门户中的存储帐户

在“监视”部分下，选择“警报(经典)”。

选择“添加指标警报(经典)”并通过填写必需的信息完成“添加规则”窗体。 从“指标”下拉列表中，选择 `SASClientOtherError`。 若要允许警报在出现第一个错误时触发，请从“条件”下拉列表中选择“大于或等于”。

![“诊断”窗格](media/storage-monitor-troubleshoot-storage-application/add-alert-rule.png)

## <a name="simulate-an-error"></a>模拟错误

要模拟一个有效的警报，可尝试从存储帐户中请求不存在的 blob。 下面的命令需要一个存储容器名称。 对于此示例，可以使用现有容器的名称，也可以创建一个新容器。

将占位符替换为实际值（确保将 `<INCORRECT_BLOB_NAME>` 设置为一个不存在的值）并运行该命令。

```azurecli-interactive
sasToken=$(az storage blob generate-sas \
    --account-name <STORAGE_ACCOUNT_NAME> \
    --account-key <STORAGE_ACCOUNT_KEY> \
    --container-name <CONTAINER_NAME> \
    --name <INCORRECT_BLOB_NAME> \
    --permissions r \
    --expiry `date --date="next day" +%Y-%m-%d`)

curl https://<STORAGE_ACCOUNT_NAME>.blob.core.windows.net/<CONTAINER_NAME>/<INCORRECT_BLOB_NAME>?$sasToken
```

下图的示例警报基于上述示例运行的模拟故障。

 ![示例警报](media/storage-monitor-troubleshoot-storage-application/email-alert.png)

## <a name="download-and-view-logs"></a>下载和查看日志

存储日志将数据存储在存储帐户下名为 $logs 的 blob 容器内的一组 blob 中。 如果列出帐户中的所有 blob 容器，则不会显示该容器，但如果直接访问它，则可查看其内容。

此方案使用 [Microsoft Message Analyzer](https://technet.microsoft.com/library/jj649776.aspx) 与 Azure 存储帐户进行交互。

### <a name="download-microsoft-message-analyzer"></a>下载 Microsoft Message Analyzer

下载 [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226) 并安装该应用程序。

启动应用程序并选择“文件” > “打开” > “从其他文件源”。

在“文件选择器”对话框中，选择“+ 添加 Azure 连接”。 输入存储帐户名称和帐户密钥，并单击“确定”。

![Microsoft Message Analyzer -“添加 Azure 存储连接”对话框](media/storage-monitor-troubleshoot-storage-application/figure3.png)

连接后，可在存储树状视图中展开容器，查看日志 blob。 选择最新日志并单击“确定”。

![Microsoft Message Analyzer -“添加 Azure 存储连接”对话框](media/storage-monitor-troubleshoot-storage-application/figure4.png)

在“新建会话”对话框中，单击“开始”查看日志。

打开日志后，便可查看存储事件。 如下图中所示，存储帐户上触发了 `SASClientOtherError`。 有关存储日志记录的详细信息，请参阅[存储分析](../common/storage-analytics.md)。

![Microsoft Message Analyzer - 查看事件](media/storage-monitor-troubleshoot-storage-application/figure5.png)

[存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)是用于与存储帐户进行交互的另一种工具，其中包含 $logs 容器和该容器中所含的日志。

## <a name="next-steps"></a>后续步骤

在该系列的第 4 部分也是最后一部分中，可了解如何监视存储帐户并排查其问题，例如，如何：

> [!div class="checklist"]
> * 启用日志记录和指标
> * 启用授权错误警报
> * 使用错误的 SAS 令牌运行测试流量
> * 下载和分析日志

请访问以下链接，查看预先生成的存储示例。

> [!div class="nextstepaction"]
> [Azure 存储脚本示例](storage-samples-blobs-cli.md)