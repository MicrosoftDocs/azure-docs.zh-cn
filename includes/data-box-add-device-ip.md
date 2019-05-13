---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/07/2018
ms.author: alkohli
ms.openlocfilehash: e4b366075cb16f62a0e16b5b06da6fb19ffefdb9
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508301"
---
1. 登录到 Data Box 设备。 请确保该设备已解锁。

    ![Data Box 仪表板](media/data-box-add-device-ip/data-box-connect-via-rest-1.png)

2. 转到“设置网络接口”。 记下用来连接到客户端的网络接口的设备 IP 地址。

    ![Data Box 仪表板](media/data-box-add-device-ip/data-box-connect-via-rest-2.png)

3. 转到“连接和复制”，并单击“Rest”。

    ![Data Box 仪表板](media/data-box-add-device-ip/data-box-connect-via-rest-3.png)

4. 从“访问存储帐户和上传数据”对话框中，复制 **Blob 服务终结点**。

    ![Data Box 仪表板](media/data-box-add-device-ip/data-box-connect-via-rest-4.png)

5. 以管理员身份启动**记事本**，并打开位于 `C:\Windows\System32\Drivers\etc` 的 **hosts** 文件。
6. 将以下条目添加到 **hosts** 文件中：`<device IP address> <Blob service endpoint>`
7. 请使用以下图像作为参考。 保存 **hosts** 文件。

    ![Data Box 仪表板](media/data-box-add-device-ip/data-box-connect-via-rest-5.png)
