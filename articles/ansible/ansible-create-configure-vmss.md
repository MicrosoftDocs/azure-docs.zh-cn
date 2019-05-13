---
title: 教程 - 使用 Ansible 在 Azure 中配置虚拟机规模集 | Microsoft Docs
description: 了解如何在 Azure 中使用 Ansible 创建和配置虚拟机规模集
keywords: ansible, azure, devops, bash, playbook, 虚拟机, 虚拟机规模集, vmss
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 41ef6103a899970142df1a6beee0ad428419f3df
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230741"
---
# <a name="tutorial-configure-virtual-machine-scale-sets-in-azure-using-ansible"></a>教程：使用 Ansible 在 Azure 中配置虚拟机规模集

[!INCLUDE [ansible-27-note.md](../../includes/ansible-27-note.md)]

[!INCLUDE [open-source-devops-intro-vmss.md](../../includes/open-source-devops-intro-vmss.md)]

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * 为 VM 配置资源
> * 配置规模集
> * 通过增加 VM 实例来扩展规模集 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="configure-a-scale-set"></a>配置规模集

本部分的 playbook 代码定义以下资源：

* 所有资源将部署到的**资源组**。
* 10.0.0.0/16 地址空间中的**虚拟网络**
* 虚拟网络中的**子网**
* 用于通过 Internet 访问资源的**公共 IP 地址**
* **网络安全组**，控制传入和传出规模集的网络流量
* **负载均衡器**，使用负载均衡器规则将流量分配到一组定义的 VM
* 使用所有已创建资源的**虚拟机规模集**

有两种方式可获取相同的 playbook：

* [下载 playbook](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-create.yml) 并将其保存到 `vmss-create.yml`。
* 新建名为 `vmss-create.yml` 的文件，并将以下内容复制到其中：

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_name: myScaleSet
    vmss_lb_name: myScaleSetLb
    location: eastus
    admin_username: azureuser
    admin_password: "{{ admin_password }}"
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
    - name: Create virtual network
      azure_rm_virtualnetwork:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        address_prefixes: "10.0.0.0/16"
    - name: Add subnet
      azure_rm_subnet:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        address_prefix: "10.0.1.0/24"
        virtual_network: "{{ vmss_name }}"
    - name: Create public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group }}"
        allocation_method: Static
        name: "{{ vmss_name }}"
    - name: Create Network Security Group that allows SSH
      azure_rm_securitygroup:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        rules:
          - name: SSH
            protocol: Tcp
            destination_port_range: 22
            access: Allow
            priority: 1001
            direction: Inbound

    - name: Create a load balancer
      azure_rm_loadbalancer:
        name: "{{ vmss_lb_name }}"
        location: "{{ location }}"
        resource_group: "{{ resource_group }}"
        public_ip: "{{ vmss_name }}"
        probe_protocol: Tcp
        probe_port: 8080
        probe_interval: 10
        probe_fail_count: 3
        protocol: Tcp
        load_distribution: Default
        frontend_port: 80
        backend_port: 8080
        idle_timeout: 4
        natpool_frontend_port_start: 50000
        natpool_frontend_port_end: 50040
        natpool_backend_port: 22
        natpool_protocol: Tcp

    - name: Create Scale Set
      azure_rm_virtualmachinescaleset:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        vm_size: Standard_DS1_v2
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        ssh_password_enabled: true
        capacity: 2
        virtual_network_name: "{{ vmss_name }}"
        subnet_name: "{{ vmss_name }}"
        upgrade_policy: Manual
        tier: Standard
        managed_disk_type: Standard_LRS
        os_disk_caching: ReadWrite
        image:
          offer: UbuntuServer
          publisher: Canonical
          sku: 16.04-LTS
          version: latest
        load_balancer: "{{ vmss_lb_name }}"
        data_disks:
          - lun: 0
            disk_size_gb: 20
            managed_disk_type: Standard_LRS
            caching: ReadOnly
          - lun: 1
            disk_size_gb: 30
            managed_disk_type: Standard_LRS
            caching: ReadOnly
