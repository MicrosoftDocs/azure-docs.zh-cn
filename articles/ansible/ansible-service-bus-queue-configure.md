---
title: 教程 - 使用 Ansible 配置 Azure 服务总线中的队列 | Microsoft Docs
description: 了解如何使用 Ansible 创建 Azure 服务总线队列
keywords: ansible, azure, devops, bash, playbook, 服务总线, 队列
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 6efc11106fae18beac43ab1896733ab6bfc64dad
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230769"
---
# <a name="tutorial-configure-queues-in-azure-service-bus-using-ansible"></a>教程：使用 Ansible 配置 Azure 服务总线中的队列

[!INCLUDE [ansible-28-note.md](../../includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-servicebus.md](../../includes/open-source-devops-intro-servicebus.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * 创建队列
> * 创建 SAS 策略
> * 检索命名空间信息
> * 检索队列信息
> * 撤销队列 SAS 策略

## <a name="prerequisites"></a>先决条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-the-service-bus-queue"></a>创建服务总线队列

示例 playbook 代码将创建以下资源：
- Azure 资源组
- 资源组中的服务总线命名空间
- 带命名空间的服务总线队列

将以下 playbook 保存为 `servicebus_queue.yml`：

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      location: eastus
      namespace: servicebustestns
      queue: servicebustestqueue
  tasks:
    - name: Ensure resource group exist
      azure_rm_resourcegroup:
          name: "{{ resource_group }}"
          location: "{{ location }}"
    - name: Create a namespace
      azure_rm_servicebus:
          name: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
    - name: Create a queue
      azure_rm_servicebusqueue:
          name: "{{ queue }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
      register: queue
    - debug:
          var: queue
```

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook servicebus_queue.yml
```

## <a name="create-the-sas-policy"></a>创建 SAS 策略

[共享访问签名 (SAS)](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) 是使用令牌的基于声明的授权机制。 

示例 playbook 代码为一个服务总线队列创建具有不同权限的两个 SAS 策略。

将以下 playbook 保存为 `servicebus_queue_policy.yml`：

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      queue: servicebustestqueue
  tasks:
    - name: Create a policy with send and listen priviledge
      azure_rm_servicebussaspolicy:
          name: "{{ queue }}-policy"
          queue: "{{ queue }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          rights: listen_send
      register: policy
    - debug:
          var: policy
```

在运行 playbook 之前，请参阅以下说明：
- `rights` 值表示用户对队列具有的权限。 请指定下列值之一：`manage`、`listen`、`send` 或 `listen_send`。

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook servicebus_queue_policy.yml
```

## <a name="retrieve-namespace-information"></a>检索命名空间信息

示例 playbook 代码将查询命名空间信息。

将以下 playbook 保存为 `servicebus_namespace_info.yml`：

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
  tasks:
    - name: Get a namespace's information
      azure_rm_servicebus_facts:
          type: namespace
          name: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          show_sas_policies: yes
      register: ns
    - debug:
          var: ns
```

在运行 playbook 之前，请参阅以下说明：
- `show_sas_policies` 值指示是否在指定的命名空间下显示 SAS 策略。 默认情况下，值为 `False`，以避免额外的网络开销。

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook servicebus_namespace_info.yml
```

## <a name="retrieve-queue-information"></a>检索队列信息

示例 playbook 代码将查询队列信息。 

将以下 playbook 保存为 `servicebus_queue_info.yml`：

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      queue: servicebustestqueue
  tasks:
    - name: Get a queue's information
      azure_rm_servicebus_facts:
          type: queue
          name: "{{ queue }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          show_sas_policies: yes
      register: queue
    - debug:
          var: queue
```

在运行 playbook 之前，请参阅以下说明：
- `show_sas_policies` 值指示是否在指定的队列下显示 SAS 策略。 默认情况下，此值设置为 `False`，以避免额外的网络开销。

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook servicebus_queue_info.yml
```

## <a name="revoke-the-queue-sas-policy"></a>撤销队列 SAS 策略

示例 playbook 代码将删除队列 SAS 策略。

将以下 playbook 保存为 `servicebus_queue_policy_delete.yml`：

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      queue: servicebustestqueue
  tasks:
    - name: Create a policy with send and listen priviledge
      azure_rm_servicebussaspolicy:
          name: "{{ queue }}-policy"
          queue: "{{ queue }}"
          namespace: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          state: absent
```

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook servicebus_queue_policy_delete.yml
```

## <a name="clean-up-resources"></a>清理资源

如果不再需要本教程中创建的资源，请将其删除。 

将以下代码保存为 `cleanup.yml`：

```yml
---
- hosts: localhost
  vars:
      resource_group: servicebustest
      namespace: servicebustestns
      queue: servicebustestqueue
  tasks:
    - name: Delete queue
      azure_rm_servicebusqueue:
          name: "{{ queue }}"
          resource_group: "{{ resource_group }}"
          namespace: "{{ namespace }}"
          state: absent
    - name: Delete namespace
      azure_rm_servicebus:
          name: "{{ namespace }}"
          resource_group: "{{ resource_group }}"
          state: absent
    - name: Delete resource group
      azure_rm_resourcegroup:
          name: "{{ resource_group }}"
          state: absent
          force_delete_nonempty: yes
```

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook cleanup.yml
```

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"] 
> [教程：使用 Ansible 配置 Azure 服务总线中的主题](ansible-service-bus-topic-configure.md)