---
title: 配置 Azure HPC 缓存聚合命名空间
description: 如何通过 Azure HPC 缓存为后端存储创建面向客户端的路径
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 12/22/2020
ms.author: v-erkel
ms.openlocfilehash: 5549670dbd1f302bdb17b8b94cbd1fb5c4c1a1d9
ms.sourcegitcommit: 6cca6698e98e61c1eea2afea681442bd306487a4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/24/2020
ms.locfileid: "97760517"
---
# <a name="set-up-the-aggregated-namespace"></a>设置聚合命名空间

创建存储目标后，还必须为其创建命名空间路径。 客户端计算机使用这些虚拟路径通过缓存访问文件，而不是直接连接到后端存储。 此系统允许缓存管理员更改后端存储系统，而无需重写客户端说明。

有关此功能的详细信息，请参阅 [计划聚合命名空间](hpc-cache-namespace.md) 。

Azure 门户中的 " **命名空间** " 页显示客户端用于通过缓存访问数据的路径。 使用此页可以创建、删除或更改命名空间路径。 你还可以使用 Azure CLI 配置命名空间路径。

已为此缓存定义的所有面向客户端的路径都列在 " **命名空间** " 页上。 表中未定义任何命名空间路径的存储目标。

您可以对表中的列进行排序，以便更好地了解缓存的聚合命名空间。 单击列标题中的箭头以对路径进行排序。

