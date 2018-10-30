---
title: 在 Azure Blockchain Workbench 中使用应用程序
description: 如何在 Azure Blockchain Workbench 中使用应用程序合约。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: 4fe6f164882ffce7bf22ec0c0b94107abcf6a20e
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/03/2018
ms.locfileid: "48240742"
---
# <a name="using-applications-in-azure-blockchain-workbench"></a>在 Azure Blockchain Workbench 中使用应用程序

可以使用 Blockchain Workbench 来创建合约并对其执行操作。 也可查看合约详细信息，例如状态和事务历史记录。

## <a name="prerequisites"></a>先决条件

* Blockchain Workbench 部署。 有关部署的详细信息，请参阅 [Azure Blockchain Workbench 部署](deploy.md)
* Blockchain Workbench 中部署的区块链应用程序。 请参阅 [在 Azure Blockchain Workbench 中创建区块链应用程序](create-app.md)

在浏览器中[打开 Blockchain Workbench](deploy.md#blockchain-workbench-web-url)。

![Blockchain Workbench](./media/use/workbench.png)

需以 Blockchain Workbench 成员身份登录。 如果没有应用程序列出，则表明你是 Blockchain Workbench 的成员，但不是任何应用程序的成员。 Blockchain Workbench 管理员可以向应用程序分配成员。

## <a name="create-new-contract"></a>创建新合约 

若要创建新合约，需要成为指定为合约**发起人**的成员。 有关定义应用程序角色和合约发起人的信息，请参阅[配置中的工作流概述](configuration.md#workflows)。 有关将成员分配给应用程序角色的信息，请参阅[将成员添加到应用程序](manage-users.md#add-member-to-application)。

1. 在 Blockchain Workbench 应用程序部分，选择包含要创建的合约的应用程序磁贴。 此时会显示有效合约的列表。

2. 若要创建新的合约，请选择“新建合约”。

    ![“新建合约”按钮](./media/use/contract-list.png)

3. 此时会显示“新建合约”窗格。 指定初始参数值。 选择**创建**。

    ![“新建合约”窗格](./media/use/new-contract.png)

    新创建的合约与其他有效合约一起显示在列表中。

    ![有效合约列表](./media/use/active-contracts.png)

## <a name="take-action-on-contract"></a>对合约执行操作

根据合约的状态，成员可以采取行动转换到合约的下一个状态。 操作定义为[状态](configuration.md#states)内的[转换](configuration.md#transitions)。 属于转换允许的应用程序或实例角色的成员可以执行操作。 

1. 在 Blockchain Workbench 应用程序部分，选择包含要采取操作的合约的应用程序磁贴。
2. 在列表中选择合约。 有关合约的详细信息显示在不同部分。 

    ![合约详细信息](./media/use/contract-details.png)

    | 部分  | Description  |
    |---------|---------|
    | 状态 | 列出合约各阶段的当前进度 |
    | 详细信息 | 合约的当前值 |
    | 操作 | 有关上一操作的详细信息 |
    | 活动 | 合约的事务历史记录 |
    
3. 在“操作”部分，选择“执行操作”。

4. 有关合约当前状态的详细信息显示在窗格中。 在下拉列表中选择要执行的操作。 

    ![选择操作](./media/use/choose-action.png)

5. 选择“执行操作”以启动操作。
6. 如果操作需要参数，请指定操作的值。

    ![执行操作](./media/use/take-action.png)

7. 选择“执行操作”以执行操作。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [如何排查 Azure Blockchain Workbench 问题](troubleshooting.md)
