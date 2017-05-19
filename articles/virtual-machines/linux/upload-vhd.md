---
title: "使用 Azure CLI 2.0 上载自定义 Linux 磁盘 | Microsoft 文档"
description: "使用 Resource Manager 部署模型和 Azure CLI 2.0 创建虚拟硬盘 (VHD) 并将其上载到 Azure"
services: virtual-machines-linux
documentationcenter: 
author: iainfoulds
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: a8c7818f-eb65-409e-aa91-ce5ae975c564
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 02/02/2017
ms.author: iainfou
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 1ace4120bd302ba10742bc9cd2af5b788a0537da
ms.lasthandoff: 04/03/2017


---
# <a name="upload-and-create-a-linux-vm-from-custom-disk-with-the-azure-cli-20"></a>使用 Azure CLI 2.0 上载自定义磁盘并从其创建 Linux VM
本文说明如何使用 Azure CLI 2.0 将虚拟硬盘 (VHD) 上载到 Azure，并从此自定义磁盘创建 Linux VM。 还可以使用 [Azure CLI 1.0](upload-vhd-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 执行这些步骤。 此功能可让你安装并配置 Linux 分发以满足你的需求，然后使用该 VHD 快速创建 Azure 虚拟机 (VM)。

## <a name="quick-commands"></a>快速命令
如果需要快速完成任务，请参阅以下部分，其中详细说明了用于将 VHD 上载到 Azure 的基本命令。 本文档的余下部分（[从此处开始](#requirements)）提供了每个步骤的更详细信息和上下文。

确保已安装了最新的 [Azure CLI 2.0](/cli/azure/install-az-cli2) 并已使用 [az login](/cli/azure/#login) 登录到 Azure 帐户。

在以下示例中，请将示例参数名称替换为自己的值。 示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `mydisks`。

首先，使用 [az group create](/cli/azure/group#create) 创建资源组。 以下示例在 `WestUs` 位置创建名为 `myResourceGroup` 的资源组：

```azurecli
az group create --name myResourceGroup --location westus
```

使用 [az storage account create](/cli/azure/storage/account#create) 创建一个用于存放虚拟磁盘的存储帐户。 即使根据 [Azure 托管磁盘概述](../../storage/storage-managed-disks-overview.md)使用 Azure 托管磁盘，也需要先创建一个用作 VHD 上载目标的存储帐户，然后再将 VHD 转换为托管磁盘。 以下示例创建名为 `mystorageaccount` 的存储帐户：

```azurecli
az storage account create --resource-group myResourceGroup --location westus \
  --name mystorageaccount --kind Storage --sku Standard_LRS
```

使用 [az storage account keys list](/cli/azure/storage/account/keys#list) 列出存储帐户的访问密钥。 记下 `key1`：

```azurecli
az storage account keys list --resource-group myResourceGroup --account-name mystorageaccount
```

通过 [az storage container create](/cli/azure/storage/container#create) 使用获取的存储密钥在存储帐户中创建一个容器。 以下示例使用 `key1` 中的存储密钥值创建名为 `mydisks` 的容器：

```azurecli
az storage container create --account-name mystorageaccount \
    --account-key key1 --name mydisks
```

最后，使用 [az storage blob upload](/cli/azure/storage/blob#upload) 将 VHD 上载到创建的容器。 在 `/path/to/disk/mydisk.vhd` 下指定 VHD 的本地路径：

```azurecli
az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd
```

### <a name="azure-managed-disks"></a>Azure 托管磁盘
可以使用 Azure 托管磁盘或非托管磁盘创建 VM。 托管磁盘由 Azure 平台处理，无需任何准备或位置来存储它们。 有关 Azure 托管磁盘的详细信息，请参阅 [Azure 托管磁盘概述](../../storage/storage-managed-disks-overview.md)。 若要从 VHD 创建 VM，请先使用 [az disk create](/cli/azure/disk/create) 将 VHD 转换为托管磁盘：

```azurecli
az disk create --resource-group myResourceGroup --name myManagedDisk \
  --source https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd
```

使用 [az disk list](/cli/azure/disk/list) 获取创建的托管磁盘的详细信息：

```azurecli
az disk list --resource-group myResourceGroup \
  --query [].{Name:name,ID:id} --output table
```

输出类似于以下示例：

```azurecli
Name               ID
-----------------  ----------------------------------------------------------------------------------------------------
myManagedDisk    /subscriptions/mySubscriptionId/resourceGroups/myResourceGroup/providers/Microsoft.Compute/disks/myManagedDisk
```

现在，使用 [az vm create](/cli/azure/vm#create) 创建 VM，并指定托管磁盘的名称 (`--attach-os-disk`)。 以下示例使用基于上载的 VHD 创建的托管磁盘创建名为 `myVM` 的 VM：

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --attach-os-disk myManagedDisk
```

### <a name="unmanaged-disks"></a>非托管磁盘
若要使用非托管磁盘创建 VM，请使用 [az vm create](/cli/azure/vm#create) 指定磁盘的 URI (`--image`)。 以下示例使用前面上载的虚拟磁盘创建名为 `myVM` 的 VM：

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --storage-account mystorageaccount --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --image https://mystorageaccount.blob.core.windows.net/mydisk/myDisks.vhd \
    --use-unmanaged-disk
```

目标存储帐户必须与虚拟磁盘上载到的位置相同。 还需要指定或根据提示输入 **az vm create** 命令所需的所有其他参数，例如虚拟网络、公共 IP 地址、用户名和 SSH 密钥。 阅读有关[可用 CLI Resource Manager 参数](../azure-cli-arm-commands.md#azure-vm-commands-to-manage-your-azure-virtual-machines)的详细信息。

## <a name="requirements"></a>要求
若要完成以下步骤，需要：

* **安装在 .vhd 文件中的 Linux 操作系统** - 将 [Azure 认可的 Linux 分发](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)（或参阅[关于未认可分发的信息](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)）安装在 VHD 格式的虚拟磁盘中。 可使用多种工具创建 VM 和 VHD：
  * 安装并配置 [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) 或 [KVM](http://www.linux-kvm.org/page/RunningKVM)，并注意使用 VHD 作为映像格式。 如果需要，可以使用 `qemu-img convert` [转换映像](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats)。
  * 也可以在 [Windows 10](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install) 或 [Windows Server 2012/2012 R2](https://technet.microsoft.com/library/hh846766.aspx) 上使用 Hyper-V。

> [!NOTE]
> Azure 不支持更新的 VHDX 格式。 创建 VM 时，请将 VHD 指定为映像格式。 如果需要，可以使用 [`qemu-img convert`](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) 或 [`Convert-VHD`](https://technet.microsoft.com/library/hh848454.aspx) PowerShell cmdlet 将 VHDX 磁盘转换为 VHD。 此外，Azure 不支持上载动态 VHD，因此，上载之前，你需要将此类磁盘转换为静态 VHD。 可以使用 [Azure VHD Utilities for GO](https://github.com/Microsoft/azure-vhd-utils-for-go) 等工具在上载到 Azure 的过程中转换动态磁盘。
> 
> 

* 从自定义磁盘创建的 VM 必须位于磁盘本身所在的存储帐户中
  * 创建存储帐户和容器，用于存放自定义磁盘以及所创建的 VM
  * 创建所有 VM 之后，可以安全地删除磁盘

确保已安装了最新的 [Azure CLI 2.0](/cli/azure/install-az-cli2) 并已使用 [az login](/cli/azure/#login) 登录到 Azure 帐户。

在以下示例中，请将示例参数名称替换为自己的值。 示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `mydisks`。

<a id="prepimage"> </a>

## <a name="prepare-the-disk-to-be-uploaded"></a>准备要上载的磁盘
Azure 支持各种 Linux 分发（请参阅 [Endorsed Distributions](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)（认可的分发））。 以下文章将指导你如何准备 Azure 上支持的各种 Linux 分发：

* **[基于 CentOS 的分发版](create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Debian Linux](debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Oracle Linux](oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Red Hat Enterprise Linux](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[SLES 和 openSUSE](suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[Ubuntu](create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**
* **[其他 - 非认可分发](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**

另请参阅 **[Linux 安装说明](create-upload-generic.md#general-linux-installation-notes)**，以获取更多有关如何为 Azure 准备 Linux 映像的一般提示。

> [!NOTE]
> 只有在使用某个认可的分发的时候也使用 [Azure 认可的分发中的 Linux](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 中“支持的版本”下指定的配置详细信息时，[Azure 平台 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) 才适用于运行 Linux 的 VM。
> 
> 

## <a name="create-a-resource-group"></a>创建资源组
资源组以逻辑方式将所有 Azure 资源（例如虚拟网络和存储）聚集在一起，以支持虚拟机。 有关资源组的详细信息，请参阅[资源组概述](../../azure-resource-manager/resource-group-overview.md)。 在上载自定义磁盘和创建 VM 之前，首先需要使用 [az group create](/cli/azure/group#create) 创建一个资源组。

以下示例在 `westus` 位置创建名为 `myResourceGroup` 的资源组：[Azure 托管磁盘概述](../../storage/storage-managed-disks-overview.md)
```azurecli
az group create --name myResourceGroup --location westus
```

## <a name="create-a-storage-account"></a>创建存储帐户
创建 VM 时，可以使用 Azure 托管磁盘或非托管磁盘。 托管磁盘由 Azure 平台处理，无需任何准备或位置来存储它们。 非托管磁盘以页 Blob 形式存储在存储帐户中。 有关详细信息，请参阅 [Azure 托管磁盘概述](../../storage/storage-managed-disks-overview.md)或 [Azure Blob 存储](../../storage/storage-introduction.md#blob-storage)。 即使使用托管磁盘，也需要先创建一个用作 VHD 上载目标的存储帐户，然后再将 VHD 转换为托管磁盘。

可以使用 [az storage account create](/cli/azure/storage/account#create) 为自定义磁盘和 VM 创建存储帐户。 从自定义磁盘创建的、使用非托管磁盘的所有 VM 都必须位于该磁盘所在的同一存储帐户中。 可在订阅中的任何资源组内创建使用托管磁盘的 VM。

以下示例在前面创建的资源组中创建名为 `mystorageaccount` 的存储帐户：

```azurecli
az storage account create --resource-group myResourceGroup --location westus \
  --name mystorageaccount --kind Storage --sku Standard_LRS
```

## <a name="list-storage-account-keys"></a>列出存储帐户密钥
Azure 将为每个存储帐户生成两个 512 位的访问密钥。 在向存储帐户进行身份验证以执行操作（例如执行写入操作）时，将使用这些访问密钥。 从此处了解有关[管理对存储的访问](../../storage/storage-create-storage-account.md#manage-your-storage-account)的详细信息。 可使用 [az storage account keys list](/cli/azure/storage/account/keys#list) 查看访问密钥。

查看创建的存储帐户的访问密钥：

```azurecli
az storage account keys list --resource-group myResourceGroup --account-name mystorageaccount
```

输出类似于：

```azurecli
info:    Executing command storage account keys list
+ Getting storage account keys
data:    Name  Key                                                                                       Permissions
data:    ----  ----------------------------------------------------------------------------------------  -----------
data:    key1  d4XAvZzlGAgWdvhlWfkZ9q4k9bYZkXkuPCJ15NTsQOeDeowCDAdB80r9zA/tUINApdSGQ94H9zkszYyxpe8erw==  Full
data:    key2  Ww0T7g4UyYLaBnLYcxIOTVziGAAHvU+wpwuPvK4ZG0CDFwu/mAxS/YYvAQGHocq1w7/3HcalbnfxtFdqoXOw8g==  Full
info:    storage account keys list command OK
```
记下 `key1`，因为你将在后续步骤中使用它与存储帐户进行交互。

## <a name="create-a-storage-container"></a>创建存储容器
在存储帐户中创建用于整理磁盘的容器的方式，与创建各种目录以便有条理地整理本地文件系统的方式相同。 一个存储帐户可以包含任意数目的容器。 可以使用 [az storage container create](/cli/azure/storage/container#create) 创建容器。

以下示例通过指定上一步骤中获取的访问密钥 (`key1`) 创建名为 `mydisks` 的容器：

```azurecli
az storage container create --account-name mystorageaccount \
    --account-key key1 --name mydisks
```

## <a name="upload-vhd"></a>上载 VHD
现在，使用 [az storage blob upload](/cli/azure/storage/blob#upload) 上载自定义磁盘。 可以页 Blob 的形式上载和存储自定义磁盘。

指定访问密钥、在上一步中创建的容器，以及自定义磁盘在本地计算机上的路径：

```azurecli
az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd
```

## <a name="create-vm-from-custom-disk"></a>从自定义磁盘创建 VM
同样，可以使用 Azure 托管磁盘或非托管磁盘创建 VM。 对于这两种类型的 VM，请在创建 VM 时指定托管磁盘或非托管磁盘的 URI。 对于非托管磁盘，请确保目标存储帐户与自定义磁盘的存储位置匹配。 可以使用 Azure 2.0 或 Resource Manager JSON 模板创建 VM。

### <a name="azure-cli-20---azure-managed-disks"></a>Azure CLI 2.0 - Azure 托管磁盘
若要从 VHD 创建 VM，请先使用 [az disk create](/cli/azure/disk/create) 将 VHD 转换为托管磁盘。 以下示例从已上载到命名存储帐户和容器的 VHD 创建名为 `myManagedDisk` 的托管磁盘：

```azurecli
az disk create --resource-group myResourceGroup --name myManagedDisk \
  --source https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd
```

使用 [az disk list](/cli/azure/disk/list) 获取创建的托管磁盘的 URI：

```azurecli
az disk list --resource-group myResourceGroup \
  --query '[].{Name:name,URI:creationData.sourceUri}' --output table
```

输出类似于以下示例：

```azurecli
Name               URI
-----------------  ----------------------------------------------------------------------------------------------------
myUMDiskFromVHD    https://vhdstoragezw9.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/my_image-osDisk.vhd
```

现在，使用 [az vm create](/cli/azure/vm#create) 创建 VM，并指定托管磁盘的 URI (`--image`)。 以下示例使用基于上载的 VHD 创建的托管磁盘创建名为 `myVM` 的 VM：

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --attach-os-disk myUMDiskFromVHD
```

### <a name="azure-20---unmanaged-disks"></a>Azure 2.0 - 非托管磁盘
若要使用非托管磁盘创建 VM，请使用 [az vm create](/cli/azure/vm#create) 指定磁盘的 URI (`--image`)。 以下示例使用前面上载的虚拟磁盘创建名为 `myVM` 的 VM：

在 [az vm create](/cli/azure/vm#create) 中指定 `--image` 参数，指向自定义磁盘。 确保 `--storage-account` 与用于存储自定义磁盘的存储帐户匹配。 不需要使用与自定义磁盘相同的容器来存储 VM。 上载自定义磁盘之前，请确保使用前面步骤中所述的相同方式创建任何附加容器。

以下示例从自定义磁盘创建名为 `myVM` 的 VM：

```azurecli
az vm create --resource-group myResourceGroup --location westus \
    --name myVM --storage-account mystorageaccount --os-type linux \
    --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
    --image https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd \
    --use-unmanaged-disk
```

仍需要指定或根据提示输入 **az vm create** 命令所需的所有其他参数，例如用户名和 SSH 密钥。


### <a name="resource-manager-template---unmanaged-disks"></a>Resource Manager 模板 - 非托管磁盘
Azure Resource Manager 模板是用于定义你希望生成的环境的 JavaScript 对象表示法 (JSON) 文件。 这些模板细分为不同的资源提供程序，如计算或网络。 你可以使用现有模板，也可以编写自己的模板。 阅读有关[使用 Resource Manager 和模板](../../azure-resource-manager/resource-group-overview.md)的详细信息。

在模板的 `Microsoft.Compute/virtualMachines` 提供程序中有一个 `storageProfile` 节点，其中包含 VM 的配置详细信息。 需要编辑的两个主要参数为 `image` 和 `vhd` URI，它们指向自定义磁盘和新 VM 的虚拟磁盘。 下面显示了使用自定义磁盘的 JSON 示例：

```json
"storageProfile": {
          "osDisk": {
            "name": "myVM",
            "osType": "Linux",
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "image": {
              "uri": "https://mystorageaccount.blob.core.windows.net/mydisks/myDisk.vhd"
            },
            "vhd": {
              "uri": "https://mystorageaccount.blob.core.windows.net/vhds/newvmname.vhd"
            }
          }
```

可以使用[此现有模板从自定义映像创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)，或阅读有关[创建自己的 Azure Resource Manager 模板](../../azure-resource-manager/resource-group-authoring-templates.md)的信息。 

配置模板之后，使用 [az group deployment create](/cli/azure/group/deployment#create) 创建 VM。 使用 `--template-uri` 参数指定 JSON 模板的 URI：

```azurecli
az group deployment create --resource-group myNewResourceGroup \
  --template-uri https://uri.to.template/mytemplate.json
```

如果在计算机上以本地方式存储了一个 JSON 文件，则可以改用 `--template-file` 参数：

```azurecli
az group deployment create --resource-group myNewResourceGroup \
  --template-file /path/to/mytemplate.json
```


## <a name="next-steps"></a>后续步骤
准备好并上载自定义虚拟磁盘之后，可以阅读有关[使用 Resource Manager 和模板](../../azure-resource-manager/resource-group-overview.md)的详细信息。 可能还需要向新 VM [添加数据磁盘](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 如果需要访问在 VM 上运行的应用程序，请务必[打开端口和终结点](nsg-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。


