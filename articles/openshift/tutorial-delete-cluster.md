---
title: 教程 - 删除 Azure Red Hat OpenShift 群集 | Microsoft Docs
description: 在本教程中，了解如何使用 Azure CLI 删除 Azure Red Hat OpenShift 群集
services: container-service
author: tylermsft
ms.author: twhitney
manager: jeconnoc
ms.topic: tutorial
ms.service: openshift
ms.date: 05/06/2019
ms.openlocfilehash: 679be645f3c97d30df65c9f410d286d243bca73b
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466579"
---
# <a name="tutorial-delete-an-azure-red-hat-openshift-cluster"></a>教程：删除 Azure Red Hat OpenShift 群集

本教程到此结束。 完成测试示例群集后，以下介绍如何删除群集及其相关资源，使你不会因为不使用的资源而支付费用。

在该系列的第三部分中，你会学习如何：

> [!div class="checklist"]
> * 删除 Azure Red Hat OpenShift 群集

在此系列教程中，你会学习如何：
> [!div class="checklist"]
> * [创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)
> * [缩放 Azure Red Hat OpenShift 群集](tutorial-scale-cluster.md)
> * 删除 Azure Red Hat OpenShift 群集

## <a name="prerequisites"></a>先决条件

在开始学习本教程之前：

* 按照[创建 Azure Red Hat OpenShift 群集](tutorial-create-cluster.md)教程来创建群集。

## <a name="step-1-sign-in-to-azure"></a>步骤 1：登录 Azure

如果在本地运行 Azure CLI，请运行 `az login` 以登录到 Azure。

```bash
az login
```

如果你具有多个订阅的访问权限，请运行 `az account set -s {subscription ID}`，将 `{subscription ID}` 替换为要使用的订阅。

## <a name="step-2-delete-the-cluster"></a>步骤 2：删除群集

打开 Bash 终端，并将变量 CLUSTER_NAME 设置为群集的名称：

```bash
CLUSTER_NAME=yourclustername
```

现在，删除群集：

```bash
az openshift delete --resource-group $CLUSTER_NAME --name $CLUSTER_NAME
```

系统将提示你是否要删除该群集。 通过 `y` 进行确认后，删除群集需要几分钟。 此命令完成后，将删除整个资源组以及其中所有资源，包括群集。

## <a name="deleting-a-cluster-using-the-azure-portal"></a>使用 Azure 门户删除群集

或者，可以通过在线 Azure 门户删除相关联的资源组。 资源组的名称与你的群集名称相同。

目前，创建群集时创建的 `Microsoft.ContainerService/openShiftManagedClusters` 资源隐藏在 Azure 门户中。 在 `Resource group` 视图中，勾选 `Show hidden types` 以查看资源组。

![隐藏的类型复选框的屏幕截图](./media/aro-portal-hidden-type.png)

如果删除资源组，将删除构建 Azure Red Hat OpenShift 群集时创建的所有相关资源。

## <a name="next-steps"></a>后续步骤

本教程的此部分介绍了如何：
> [!div class="checklist"]
> * 删除 Azure Red Hat OpenShift 群集

详细了解如何将 OpenShift 与官方 [Red Hat OpenShift 文档](https://docs.openshift.com/aro/welcome/index.html)结合使用