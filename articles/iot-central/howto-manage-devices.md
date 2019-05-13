---
title: 在 Azure IoT Central 应用程序中管理设备 | Microsoft Docs
description: 如何以操作员的身份在 Azure IoT Central 应用程序中管理设备。
author: ellenfosborne
ms.author: elfarber
ms.date: 01/30/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: e4746620f083996bf64e77617ec472c3d3894d91
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464307"
---
# <a name="manage-devices-in-your-azure-iot-central-application"></a>在 Azure IoT Central 应用程序中管理设备

本文介绍如何以操作员的身份在 Azure IoT Central 应用程序中管理设备。 操作员可以：

- 使用“Device Explorer”页查看、添加和删除与 Azure IoT Central 应用程序相连接的设备。
- 维护最新的设备清单。
- 通过更改设备属性中存储的值，使设备元数据保持最新状态。
- 通过在“设置”页中更新特定设备的设置，来控制设备的行为。

## <a name="view-your-devices"></a>查看设备

若要查看单个设备：

1. 在左侧导航菜单中选择“Device Explorer”。 此处会显示[设备模板](howto-set-up-template.md)的列表。

1. 在“模板”列表中选择设备模板。

1. 在“Device Explorer”页面的右侧窗格中，可以看到根据该设备模板创建的设备的列表。 选择单个设备以查看该设备的“设备详细信息”页：

    ![设备详细信息页](./media/howto-manage-devices/devicelist.png)

## <a name="add-a-device"></a>添加设备

若要将设备添加到 Azure IoT Central 应用程序：

1. 在左侧导航菜单中选择“Device Explorer”。

1. 选择要从中创建设备的设备模板。

1. 选择“+ 新建”。

1. 选择“真实”或“模拟”。 真实设备是指要连接到 Azure IoT Central 应用程序的物理设备。 模拟设备包含 Azure IoT Central 生成的示例数据。

## <a name="import-devices"></a>导入设备

若要将大量设备连接到应用程序，可从 CSV 文件批量导入设备。 该 CSV 文件应包含以下列和标头：

* **IOTC_DeviceID** - 设备 ID 应全部小写。
* **IOTC_DeviceName** - 此列是可选的。

若要在应用程序中批量注册设备：

1. 在左侧导航菜单中选择“Device Explorer”。

1. 在左面板中，选择要为其批量创建设备的设备模板。

    > [!NOTE]
    > 如果还没有设备模板，则可以在“未关联的设备”下导入设备，并在没有模板的情况下注册这些设备。 导入设备后，可以将其与模板相关联。

1. 选择“导入”。

    ![导入操作](./media/howto-manage-devices/bulkimport1a.png)

1. 选择包含要导入的设备 ID 列表的 CSV 文件。

1. 上传文件后，会立即开始导入设备。 可以在设备网格的顶部跟踪导入状态。

1. 导入完成后，设备网格上会显示成功消息。

    ![导入成功](./media/howto-manage-devices/bulkimport3a.png)

如果设备导入操作失败，将在设备网格上看到错误消息。 系统会生成一个捕获所有错误的可下载的日志文件。

**将设备与模板关联**

如果通过在“未关联设备”下启动导入来注册设备，则无需任何设备模板关联即可创建设备。 设备必须与模板关联起来，才能浏览有关设备的数据和其他详细信息。 请按照以下步骤将设备与模板进行关联：

1. 在左侧导航菜单中选择“Device Explorer”。

1. 在左面板中，选择“未关联的设备”：

    ![未关联的设备](./media/howto-manage-devices/unassociateddevices1a.png)

1. 选择想要与模板关联的设备：

1. 选择**相关联**:

    ![关联设备](./media/howto-manage-devices/unassociateddevices2a.png)

1. 从可用模板列表中选择该模板，然后选择**相关联**。

1. 所选设备与所选设备模板相关联。

> [!NOTE]
> 设备与某个模板关联后，就无法取消关联或与其他模板关联。

## <a name="export-devices"></a>导出设备

若要将真实设备连接到 IoT Central，需要其连接字符串。 您可以导出大容量以获取你需要创建设备的连接字符串信息中的设备详细信息。 导出过程中使用设备标识、 设备名称和所有所选设备的密钥创建的 CSV 文件。

从应用程序中批量导出设备：

1. 在左侧导航菜单中选择“Device Explorer”。

1. 在左面板中，选择要从中导出设备的设备模板。

1. 选择你想要导出，然后选择的设备**导出**操作。

    ![导出](./media/howto-manage-devices/export1a.png)

1. 导出过程开始。 可以在网格顶部跟踪状态。

1. 导出完成后，将显示一条成功消息，并提供一个用来下载生成文件的链接。

1. 选择**条成功消息**若要将文件下载到磁盘上的本地文件夹。

    ![导出成功](./media/howto-manage-devices/export2a.png)

1. 导出的 CSV 文件包含以下列：设备 ID、设备名称、设备密钥和 X509 证书指纹：

    * IOTC_DEVICEID
    * IOTC_DEVICENAME
    * IOTC_SASKEY_PRIMARY
    * IOTC_SASKEY_SECONDARY
    * IOTC_X509THUMBPRINT_PRIMARY
    * IOTC_X509THUMBPRINT_SECONDARY

请参阅[在 Azure IoT 中心的设备连接性](concepts-connectivity.md)，有关连接字符串和连接到 IoT 中心应用程序的实际设备的详细信息。

## <a name="delete-a-device"></a>删除设备

若要从 Azure IoT Central 应用程序中删除真实设备或模拟设备：

1. 在导航菜单中选择“Device Explorer”。

1. 选择要删除的设备的设备模板。

1. 勾选要删除的设备旁边的框。

1. 选择“删除”。

## <a name="change-a-device-setting"></a>更改设备设置

设置控制设备的行为。 换而言之，设置可用于提供设备的输入。 可以在“设备详细信息”页上查看和更新设备设置。

1. 在导航菜单中选择“Device Explorer”。

1. 选择要更改其设置的设备的设备模板。

1. 选择“设置”选项卡。此处会显示设备的所有设置及其当前值。 对于每项设置，可以查看设备是否仍在同步。

1. 将设置修改为所需的值。 可以一次修改多个设置，并同时更新这些设置。

1. 选择“更新”。 值会发送到设备。 当设备确认设置更改时，该设置的状态将恢复为“已同步”。

## <a name="change-a-property"></a>更改属性

属性是与设备关联的设备元数据，例如城市和序列号。 可以在“设备详细信息”页上查看和更新属性。

1. 在导航菜单中选择“Device Explorer”。

1. 选择要更改其属性的设备的设备模板。

1. 选择“属性”选项卡，在其中可以查看所有属性。

1. 将应用程序属性修改为所需的值。 可以一次性修改多个属性，也可以同时更新所有属性。 选择“更新”。

> [!NOTE]
> 无法更改设备属性的值。 设备属性由设备设置，在 Azure IoT Central 应用程序中是只读的。

## <a name="next-steps"></a>后续步骤

了解如何在 Azure IoT Central 应用程序中管理设备后，建议接下来执行以下步骤：

> [!div class="nextstepaction"]
> [如何使用设备集](howto-use-device-sets.md)

<!-- Next how-tos in the sequence -->
