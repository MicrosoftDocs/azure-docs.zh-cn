---
title: 教程 - 缩放 Azure Red Hat OpenShift 群集 | Microsoft Docs
description: 了解如何使用 Azure CLI 缩放 Microsoft Azure Red Hat OpenShift 群集
services: container-service
author: tylermsft
ms.author: twhitney
manager: jeconnoc
ms.topic: tutorial
ms.service: openshift
ms.date: 05/06/2019
ms.openlocfilehash: f86b60fbe20fc630863d6e177d45f2c1f06a7863
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65079392"
---
# <a name="tutorial-scale-an-azure-red-hat-openshift-cluster"></a>教程：缩放 Azure Red Hat OpenShift 群集

本教程是一个系列中的第二部分。 你将了解如何使用 Azure CLI 创建 Microsoft Azure Red Hat OpenShift 群集，对其进行缩放，然后删除该群集以清理资源。

本系列教程的第二部分将介绍如何：

> [!div class="checklist"]
> * 缩放 Red Hat OpenShift 群集

在此系列教程中，你会学习如何：
> [!div class="checklist"]
> * [创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)
> * 缩放 Azure Red Hat OpenShift 群集
> * [删除 Azure Red Hat OpenShift 群集](tutorial-delete-cluster.md)

## <a name="prerequisites"></a>先决条件

在开始学习本教程之前：

* 按照[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程来创建群集。

## <a name="step-1-sign-in-to-azure"></a>步骤 1：登录 Azure

如果在本地运行 Azure CLI，请运行 `az login` 以登录到 Azure。

```bash
az login
```

如果你具有多个订阅的访问权限，请运行 `az account set -s {subscription ID}`，将 `{subscription ID}` 替换为要使用的订阅。

## <a name="step-2-scale-the-cluster-with-additional-nodes"></a>步骤 2：通过额外节点缩放群集

从 Bash 终端，将变量 CLUSTER_NAME 设置为群集的名称：

```bash
CLUSTER_NAME=yourclustername
```

现在，使用 Azure CLI 将群集缩放为五个节点：

```bash
az openshift scale --resource-group $CLUSTER_NAME --name $CLUSTER_NAME --compute-count 5
```

几分钟后，`az openshift scale` 将成功完成并返回包含缩放的群集详细信息的 JSON 文档。

## <a name="next-steps"></a>后续步骤

本教程的此部分介绍了如何：

> [!div class="checklist"]
> * 缩放 Azure Red Hat OpenShift 群集

转到下一教程：
> [!div class="nextstepaction"]
> [删除 Azure Red Hat OpenShift 群集](tutorial-delete-cluster.md)