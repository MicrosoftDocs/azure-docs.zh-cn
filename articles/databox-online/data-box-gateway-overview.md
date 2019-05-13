---
title: Microsoft Azure Data Box Gateway 概述 | Microsoft Docs
description: 介绍 Azure Data Box Gateway - 一种可以将数据传输到 Azure 的虚拟设备存储解决方案
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: overview
ms.date: 04/08/2019
ms.author: alkohli
ms.openlocfilehash: 2905cae6d5d97062d5a2fc1da41d55290d89fb4a
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64924493"
---
# <a name="what-is-azure-data-box-gateway"></a>什么是 Data Box Gateway？

Azure Data Box Gateway 是一种可以将数据无缝地发送到 Azure 的存储解决方案。 本文概述了 Azure Data Box Gateway 解决方案、优势、关键功能以及可以部署此设备的场景。

Data Box Gateway 是一种虚拟设备，基于在虚拟化环境或虚拟机监控程序中预配的虚拟机。 该虚拟设备驻留在本地，可以使用 NFS 和 SMB 协议将数据写入其中。 然后，设备会将你的数据传输到 Azure 块 blob、页 blob 或 Azure 文件。

## <a name="use-cases"></a>用例

可利用 Data Box Gateway 将数据传输到云，例如云存档、灾难恢复，或根据需要以云规模处理数据。 下面是可以使用 Data Box Gateway 来传输数据的各种场景。

- **云存档** - 使用 Data Box Gateway 以安全有效的方式将数百 TB 的数据复制到 Azure 存储。 对于存档场景，可以一次性或持续地引入数据。

- **持续数据引入** - 无论数据大小如何，都能够将数据持续引入到设备中，以复制到云端。 在将数据写入网关设备时，设备会将数据上传到 Azure 存储。  

- **初始批量传输后进行增量传输** - Data Box 用于在脱机模式下进行批量传输（初始播种），Data Box Gateway 用于在网络上进行增量传输（持续馈送）。

有关详细信息，请转到 [Azure Data Box Gateway 用例](data-box-gateway-use-cases.md)。

## <a name="benefits"></a>优点

Data Box Gateway 具有以下优势：

- **轻松传输数据** - 可以轻松地将数据移入和移出 Azure 存储，就像使用本地网络共享一样简单。  
- **高性能** - 通过与 Azure 之间的高性能传输，避免了网络数据传输的麻烦。
- **营业时间的快速访问和高数据引入率** - Data Box Gateway 有一个本地缓存，可以在预配虚拟设备时将其定义为本地容量大小。 数据磁盘大小应该按[虚拟设备最低要求](data-box-gateway-system-requirements.md#specifications-for-the-virtual-device)进行指定。 本地缓存具有以下优势：
    - 可以通过本地缓存高速引入时间。 在高峰营业时间引入大量数据时，可以由缓存来存储该数据并将其上传到云。
    - 可以通过本地缓存进行快速读取访问，直至达到特定的阈值。 在设备达到 50-60% 的容量之前，从设备进行的所有读取都是通过缓存来访问的，因此速度更快。 在设备上使用的空间超出此阈值后，设备就会开始删除本地文件。
 
- **带宽用量受限** - 即使在营业高峰期网络用量受限，也能将数据写入 Azure。  

## <a name="key-capabilities"></a>关键功能

Data Box Gateway 具有以下功能：

|功能 |说明  |
|---------|---------|
|Speed     | 完全自动化，高度优化的数据传输和带宽。|
|支持的协议     | 支持用于数据引入的标准 SMB 和 NFS 协议。 <br> 有关支持的版本的详细信息，请转到 [Data Box Gateway 系统要求](data-box-gateway-system-requirements.md)。|
|数据访问     | 由设备发送的数据位于云中以后，即可直接访问云 API，对其进行进一步的修改。|
|快速访问     | 设备带有本地缓存，以便快速访问最近使用的文件。|
|离线上传     | 离线模式支持离线上传方案。|
|数据刷新     | 可以使用云中的最新内容刷新本地文件。|
|加密    | BitLocker 支持本地加密数据，并通过 http 安全地将数据传输到云中       |
|复原     | 内置网络复原能力        |


## <a name="specifications"></a>规范

Data Box Gateway 虚拟设备具有以下规范：

| 规范                                          | 说明              |
|---------------------------------------------------------|--------------------------|
| 虚拟处理器（核心数）   | 最低为 4 核 |
| 内存  |最低为 8 GB|
| 可用性|单节点|
| 磁盘|OS 磁盘：250 GB <br> 数据磁盘：最低为 2 TB、预配精简，并且必须由 SSD 提供支持|
| 网络接口 |1 个或多个虚拟网络接口|
| 本机文件共享协议|SMB 和 NFS  |
| 安全|必须进行身份验证才能访问设备和数据 <br> 使用 AES-256 位加密动态数据进行加密|
| 管理|本地 Web UI - 设备的初始设置、诊断和电源管理 <br> Azure 门户 - Data Box Gateway 设备的日常管理       |

## <a name="components"></a>组件

Data Box Gateway 解决方案包括 Data Box Gateway 资源、Data Box Gateway 虚拟设备和本地 Web UI。

- **Data Box Gateway 虚拟设备** - 一种基于在虚拟化环境或虚拟机监控程序中预配的虚拟机的设备，允许将数据发送到 Azure。
    
- **Data Box Gateway 资源** - Azure 门户的资源，使用该资源可以通过 Web 界面（可从不同的地理位置访问该界面）管理 Data Box Gateway 设备。 使用 Data Box Gateway 资源查看和管理设备、共享、用户和警报。 有关详细信息，请参阅如何[使用 Azure 门户进行管理](data-box-gateway-manage-shares.md)。

- **Data Box 本地 Web UI** - 使用本地 Web UI 可以运行诊断、关闭和重启设备、生成支持包，或者联系 Microsoft 支持部门以提交服务请求。 有关详细信息，请参阅如何[使用本地 Web UI 进行管理](data-box-gateway-manage-access-power-connectivity-mode.md)。

## <a name="region-availability"></a>上市区域

将数据传输到的 Data Box Gateway 物理设备、Azure 资源和目标存储帐户不一定非要位于同一区域。

- **资源可用性** - 对于此版本，Data Box Gateway 资源可以在以下支持公有云的区域获得：
    - **美国** - 美国东部
    - **欧盟** - 西欧
    - **亚太** - 东南亚

    Data Box Gateway 也可以部署在 Azure 政府云中。 有关详细信息，请参阅[什么是 Azure 政府？](https://docs.microsoft.com/azure/azure-government/documentation-government-welcome)。

- **目标存储帐户** - 存储数据的存储帐户可在所有 Azure 区域中获得。

    存储帐户存储 Data Box 数据的区域应靠近设备所在位置，以获得最佳性能。 远离设备的存储帐户会导致长时间的延迟和性能下降。


## <a name="next-steps"></a>后续步骤

- 查看 [Data Box Gateway 系统要求](data-box-gateway-system-requirements.md)。
- 了解 [Data Box Gateway 限制](data-box-gateway-limits.md)。
- 在 Azure 门户中部署 [Azure Data Box Gateway](data-box-gateway-deploy-prep.md)。

