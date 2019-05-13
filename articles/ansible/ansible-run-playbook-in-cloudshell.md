---
title: 快速入门 - 通过 Azure Cloud Shell 中的 Bash 运行 Ansible Playbook | Microsoft Docs
description: 本快速入门介绍如何使用 Azure Cloud Shell 中的 Bash 执行各种 Ansible 任务
keywords: ansible, azure, devops, bash, cloudshell, playbook, bash
ms.topic: quickstart
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 083d10de94c39ab134b8e475b66ebf2df30088bc
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407649"
---
# <a name="quickstart-run-ansible-playbooks-via-bash-in-azure-cloud-shell"></a>快速入门：通过 Azure Cloud Shell 中的 Bash 运行 Ansible Playbook

Azure Cloud Shell 是一个用于管理 Azure 资源的交互式的可通过浏览器访问的 shell。 Cloud Shell 允许使用 Bash 或 Powershell 命令行。 在本文中，你将使用 Azure Cloud Shell 中的 Bash 运行一个 Ansible Playbook。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
- **配置 Azure Cloud Shell** - 如果你是 Azure Cloud Shell 的新手，请参阅 [Azure Cloud Shell 中的 Bash 快速入门](https://docs.microsoft.com/azure/cloud-shell/quickstart)。
[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="automatic-credential-configuration"></a>自动凭据配置

登录 Cloud Shell 后，Ansible 将通过 Azure 进行身份验证以管理基础架构，无需任何其他配置。 

使用多个订阅时，请通过导出 `AZURE_SUBSCRIPTION_ID` 环境变量来指定 Ansible 使用的订阅。 

要列出所有 Azure 订阅，请运行以下命令：

```azurecli-interactive
az account list
```

按如下所示使用 Azure 订阅 ID 设置 `AZURE_SUBSCRIPTION_ID`：

```azurecli-interactive
export AZURE_SUBSCRIPTION_ID=<your-subscription-id>
```

## <a name="verify-the-configuration"></a>验证配置
若要验证配置是否成功，请使用 Ansible 创建一个 Azure 资源组。

[!INCLUDE [create-resource-group-with-ansible.md](../../includes/ansible-snippet-create-resource-group.md)]

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"] 
> [快速入门：使用 Ansible 在 Azure 中配置虚拟机](/azure/virtual-machines/linux/ansible-create-vm)