---
title: 快速入门 - 使用 Ansible 管理 Azure 中的 Linux 虚拟机 | Microsoft Docs
description: 在本快速入门中，你将了解如何使用 Ansible 管理 Azure 中的 Linux 虚拟机
keywords: ansible, azure, devops, bash, cloudshell, playbook, bash
ms.topic: quickstart
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: a7862e95966d7b0e0ab31f242dff0244735fe7a1
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409229"
---
# <a name="quickstart-manage-linux-virtual-machines-in-azure-using-ansible"></a>快速入门：使用 Ansible 管理 Azure 中的 Linux 虚拟机

使用 Ansible 可以在环境中自动部署和配置资源。 在本文中，你将使用 Ansible playbook 来启动和停止 Linux 虚拟机。 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [open-source-devops-prereqs-azure-sub.md](../../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="stop-a-virtual-machine"></a>停止虚拟机

在本部分中，你将使用 Ansible 解除分配（停止）Azure 虚拟机。

1. 登录到 [Azure 门户](https://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 打开 [Cloud Shell](/azure/cloud-shell/overview)。

1. 创建一个名为 `azure-vm-stop.yml` 的文件，并在编辑器中将其打开：

    ```azurecli-interactive
    code azure-vm-stop.yml
    ```

1. 将以下示例代码粘贴到编辑器中：

    ```yaml
    - name: Stop Azure VM
      hosts: localhost
      connection: local
      tasks:
        - name: Stop virtual machine
          azure_rm_virtualmachine:
            resource_group: {{ resource_group_name }}
            name: {{ vm_name }}
            allocated: no
    ```

1. 将占位符 `{{ resource_group_name }}` 和 `{{ vm_name }}` 替换成自己的值。

1. 保存文件并退出编辑器。

1. 使用 `ansible-playbook` 命令运行 playbook：

    ```bash
    ansible-playbook azure-vm-stop.yml
    ```

1. 运行 playbook 后，将看到类似于以下结果的输出：

    ```bash
    PLAY [Stop Azure VM] ********************************************************

    TASK [Gathering Facts] ******************************************************
    ok: [localhost]

    TASK [Deallocate the Virtual Machine] ***************************************
    changed: [localhost]

    PLAY RECAP ******************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0
    ```

## <a name="start-a-virtual-machine"></a>启动虚拟机

在本部分中，你将使用 Ansible 启动已解除分配（停止）的 Azure 虚拟机。

1. 登录到 [Azure 门户](https://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 打开 [Cloud Shell](/azure/cloud-shell/overview)。

1. 创建一个名为 `azure-vm-start.yml` 的文件，并在编辑器中将其打开：

    ```azurecli-interactive
    code azure-vm-start.yml
    ```

1. 将以下示例代码粘贴到编辑器中：

    ```yaml
    - name: Start Azure VM
      hosts: localhost
      connection: local
      tasks:
        - name: Start virtual machine
          azure_rm_virtualmachine:
            resource_group: {{ resource_group_name }}
            name: {{ vm_name }}
    ```

1. 将占位符 `{{ resource_group_name }}` 和 `{{ vm_name }}` 替换成自己的值。

1. 保存文件并退出编辑器。

1. 使用 `ansible-playbook` 命令运行 playbook：

    ```bash
    ansible-playbook azure-vm-start.yml
    ```

1. 运行 playbook 后，将看到类似于以下结果的输出：

    ```bash
    PLAY [Start Azure VM] ********************************************************

    TASK [Gathering Facts] ******************************************************
    ok: [localhost]

    TASK [Start the Virtual Machine] ********************************************
    changed: [localhost]

    PLAY RECAP ******************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0
    ```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"] 
> [教程：使用 Ansible 管理 Azure 动态库存](~/articles/ansible/ansible-manage-azure-dynamic-inventories.md)