---
title: Azure Blockchain Workbench 故障排除
description: 如何解决 Azure Blockchain Workbench 应用程序。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/09/2019
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: b0263761a4aaf663b16584fbf9caa11bb124d5c4
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510093"
---
# <a name="azure-blockchain-workbench-troubleshooting"></a>Azure Blockchain Workbench 故障排除

PowerShell 脚本用于协助开发人员进行调试或提供支持。 此脚本生成摘要并收集进行故障排除所需的详细日志。 收集的日志包括：

* Blockchain 网络，例如 Ethereum
* Blockchain Workbench 微服务
* Application Insights
* Azure 监视 （Azure Monitor 日志）

可以根据此信息确定后续步骤和问题的根本原因。

## <a name="troubleshooting-script"></a>故障排除脚本

PowerShell 故障排除脚本在 GitHub 上提供。 从 GitHub [下载 zip 文件](https://github.com/Azure-Samples/blockchain/archive/master.zip)或克隆该示例。

```
git clone https://github.com/Azure-Samples/blockchain.git
```

## <a name="run-the-script"></a>运行脚本
[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install.md)]

运行 `collectBlockchainWorkbenchTroubleshooting.ps1` 脚本可收集日志并创建一个 ZIP 文件，该文件包含一个文件夹，其中有故障排除信息。 例如:

``` powershell
collectBlockchainWorkbenchTroubleshooting.ps1 -SubscriptionID "<subscription_id>" -ResourceGroupName "workbench-resource-group-name"
```
此脚本接受以下参数：

| 参数  | 描述 | 需要 |
|---------|---------|----|
| 订阅 ID | SubscriptionID，用于创建或定位所有资源。 | “是” |
| ResourceGroupName | Blockchain Workbench 部署时所在的 Azure 资源组的名称。 | “是” |
| OutputDirectory | 用于创建输出 .ZIP 文件的路径。 如果未指定，则默认为当前目录。 | “否” |
| LookbackHours | 拉取遥测数据时要使用的小时数。 默认值为 24 小时。 最大值为 90 小时 | “否” |
| OmsSubscriptionId | 部署 Azure 监视器将记录其中的订阅 ID。 如果外 Blockchain Workbench 的资源组部署区块链网络的 Azure Monitor 日志，仅将此参数传递。| “否” |
| OmsResourceGroup |部署 Azure 监视器将记录其中的资源组。 如果外 Blockchain Workbench 的资源组部署区块链网络的 Azure Monitor 日志，仅将此参数传递。| “否” |
| OmsWorkspaceName | Log Analytics 工作区名称。 仅传递此参数，如果外 Blockchain Workbench 的资源组部署区块链网络的 Azure Monitor 日志 | “否” |

## <a name="what-is-collected"></a>收集什么内容？

输出 ZIP 文件包含以下文件夹结构：

| 文件夹或文件 | 描述  |
|---------|---------|
| \Summary.txt | 系统摘要 |
| \Metrics\blockchain | 有关区块链的指标 |
| \Metrics\Workbench | 有关 Workbench 的指标 |
| \Details\Blockchain | 有关区块链的详细日志 |
| \Details\Workbench | 有关 Workbench 的详细日志 |

摘要文件提供了应用程序总体状态的快照以及应用程序的运行状况。 此摘要提供建议的操作，突出显示了顶级错误以及正在运行的服务的相关元数据。

**Metrics** 文件夹包含各种系统组件随时间变化的指标。 例如，输出文件 `\Details\Workbench\apiMetrics.txt` 包含不同响应代码的摘要，以及整个收集期间的响应时间。 **Details** 文件夹包含用于解决 Workbench 或底层区块链网络特定问题的详细日志。 例如，`\Details\Workbench\Exceptions.csv` 包含系统中最近发生的异常列表，这对于解决智能合同或与区块链交互的错误很有用。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench Application Insights 故障排除指南](https://aka.ms/workbenchtroubleshooting)
