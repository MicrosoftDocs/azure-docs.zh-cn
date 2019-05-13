---
title: Microsoft Azure Data Box 磁盘概述 | Microsoft Docs
description: 介绍 Azure Data Box 磁盘 - 用于将大量数据传输到 Azure 的云解决方案
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: overview
ms.date: 01/09/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to understand what Data Box Disk is and how it works so I can use it to import on-premises data into Azure.
ms.openlocfilehash: 20b8d7a6a7d6da639481045d9462c484b820a4df
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64575824"
---
# <a name="what-is-azure-data-box-disk"></a>什么是 Azure Data Box 磁盘？

使用 Microsoft Azure Data Box 磁盘解决方案可以通过快速、经济、可靠的方式将 TB 量级的本地数据发送到 Azure。 Microsoft 会向你寄送 1 到 5 块固态硬盘 (SSD)，帮助加速安全数据传输。 这些加密的 8 TB 磁盘将通过区域承运人寄送到你的数据中心。 

可以通过 Azure 门户中的 Data Box 服务快速配置、连接和解锁这些磁盘。 将数据复制到磁盘，然后将磁盘寄回到 Azure。 在 Azure 数据中心，将会通过高速专用上传链路，将数据自动从驱动器上传到云中。

## <a name="use-cases"></a>用例

在无网络连接或者连接受限的情况下，使用 Data Box 磁盘传输 TB 量级的数据。 数据移动的方式可以是一次性的、定期的，或者先执行批量数据传输，再定期传输。 

- **一次性迁移** - 需要将大量本地数据移到 Azure 时。 例如，将离线磁带中的数据移到 Azure 冷存储中的存档数据。
- **增量传输** - 需要先使用 Data Box 磁盘（种子）执行初始批量传输，然后通过网络执行增量传输时。 例如，使用 Commvault 和 Data Box 磁盘将备份副本移到 Azure。 完成此迁移后，再通过网络将增量数据复制到 Azure 存储。 
- **定期上传** - 定期生成大量数据，并需要将这些数据移到 Azure 时。 例如，能源勘探领域生成有关钻井平台和风力发电农场的视频内容。

## <a name="the-workflow"></a>工作流

典型的流包括以下步骤：

1. **订购** - 在 Azure 门户中创建订单，提供发货信息和数据的目标 Azure 存储帐户。 如果磁盘有货，则 Azure 会加密、准备、寄送磁盘并随附发货跟踪 ID。

2. **接收** - 收到磁盘后，打开包装并将磁盘连接到要复制数据的计算机。 解锁磁盘。
    
3. **复制数据** - 通过拖放操作复制磁盘上的数据。

4. **退回** - 准备好磁盘并寄回到 Azure 数据中心。

5. **上传** - 将磁盘中的数据自动复制到 Azure。 根据美国国家标准和技术协会 (NIST) 的准则安全擦除磁盘数据。

在整个过程中，你会收到有关所有状态更改的电子邮件通知。 有关详细流的详细信息，请参阅[在 Azure 门户中部署 Data Box 磁盘](data-box-disk-quickstart-portal.md)。


## <a name="benefits"></a>优点

Data Box 磁盘旨在将大量数据移到 Azure，且不影响网络。 该解决方案具有以下优点：

- **速度** - Data Box 磁盘使用 USB 3.0 连接，在不到一周内将最多 35 TB 数据移到 Azure。   

- **易用** - Data Box 是一种简单易用的解决方案。

    - 磁盘使用 USB 连接，几乎不需要任何设置。
    - 磁盘的外形规格较小，使之易于操控。
    - 磁盘没有外部电源要求。
    - 可以在数据中心服务器、台式机或便携式计算机上使用这些磁盘。
    - 该解决方案通过 Azure 门户提供端到端的跟踪。    

- **安全** - Data Box 磁盘为磁盘、数据和服务提供内置安全保护。 
    - 磁盘可防篡改，支持安全更新功能。 
    - 磁盘上的数据始终受到 AES 128 位加密的保护。 
    - 只能使用 Azure 门户中提供的密钥解锁磁盘。 
    - 服务受 Azure 安全功能的保护。 
    - 将数据上传到 Azure 后，会根据 NIST 800-88r1 标准完全擦除磁盘数据。  
    
有关详细信息，请参阅 [Azure Data Box 磁盘安全性和数据保护](data-box-disk-security.md)。


## <a name="features-and-specifications"></a>功能和规格


| 规范                                          | 说明              |
|---------------------------------------------------------|--------------------------|
| 重量                                                  | < 0.9 千克 /盒。 每盒最多 5 块磁盘                |
| 维度                                              | 磁盘 - 2.5 英寸 SSD |            
| 电缆                                                  | 每块磁盘配 1 根 USB 3.1 连接线|
| 每笔订单的存储容量                              | 40 TB（可用容量大约为 35 TB）|
| 磁盘存储容量                                   | 8 TB（可用容量大约为 7 TB）|
| 数据接口                                          | USB   |
| 安全                                                | 使用 BitLocker 和安全更新预先加密 <br> 通过支持密钥保护的磁盘 <br> 数据始终加密  |
| 数据传输率                                      | 最高 430 MBps，具体取决于文件大小      |
|管理                                               | Azure 门户 |


## <a name="region-availability"></a>上市区域

目前，Data Box Disk 可将数据传输到以下 Azure 区域：


|Azure 区域  |Azure 区域  |
|---------|---------|
|美国中西部     |加拿大中部       |        
|美国西部 2     |加拿大东部         |     
|美国西部     | 西欧        |      
|美国中南部   |北欧     |         
|美国中部     |澳大利亚东部|
|美国中北部  |澳大利亚东南部   |
|美国东部      |澳大利亚中部 |
|美国东部 2     |澳大利亚中部 2|
|日本东部     |韩国中部 |
|日本西部     |韩国南部 |
|东南亚 |


## <a name="pricing"></a>定价

有关定价的信息，请访问[定价页](https://azure.microsoft.com/pricing/details/databox/disk/)。

## <a name="next-steps"></a>后续步骤

- 查看 [Data Box 磁盘要求](data-box-disk-system-requirements.md)。
- 了解 [Data Box 磁盘限制](data-box-disk-limits.md)。
- 在 Azure 门户中快速部署 [Azure Data Box 磁盘](data-box-disk-quickstart-portal.md)。
