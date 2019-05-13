---
title: Microsoft Azure Data Box 常见问题解答 | 关于数据的 Microsoft Docs
description: 包含有关 Azure Data Box（用于将大量数据传输到 Azure 的云解决方案）的常见问题和解答。
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: overview
ms.date: 05/08/2019
ms.author: alkohli
ms.openlocfilehash: 3fca94a56f80ec520ac6777610a90bdddf1b8988
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473263"
---
# <a name="azure-data-box-frequently-asked-questions"></a>Azure Data Box：常见问题

利用 Microsoft Azure Data Box 混合云解决方案，可使用传输设备通过快速、经济和可靠的方式将 TB 量级的数据发送到 Azure。 本常见问题解答文章包含了在 Azure 门户使用 Data Box 时可能遇到的问题及其解答。 

问题和解答分为以下几个类别：

- 关于该服务
- 订购设备
- 配置和连接 
- 跟踪状态
- 复制数据 
- 运送设备
- 验证和上传数据 
- 监管支持链

## <a name="about-the-service"></a>关于该服务

### <a name="q-what-is-azure-data-box-service"></a>问： 什么是 Azure Data Box 服务？ 
A.  Azure Data Box 服务专为脱机数据引入而设计。 此服务可以管理针对数据传输定制的、具有不同存储容量的所有产品阵列。 

### <a name="q-what-is-azure-data-box"></a>问： 什么是 Azure Data Box？
A. 使用 Azure Data Box，可以快速、廉价且安全地将数 TB 的数据传输到 Azure。 可通过 Azure 门户订购 Data Box 设备。 Microsoft 通过区域承运人向你寄送具有 80 TB 可用容量的存储设备。 

收到设备后，可使用本地 Web UI 进行快速设置。 将数据从服务器复制到设备，然后将设备发回 Azure。 在 Azure 数据中心内，数据将从设备自动上传到 Azure。 通过 Azure 门户中的 Data Box 服务对整个过程进行端到端跟踪。

### <a name="q-when-should-i-use-data-box"></a>问： 何时应使用 Data Box？
A. 如果需要将 40 - 500 TB 的数据传输到 Azure，则使用 Data Box 会很有利。 对于 < 40 TB 的数据大小，请使用 Data Box Disk，对于 > 500 TB 的数据大小，请注册 [Data Box Heavy](data-box-heavy-overview.md)。