```

在运行 playbook 之前，请参阅以下说明：

* 在 `vars` 节中，请将 `{{ admin_password }}` 占位符替换为你自己的密码。

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook vmss-create.yml
```

运行 playbook 后，将看到类似于以下结果的输出：

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create a resource group] 
changed: [localhost]

TASK [Create virtual network] 
changed: [localhost]

TASK [Add subnet] 
changed: [localhost]

TASK [Create public IP address] 
changed: [localhost]

TASK [Create Network Security Group that allows SSH] 
changed: [localhost]

TASK [Create a load balancer] 
changed: [localhost]

TASK [Create Scale Set] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=8    changed=7    unreachable=0    failed=0

```

## <a name="view-the-number-of-vm-instances"></a>查看 VM 实例数目。

[配置的规模集](#configure-a-scale-set)当前有两个实例。 使用以下步骤确认该值：

1. 登录到 [Azure 门户](https://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 导航到配置的规模集。

1. 将会看到规模集名称以及括号中的实例数目：`Standard_DS1_v2 (2 instances)`

1. 还可以通过运行以下命令，使用 [Azure Cloud Shell](https://shell.azure.com/) 检查实例数目：

    ```azurecli-interactive
    az vmss show -n myScaleSet -g myResourceGroup --query '{"capacity":sku.capacity}' 
    ```

    在 Cloud Shell 中运行 Azure CLI 命令的结果显示现在存在三个实例： 

    ```bash
    {
      "capacity": 3,
    }
    ```

## <a name="scale-out-a-scale-set"></a>横向扩展规模集

本部分的 playbook 代码检索有关规模集的信息，并将其容量从 2 更改为 3。

有两种方式可获取相同的 playbook：

* [下载 playbook](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-scale-out.yml) 并将其保存到 `vmss-scale-out.yml`。
* 新建名为 `vmss-scale-out.yml` 的文件，并将以下内容复制到其中：

```yml
- hosts: localhost
  vars:
    resource_group: myResourceGroup
    vmss_name: myScaleSet
  tasks: 
    - name: Get scaleset info
      azure_rm_virtualmachine_scaleset_facts:
        resource_group: "{{ resource_group }}"
        name: "{{ vmss_name }}"
        format: curated
      register: output_scaleset

    - name: Dump scaleset info
      debug:
        var: output_scaleset

    - name: Modify scaleset (change the capacity to 3)
      set_fact:
        body: "{{ output_scaleset.ansible_facts.azure_vmss[0] | combine({'capacity': 3}, recursive=True) }}"

    - name: Update something in that scale set
      azure_rm_virtualmachinescaleset: "{{ body }}"
```

使用 `ansible-playbook` 命令运行 playbook：

```bash
ansible-playbook vmss-scale-out.yml
```

运行 playbook 后，将看到类似于以下结果的输出：

```Output
PLAY [localhost] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Get scaleset info] 
ok: [localhost]

TASK [Dump scaleset info] 
ok: [localhost] => {
    "output_scaleset": {
        "ansible_facts": {
            "azure_vmss": [
                {
                    ......
                }
            ]
        },
        "changed": false,
        "failed": false
    }
}

TASK [Modify scaleset (set upgradePolicy to Automatic and capacity to 3)] 
ok: [localhost]

TASK [Update something in that scale set] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=5    changed=1    unreachable=0    failed=0
```

## <a name="verify-the-results"></a>验证结果

通过 Azure 门户验证操作结果：

1. 登录到 [Azure 门户](https://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 导航到配置的规模集。

1. 将会看到规模集名称以及括号中的实例数目：`Standard_DS1_v2 (3 instances)` 

1. 还可以通过运行以下命令，使用 [Azure Cloud Shell](https://shell.azure.com/) 验证更改：

    ```azurecli-interactive
    az vmss show -n myScaleSet -g myResourceGroup --query '{"capacity":sku.capacity}' 
    ```

    在 Cloud Shell 中运行 Azure CLI 命令的结果显示现在存在三个实例： 

    ```bash
    {
      "capacity": 3,
    }
    ```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"] 
> [教程：使用 Ansible 将应用部署到 Azure 中的虚拟机规模集](./ansible-deploy-app-vmss.md)