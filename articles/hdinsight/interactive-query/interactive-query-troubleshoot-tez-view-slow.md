---
title: Apache Ambari Tez 视图在 Azure HDInsight 中加载缓慢
description: Apache Ambari Tez 视图可能会加载缓慢或根本不会加载到 Azure HDInsight 中
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/30/2019
ms.openlocfilehash: 4fe66b3104be0351a9b0e1df6b6545f71ff276ab
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98930774"
---
# <a name="scenario-apache-ambari-tez-view-loads-slowly-in-azure-hdinsight"></a>方案：Apache Ambari Tez 视图在 Azure HDInsight 中加载缓慢

本文介绍在 Azure HDInsight 群集中使用交互式查询组件时出现的问题的故障排除步骤和可能的解决方案。

## <a name="issue"></a>问题

Apache Ambari Tez 视图可能加载缓慢或根本无法加载。 加载 Ambari Tez 视图时，可能会看到头节点上的进程变得无响应。

## <a name="cause"></a>原因

当群集上运行大量 Hive 作业时，访问 Yarn ATS API 有时可能会使 2017 年 10 月之前创建的群集产生不良性能。

## <a name="resolution"></a>解决方法

此问题已在 2017 年 10 月修复。 重新创建群集后，将不会再遇到此问题。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]