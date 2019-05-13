---
title: Azure 文件的共享快照概述 | Microsoft Docs
description: 作为备份共享的一种方式，共享快照是某个时间点拍摄的 Azure 文件共享的只读版本。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 01/17/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: d83cf20c856d37d337f4eb22c30ee9b6823d096b
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65235815"
---
# <a name="overview-of-share-snapshots-for-azure-files"></a>Azure 文件的共享快照概述 
Azure 文件提供了获取文件共享的共享快照的功能。 共享快照可以捕获在某个时间点的共享状态。 本文介绍共享快照提供的功能，以及如何在自定义用例中加以利用。

## <a name="when-to-use-share-snapshots"></a>何时使用共享快照

### <a name="protection-against-application-error-and-data-corruption"></a>防止应用程序错误和数据损坏
使用文件共享的应用程序执行写入、读取、存储、传输和处理等操作。 如果应用程序配置不当或引入了意外的 bug，某些块可能会出现意外的覆盖或损坏。 为了帮助防止这种情况，可在部署新应用程序代码之前创建共享快照。 如果在新部署中引入了 bug 或应用程序错误，可以恢复到该文件共享中的先前数据版本。 

### <a name="protection-against-accidental-deletions-or-unintended-changes"></a>防止意外删除或意外更改
假设我们要处理文件共享中的某个文本文件。 关闭该文本文件后，无法撤消所做的更改。 在这些情况下，需要恢复文件的先前版本。 使用共享快照可在文件被意外重命名或删除时将其恢复到先前的版本。

### <a name="general-backup-purposes"></a>常规备份目的
创建文件共享后，可以定期创建文件共享的共享快照，以便使用它进行数据备份。 定期创建共享快照有助于维护之前版本的数据，以备未来审计之需或灾难恢复之用。

## <a name="capabilities"></a>功能
共享快照是数据在一个时间点只读副本。 可以使用 REST API 创建、删除和管理快照。 此外，客户端库、Azure CLI 和 Azure 门户中也提供了相同的功能。 

可以使用 REST API 和 SMB 查看共享快照。 可以检索目录或文件的版本列表，可以直接作为驱动程序装载特定版本（仅适用于 Windows - 请参阅[限制](#limits)）。 

在创建共享快照后，可以读取、复制或删除该快照，但无法对其进行修改。 无法将整个共享快照复制到另一个存储帐户。 必须使用 AzCopy 或其他复制机制逐个复制文件。

共享快照功能是在文件共享级别提供的。 检索是在单个文件级别提供的，可用于还原单个文件。 可以使用 SMB、REST API、门户、客户端库或 PowerShell/CLI 工具还原整个文件共享。

文件共享的共享快照与其基本文件共享相同。 唯一的差别在于，共享 URI 的后面追加了一个 **DateTime** 值，用于指示共享快照的创建时间。 例如，如果文件共享 URI 为 http://storagesample.core.file.windows.net/myshare，则共享快照 URI 将类似于：
```
http://storagesample.core.file.windows.net/myshare?snapshot=2011-03-09T01:42:34.9360000Z
```

除非显式删除，否则共享快照会一直保留。 共享快照的生存期不能长于其基本文件共享。 可以枚举与基本文件共享相关联的快照，以跟踪当前快照。 

创建文件共享的共享快照时，共享系统属性中的文件将会复制到具有相同值的共享快照中。 基本文件和文件共享的元数据也会复制到共享快照中，除非在创建共享快照时为其指定了不同的元数据。

除非先删除所有共享快照，否则无法删除具有共享快照的共享。

## <a name="space-usage"></a>空间使用量 
共享快照在本质上是递增的。 只会保存最新共享快照之后发生更改的数据。 这将减少创建共享快照所需的时间，并节省存储成本。 对象或属性或元数据更新操作的任何写入操作，都将被计入“已更改内容”，并存储在共享快照中。 

为了节省空间，可以在改动幅度最大的期间删除共享快照。

尽管共享快照是增量方式进行保存的，但只需保留最新的共享快照即可还原共享。 删除共享快照时，仅删除该共享快照所独有的数据。 活动快照包含浏览和还原数据（从创建共享快照开始）到原始位置或替代位置所需的所有信息。 可以在项级还原。

快照不计入 5-TB 共享限制。 共享快照占用的总空间没有限制。 存储帐户限制仍然适用。

## <a name="limits"></a>限制
Azure 文件目前允许的共享快照的上限是 200 个。 在 200 个共享快照之后，必须删除旧的共享快照，以便创建新的共享快照。 

对创建共享快照的同时调用没有限制。 特定文件共享所能占用的共享快照空间没有限制。 

目前，不能在 Linux 上装载共享快照。 因为 Linux SMB 客户端不支持装载快照，这一点与 Windows 不同。

## <a name="copying-data-back-to-a-share-from-share-snapshot"></a>数据从共享快照复制回共享
涉及文件和共享快照的复制操作遵循以下规则：

可以将文件共享快照中的各个文件复制到其基本共享或任何其他位置。 通过从共享快照中逐个复制文件，可以还原文件的早期版本或还原完整的文件共享。 共享快照不会被提升到基本共享。 

复制后共享快照保持不变，但基本文件共享将被共享快照中可用的数据副本覆盖。 所有还原后的文件会计入“已更改内容”。

可将共享快照中的文件复制到具有不同名称的目标。 生成的目标文件是可写文件，而不是共享快照。

使用副本覆盖目标文件时，与原始目标文件关联的所有共享快照均将保持不变。

## <a name="general-best-practices"></a>常规最佳做法 
在 Azure 上运行基础结构时，尽可能自动备份数据恢复。 自动化操作比手动流程更可靠，有助于提高数据保护和可恢复性。 可以使用 REST API、客户端 SDK 或脚本来实现自动化。

在部署共享快照计划程序之前，仔细考虑共享快照频率和保留设置，以免产生不必要的费用。

共享快照只提供文件级保护。 共享快照无法防止文件共享或存储帐户中的意外删除。 为了帮助防止存储帐户被意外删除，可以锁定存储帐户或资源组。

## <a name="next-steps"></a>后续步骤
- 在以下环境中使用共享快照：
    - [PowerShell](storage-how-to-use-files-powershell.md)
    - [CLI](storage-how-to-use-files-cli.md)
    - [Windows](storage-how-to-use-files-windows.md#accessing-share-snapshots-from-windows)
    - [共享快照常见问题解答](storage-files-faq.md#share-snapshots)