[![表中包含两个路径的门户命名空间页的屏幕截图。列标题：命名空间路径、存储目标、导出路径和导出子目录，以及客户端访问策略。第一列中的路径名称是可单击的链接。顶部按钮：添加命名空间路径，刷新，删除 ](media/namespace-page.png)](media/namespace-page.png#lightbox)

## <a name="add-or-edit-namespace-paths"></a>添加或编辑命名空间路径

你必须至少创建一个命名空间路径，然后客户端才能访问存储目标。 有关客户端访问的详细信息，请参阅 [安装 AZURE HPC 缓存](hpc-cache-mount.md) (。 ) 

### <a name="blob-namespace-paths"></a>Blob 命名空间路径

Azure Blob 存储目标只能有一个命名空间路径。

按照以下说明设置或更改 Azure 门户或 Azure CLI 的路径。

### <a name="portal"></a>[Portal](#tab/azure-portal)

在 Azure 门户中，加载 " **命名空间** 设置" 页。 您可以在此页中添加、更改或删除命名空间路径。

* **添加新路径：** 单击顶部的 " **+ 添加** " 按钮，并在 "编辑" 面板中填写信息。

  ![选择了 blob 存储目标的添加命名空间编辑字段的屏幕截图。 导出和子目录路径设置为/且不可编辑。](media/namespace-add-blob.png)

  * 输入客户端将用于访问此存储目标的路径。

  * 选择要用于此路径的访问策略。 若要详细了解如何在中自定义客户端访问， [请使用客户端访问策略](access-policies.md)。

  * 从下拉列表中选择存储目标。 如果 blob 存储目标已有命名空间路径，则无法选择它。

  * 对于 Azure Blob 存储目标，导出和子目录路径会自动设置为 ``/`` 。

* **更改现有路径：** 单击命名空间路径。 此时将打开 "编辑" 面板。 您可以修改路径和访问策略，但不能更改为其他存储目标。

* **删除命名空间路径：** 选中该路径左侧的复选框，然后单击 " **删除** " 按钮。

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[设置 AZURE HPC 缓存的 Azure CLI](./az-cli-prerequisites.md)。

使用 Azure CLI 时，必须在创建存储目标时添加命名空间路径。 有关详细信息，请参阅 [添加新的 Azure Blob 存储目标](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-azure-blob-storage-target) 。

若要更新目标的命名空间路径，请使用 [az hpc-缓存 blob-存储-目标更新](/cli/azure/ext/hpc-cache/hpc-cache/blob-storage-target#ext-hpc-cache-az-hpc-cache-blob-storage-target-update) 命令。 Update 命令的参数与 create 命令中的参数类似，不同之处在于不传递容器名称或存储帐户。

不能使用 Azure CLI 从 blob 存储目标删除命名空间路径，但可以使用不同的值覆盖该路径。

---

### <a name="nfs-namespace-paths"></a>NFS 命名空间路径

NFS 存储目标可以有多个虚拟路径，只要每个路径代表同一个存储系统上的不同导出或子目录。

为 NFS 存储目标规划命名空间时，请记住，每个路径必须唯一，并且不能是另一个命名空间路径的子目录。 例如，如果您有一个名为的命名空间路径 ``/parent-a`` ，则您也不能创建类似于和的命名空间路径 ``/parent-a/user1`` ``/parent-a/user2`` 。 这些目录路径在命名空间中已作为的子目录来访问 ``/parent-a`` 。

NFS 存储系统的所有命名空间路径都是在一个存储目标上创建的。 大多数缓存配置最多可支持10个存储目标的命名空间路径，但较大的配置最多可支持20个。

此列表显示了每个配置的最大命名空间路径数。

* 最高 2 GB/秒的吞吐量：

  * 3 TB 缓存-10 个命名空间路径
  * 6 TB 缓存-10 个命名空间路径
  * 12 TB 缓存-20 个命名空间路径

* 最大 4 GB/秒的吞吐量：

  * 6 TB 缓存-10 个命名空间路径
  * 12 TB 缓存-10 个命名空间路径
  * 24 TB 缓存-20 命名空间路径

* 最多 8 GB/秒的吞吐量：

  * 12 TB 缓存-10 个命名空间路径
  * 24 TB 缓存-10 命名空间路径
  * 48 TB 缓存-20 命名空间路径

对于每个 NFS 命名空间路径，提供面向客户端的路径和存储系统导出，还可以选择导出子目录。

### <a name="portal"></a>[Portal](#tab/azure-portal)

在 Azure 门户中，加载 " **命名空间** 设置" 页。 您可以在此页中添加、编辑或删除命名空间路径。

* **若要添加新路径：** 单击顶部的 " **+ 添加** " 按钮，并在 "编辑" 面板中填写信息。
* **更改现有路径：** 单击命名空间路径。 此时将打开 "编辑" 面板，您可以修改路径。
* **删除命名空间路径：** 选中该路径左侧的复选框，然后单击 " **删除** " 按钮。

对于每个命名空间路径，请填写以下值：

* **命名空间路径** -面向客户端的文件路径。

* **客户端访问策略** -选择要用于此路径的访问策略。 若要详细了解如何在中自定义客户端访问， [请使用客户端访问策略](access-policies.md)。

* **存储目标** -如果要创建新的命名空间路径，请从下拉菜单中选择存储目标。

* **导出路径** -输入 NFS 导出的路径。 请确保正确键入导出名称-门户将验证此字段的语法，但不会检查导出，直到提交更改。

* **导出子目录** -如果希望此路径装入导出的特定子目录，请在此处输入。 否则，请将此字段留空。

!["门户命名空间" 页的屏幕截图，其中打开了 "编辑" 页。 编辑窗体显示 nfs 存储目标命名空间路径的设置](media/namespace-edit-nfs.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[设置 AZURE HPC 缓存的 Azure CLI](./az-cli-prerequisites.md)。

使用 Azure CLI 时，必须在创建存储目标时至少添加一个命名空间路径。 有关详细信息，请参阅 [添加新的 NFS 存储目标](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-nfs-storage-target) 。

若要更新目标的命名空间路径或添加其他路径，请使用 [az hpc-缓存 nfs-存储更新](/cli/azure/ext/hpc-cache/hpc-cache/nfs-storage-target#ext-hpc-cache-az-hpc-cache-nfs-storage-target-update) 命令。 使用 ``--junction`` 选项指定所需的所有命名空间路径。

用于 update 命令的选项与 "create" 命令类似，不同之处在于，你未将存储系统信息传递 (IP 地址或主机名) ，使用模型是可选的。 有关选项的语法的更多详细信息，请参阅 [添加新的 NFS 存储目标](hpc-cache-add-storage.md?tabs=azure-cli#add-a-new-nfs-storage-target) ``--junction`` 。

---

## <a name="next-steps"></a>后续步骤

为存储目标创建聚合命名空间后，可以在缓存上装载客户端。 阅读以下文章以了解详细信息。

* [装载 Azure HPC 缓存](hpc-cache-mount.md)
* [将数据移动到 Azure Blob 存储](hpc-cache-ingest.md)