### <a name="q-what-is-the-price-of-data-box"></a>问： Data Box 的价格是多少？
A. 使用 Data Box 10 天会产生少许费用。 在 Azure 门户中创建订单并选择产品型号时，将显示设备的费用。 寄送也是免费的，但是，Azure 存储会产生费用。 有关详细信息，请转到 [Azure Data Box 定价](https://azure.microsoft.com/pricing/details/storage/databox/)。 

### <a name="q-what-is-the-maximum-amount-of-data-i-can-transfer-with-data-box-in-one-instance"></a>问： 在一个实例中使用 Data Box 最多可以传输多少数据？
A. Data Box 的原始容量为 100 TB，可用容量为 80 TB。 使用 Data Box 最多可传输 80 TB 数据。 若要传输更多数据，需订购更多设备。

### <a name="q-how-can-i-check-if-data-box-is-available-in-my-region"></a>问： 如何确认 Data Box 在我的区域是否可用？ 
A.  有关 Data Box 在哪些国家/区域可用的信息，请转到[区域可用性](data-box-overview.md#region-availability)。  

### <a name="q-which-regions-can-i-store-data-in-with-data-box"></a>问： 可在哪些区域使用 Data Box 存储数据？
A. 美国、西欧、北欧、法国和英国的所有区域支持 Data Box。 仅支持 Azure 公有云区域。 不支持 Azure 政府版或其他主权云。 有关详细信息，请参阅[适用区域](data-box-overview.md#region-availability)。

### <a name="q-whom-should-i-contact-if-i-encounter-any-issues-with-data-box"></a>问： 如果 Data Box 出现任何问题，应该与谁联系？
A. 如果 Data Box 出现任何问题，请联系 [Microsoft 支持部门](data-box-disk-contact-microsoft-support.md)。

### <a name="q-i-have-lost-my-data-box-is-there-a-lost-device-charge"></a>问： 我丢失了我的 Data Box。 丢失的设备是否也要付费？
A. 是的。 丢失或损坏的设备也会产生费用。 [定价页](https://azure.microsoft.com/pricing/details/storage/databox/)和[产品服务条款](https://www.microsoft.com/licensing/product-licensing/products)中提到了这笔费用。


## <a name="order-device"></a>订购设备

### <a name="q-how-do-i-get-data-box"></a>问： 如何获得 Data Box？ 
A.  要想获得 Azure Data Box，请登录 Azure 门户并创建 Data Box 订单。 提供联系信息和通知详细信息。 提交订单后，我们会在 10 天内将 Data Box 送达，具体时间取决于 Data Box 的供货情况。 有关详细信息，请转到[订购 Data Box](data-box-deploy-ordered.md)。

### <a name="q-i-was-not-able-to-create-a-data-box-order-in-the-azure-portal-why-would-this-be"></a>问： 我无法在 Azure 门户中创建 Data Box 订单。 这是为什么？
A. 如果无法创建 Data Box 订单，有可能是订阅类型或访问权限的问题。 

请查看你的订阅。 Data Box 仅适用于企业协议 (EA) 和云解决方案提供商 (CSP) 订阅产品/服务。 如果订阅不属于以上任何类型，请联系 Microsoft 支持部门升级订阅。

如果具有受支持的订阅产品/服务类型，请查看订阅访问级别。 只有订阅的参与者或所有者才能创建订单。

### <a name="q-i-ordered-a-couple-of-data-box-devices-i-am-not-able-to-create-any-additional-orders-why-would-this-be"></a>问： 我订购了多台 Data Box 设备。 我无法创建其他任何订单。 这是为什么？
A. 每个订阅在每个商务区域（国家和地区的组合）最多允许有五个有效订单。 如果需要订购更多设备，请联系 Microsoft 支持部门提高订阅限制。

### <a name="q-when-i-try-to-create-an-order-i-receive-a-notification-that-the-data-box-service-is-not-available-what-does-this-mean"></a>问： 我在尝试创建订单时收到通知，提示 Data Box 服务不可用。 这是什么意思呢？
A. 这表示 Data Box 服务在你所选的国家和地区组合中不可用。 更改此组合可能会允许使用 Data Box 服务。 有关该服务适用区域的列表，请转到 [Data Box 的适用区域](data-box-overview.md#region-availability)。

### <a name="q-i-placed-my-data-box-order-few-days-back-when-will-i-receive-my-data-box"></a>问： 我数天前订购了 Data Box。 何时能收到 Data Box？
A. 客户下单后，我们会检查订购的设备是否有货。 如果设备有货，我们将在 10 天内寄送。 也可能有高需求时期。 在此情况下，订单将进行排队，可在 Azure 门户中跟踪状态更改。 如果订单未在 90 天内完成，则会自动取消。

### <a name="q-i-have-filled-up-my-data-box-with-data-and-need-to-order-another-one-is-there-a-way-to-quickly-place-the-order"></a>问： 我的 Data Box 已填满数据，需要再订购一台。 是否有某种方法可让我快速下单？
A. 可以克隆以前的订单。 克隆操作会创建与以前相同的订单，并允许编辑订单详细信息（仅限这些信息），而无需键入地址、联系人和通知详细信息。

## <a name="configure-and-connect"></a>配置和连接

### <a name="q-how-do-i-unlock-the-data-box"></a>问： 如何解锁 Data Box？ 
A.  在 Azure 门户中，转到 Data Box 订单，并导航到“设备详细信息”。 复制解锁密码。 在 Data Box 上使用此密码登录本地 Web UI。 有关详细信息，请转到[教程：为 Azure Data Box 拆除包装、连接电缆并连接到它](data-box-deploy-set-up.md)。

### <a name="q-can-i-use-a-linux-host-computer-to-connect-and-copy-the-data-on-to-the-data-box"></a>问： 是否可以使用 Linux 主机建立连接并将数据复制到 Data Box？
A.  是的。 可以使用 Data Box 连接 SMB 和 NFS 客户端。 有关详细信息，请转到主机的[受支持操作系统](data-box-system-requirements.md)列表。

### <a name="q-my-data-box-is-dispatched-but-now-i-want-to-cancel-this-order-why-is-the-cancel-button-not-available"></a>问： 我的 Data Box 已发货，但现在我想取消此订单。 为何取消按钮不可用？
A.  订购 Data Box 后，只能在处理订单前取消。 一旦 Data Box 订单已处理，便无法再取消订单。 

### <a name="q-can-i-connect-a-data-box-at-the-same-to-multiple-host-computers-to-transfer-data"></a>问： 是否可以同时将 Data Box 连接到多台主机来传输数据？
A. 是的。 可将 Data Box 连接到多台主机来传输数据，并且可以并行运行多个复制作业。 有关详细信息，请转到[教程：将数据复制到 Azure Data Box](data-box-deploy-copy-data.md)。

### <a name="q-can-i-connect-to-both-the-10-gbe-interfaces-on-the-data-box-to-transfer-data"></a>问： 是否可以连接到 Data Box 上的两个 10-GbE 接口来传输数据？
A. 是的。 可以同时连接到 Data Box 上的两个 10-GbE 接口来复制数据。 有关如何复制数据的详细信息，请参阅[教程：将数据复制到 Azure Data Box](data-box-deploy-copy-data.md)。

<!--### Q. The network interface on my Data Box is not working. What should I do? 
A. 

### Q. I could not set up Data Box using a Dynamic (DHCP) IP address. Why would this be?
A.

### Q. I could not set up Data Box using a Static IP address. Why would this be?
A.

### Q. I could not set up Data Box on a private network. Why would this be?
A.-->

### <a name="q-the-system-fault-indicator-led-on-the-front-operating-panel-is-on-what-should-i-do"></a>问： 前操作面板上的系统故障指示器 LED 灯亮起。 我该怎么办？
A. 如果系统故障指示器 LED 灯亮起，则表示系统不正常。 [联系 Microsoft 支持部门](data-box-disk-contact-microsoft-support.md)以了解后续步骤。

### <a name="q-i-cant-access-the-data-box-unlock-password-in-the-azure-portal-why-would-this-be"></a>问： 无法在 Azure 门户中访问 Data Box 解锁密码。 这是为什么？
A. 如果无法在 Azure 门户中访问解锁密码，请检查订阅和存储帐户的权限。 请确保具有资源组级别的参与者或所有者权限。 如果没有，则至少需要 Data Box 操作员角色权限才能查看访问凭据。

### <a name="q-is-port-channel-configuration-supported-on-data-box-how-about-mpio"></a>问： Data Box 上是否支持端口通道配置？ 是否支持 MPIO？
A. Data Box 上不支持端口通道配置、多路径 IO (MPIO) 配置或 vLAN 配置。

## <a name="track-status"></a>跟踪状态

### <a name="q-how-do-i-track-the-data-box-from-when-i-placed-the-order-to-shipping-the-device-back"></a>问： 在从下单到寄回设备的整个过程中，如何跟踪 Data Box 的订单状态？ 
A.  可以在 Azure 门户中跟踪 Data Box 的订单状态。 创建订单时，系统还会提示你提供通知电子邮件。 如果已提供，则在订单状态发生任何变化时，你都会收到电子邮件通知。 详细了解如何[配置通知电子邮件](data-box-portal-ui-admin.md#edit-notification-details)。

### <a name="q-how-do-i-return-the-device"></a>问： 如何退回设备？ 
A.  Microsoft 将在电子墨水显示屏上显示发货标签。 如果发货标签未显示在电子墨水显示屏上，请转至“概述”>“下载发货标签”。 下载并打印标签，将标签插入设备上的透明塑料标签中，然后将设备放在运输承运商位置。 

### <a name="q-i-received-an-email-notification-that-my-device-has-reached-the-azure-datacenter-how-do-i-find-out-if-the-data-upload-is-in-progress"></a>问： 我收到电子邮件通知，提示我的设备已到达 Azure 数据中心。 如何了解数据是否正在上传？
A. 可转到 Azure 门户中的 Data Box 订单，然后转到“概况”。 如果将数据上传到 Azure 已启动，将在右窗格中看到复制进度。 

## <a name="migrate-data"></a>迁移数据

### <a name="q-what-is-the-maximum-data-size-that-can-be-used-with-data-box"></a>问： 在 Data Box 中最多可以使用多少数据？  
A.  Data Box 的可用存储容量为 80 TB。 对于大小介于 40 TB - 80 TB 之间的数据，使用一台 Data Box 设备即可。 对于高达 500 TB 的大型数据大小，可以订购多台 Data Box 设备。 对于大小超过 500 TB 的数据，请注册 Data Box Heavy。  

### <a name="q-what-are-the-maximum-block-blob-and-page-blob-sizes-supported-by-data-box"></a>问： Data Box 支持的最大块 Blob 和页 Blob 大小是什么？ 
A.  最大大小受制于 Azure 存储限制。 最大块 Blob 大致为 4.768 TiB，最大页 Blob 大小为 8 TiB。 有关详细信息，请转到 [Azure 存储可伸缩性和性能目标](../storage/common/storage-scalability-targets.md)。 

### <a name="q-how-do-i-know-that-my-data-is-secure-during-transit"></a>问： 如何知道我的数据在传输过程中是否安全？ 
A. 已实现多个安全功能，可在数据传输期间保护 Data Box。 其中包括防篡改封条、硬件和软件篡改检测和设备解锁密码。 有关详细信息，请参阅 [Azure Data Box 安全性和数据保护](data-box-security.md)。

### <a name="q-how-do-i-copy-the-data-to-the-data-box"></a>问： 如何将数据复制到 Data Box？ 
A.  如果使用 SMB 客户端，可以使用 Robocopy、Diskboss 等 SMB 复制工具，甚至是使用 Windows 文件资源管理器的拖放式操作来将数据复制到设备。 

如果使用 NFS 客户端，可以使用 [rsync](https://rsync.samba.org/)、[FreeFileSync](https://www.freefilesync.org/)、[Unison](https://www.cis.upenn.edu/~bcpierce/unison/) 或 [Ultracopier](https://ultracopier.first-world.info/)。 

有关详细信息，请转到[教程：将数据复制到 Azure Data Box](data-box-deploy-copy-data.md)。

### <a name="q-are-there-any-tips-to-speed-up-the-data-copy"></a>问： 在加快数据复制方面你们是否有诀窍？
A.  若要加快复制过程：

- 使用多个数据复制流。 例如，在 Robocopy 中使用多线程选项。 有关所用的确切命令的详细信息，请转到[教程：将数据复制到 Azure Data Box 并进行验证](data-box-deploy-copy-data.md)。
- 使用多个会话。
- 不通过网络共享进行复制（否则可能受到网络速度的限制）可确保在 Data Box 连接到的计算机本地驻留数据。
- 为用于复制数据的计算机建立性能基准。 下载并使用 [Bluestop FIO 工具](https://bluestop.org/fio/)来建立服务器硬件的性能基准。

<!--### Q. How to speed up the data copy if the source data has small files (KBs or few MBs)?
A.  To speed up the copy process:

- Create a local VHDx on fast storage or create an empty VHD on the HDD/SSD (slower).
- Mount it to a VM.
- Copy files to the VM’s disk.-->


### <a name="q-can-i-use-multiple-storage-accounts-with-data-box"></a>问： 是否可对 Data Box 使用多个存储帐户？
A.  是的。 Data Box 最多支持 10 个存储帐户（常规用途帐户、经典帐户或 blob 存储帐户）。 支持热 Blob 和冷 Blob。 在 GA 发布期，仅支持美国所有区域、西欧、北欧、法国和英国的 Azure 公有云中的存储帐户。


## <a name="ship-device"></a>运送设备

<!--### Q. How do I schedule a pickup for my Data Box?--> 

### <a name="q-my-device-was-delivered-but-the-device-seems-to-be-damaged-what-should-i-do"></a>问： 我的设备已送达，但似乎已损坏。 我该怎么办？
A. 如果设备到达时已损坏或存在被篡改的证据，请不要使用该设备。 [请联系 Microsoft 支持部门](data-box-disk-contact-microsoft-support.md)并尽快退回设备。 还可以新建针对更换设备的 Data Box 订单。 在这种情况下，更换设备不收费。

### <a name="q-can-i-use-my-own-shipping-carrier-to-ship-data-box"></a>问： 能否使用我自己的承运人运送 Data Box？
A. Microsoft 负责将 Data Box 服务寄回和寄出 Azure 数据中心。 如果想要使用你自己的承运人，可以使用 Azure 导入/导出服务。 有关详细信息，请转到[什么是 Azure 导入/导出服务？](../storage/common/storage-import-export-service.md)

### <a name="q-e-ink-display-is-not-showing-the-return-shipment-label-what-should-i-do"></a>问： 电子墨水显示屏未显示退货发货标签。 我该怎么办？
A. 如果电子墨水显示屏未显示退货发货标签，请执行以下步骤：
- 取下上一次发货留下的旧发货标签和任何便签。
- 在 Azure 门户中转到订单。 转到“概述”和“下载发货标签”。 有关详细信息，请转到[下载发货标签](data-box-portal-admin.md#download-shipping-label)。
- 打印发货标签并将其插入附加到设备的透明塑料封套中。 
- 请确保发货标签清晰可见。 

### <a name="q-how-is-my-data-protected-during-transit"></a>问： 在传输期间我的数据会受到怎样的保护？ 
A.  传输期间，Data Box 的以下功能可帮助保护数据。
 - Data Box 磁盘采用 AES 256 位加密进行加密。 
 - 设备将锁定，需要解锁密码才能进入和访问数据。
有关详细信息，请转到 [Data Box 的安全功能](data-box-security.md)。  

### <a name="q-i-have-finished-prepare-to-ship-and-shut-down-the-device-can-i-still-add-more-data-to-data-box"></a>问： 我已完成“准备交付”，并关闭了设备。 是否仍能向 Data Box 添加更多数据？
A. 是的。 可以打开设备并添加更多数据。 完成数据复制后，需要再次运行“准备交付”。

### <a name="q-i-received-my-device-and-it-is-not-booting-up-how-do-i-ship-the-device-back"></a>问： 我收到了我的设备，但它无法启动。 如何将设备寄回？
A. 如果设备无法启动，请转到 Azure 门户中的订单。 下载发货标签，将其贴在设备上。 有关详细信息，请转到[下载发货标签](data-box-portal-admin.md#download-shipping-label)。

## <a name="verify-and-upload"></a>验证和上传

### <a name="q-how-soon-can-i-access-my-data-in-azure-once-ive-shipped-the-data-box-back"></a>问： 寄回 Data Box 后，多久可以访问我在 Azure 中的数据？ 
A.  一旦“数据复制”的订单状态显示为“已完成”，应该立即就能访问数据。

### <a name="q-where-is-my-data-located-in-azure-after-the-upload"></a>问： 上传后，我的数据位于 Azure 中的哪个位置？
A.  将数据复制到 Data Box 中时，数据将上传到 Azure 存储帐户中的以下一个路径，具体取决于数据是块 Blob、页 Blob 还是 Azure 文件。
- `https://<storage_account_name>.blob.core.windows.net/<containername>` 
- `https://<storage_account_name>.file.core.windows.net/<sharename>`
 
  或者，可以转到 Azure 门户中的 Azure 存储帐户并从那里导航。

### <a name="q-i-just-noticed-that-i-did-not-follow-the-azure-naming-requirements-for-my-containers-will-my-data-fail-to-upload-to-azure"></a>问： 我刚刚发现，我并未遵循容器的 Azure 命名要求。 我的数据是否无法上传到 Azure？
A.  如果容器名称包含大写字母，这些名称会自动转换为小写。 如果名称存在其他方面的不合规情况（包含特殊字符、采用其他语言，等等），则上传将会失败。 有关命名共享、容器、文件的最佳做法的详细信息，请转到：
- [命名和引用共享](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [块 blob 和页 blob 约定](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)。

### <a name="q-how-do-i-verify-the-data-i-copied-onto-data-box"></a>问： 如何验证已复制到 Data Box 的数据？
A.  数据复制完成后，运行“准备交付”即可验证数据。 验证过程中，Data Box 将生成文件列表和数据的校验和。 可以下载文件列表，并根据源数据中的文件验证该列表。 有关详细信息，请转到[准备交付](data-box-deploy-picked-up.md#prepare-to-ship)。

### <a name="q-what-happens-to-my-data-after-i-have-returned-the-data-box"></a>问： 退回 Data Box 后，我的数据会发生什么情况？
A.  完成将数据复制到 Azure 的过程后，会根据 NIST SP 800-88 修订版 1 中的指导原则，安全擦除 Data Box 磁盘中的数据。 有关详细信息，请转到[从 Data Box 中擦除数据](data-box-deploy-picked-up.md#erasure-of-data-from-data-box)。

## <a name="audit-report"></a>审核报表

### <a name="how-does-azure-data-box-service-help-support-customers-chain-of-custody-procedure"></a>Azure Data Box 服务如何帮助支持客户的监管过程链？
A.  Azure Data Box 服务以本机方式提供可用于监管文件链的报表。 Azure 存储帐户中提供审核和复制日志。订单完成后，可以在 Azure 门户中[下载订单历史记录](data-box-portal-admin.md#download-order-history)。


### <a name="what-type-of-reporting-is-available-to-support-chain-of-custody"></a>哪种类型的报表可用于支持监管链？
A.  以下报表可用于支持监管链：

- UPS 的运输物流。
- 打开电源和用户共享访问的日志记录。
- 具有 64 位循环冗余检验 (CRC-64) 或成功引入到 Data Box 的每个文件的校验和的清单文件。
- 未能上传到 Azure 存储帐户的文件报告。
- 数据复制到 Azure 存储帐户后，依据 NIST 800 88R1 标准清理 Data Box 设备。

### <a name="are-the-carrier-tracking-logs-from-ups-available"></a>承运人跟踪日志（来自 UPS）是否可用？ 
A.  承运人跟踪日志在 Data Box 订单历史记录中捕获。 设备已返回 Azure 数据中心且其中的数据已清理后，此报告可用。 如立即需要，还可直接转到运营商的网站，使用订单跟踪号码获取跟踪信息。

### <a name="can-i-transport-the-data-box-to-azure-datacenter"></a>我能否自己将 Data Box 送到 Azure 数据中心？ 
A.  不是。 Azure 数据中心目前不接受客户或 UPS 以外的承运人交付 Data Box。


## <a name="next-steps"></a>后续步骤

- 请查看 [Data Box 系统要求](data-box-system-requirements.md)。
- 了解 [Data Box 限制](data-box-limits.md)。
- 在 Azure 门户中快速部署 [Azure Data Box](data-box-quickstart-portal.md)。
