---
title: 通过 Azure 门户管理 IoT Central | Microsoft Docs
description: 通过 Azure 门户管理 IoT Central。
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 01/14/2019
ms.topic: conceptual
manager: philmea
ms.openlocfilehash: c72de0ef874659a5d7840689e38bd7857c25b840
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65464119"
---
# <a name="manage-iot-central-from-the-azure-portal"></a>通过 Azure 门户管理 IoT Central

[!INCLUDE [iot-central-selector-manage](../../includes/iot-central-selector-manage.md)]

如果不从 IoT Central [应用程序管理器](https://aka.ms/iotcentral)页面创建和管理 IoT Central 应用程序，可以使用 [Azure 门户](https://portal.azure.com) 来管理应用程序。

## <a name="create-iot-central-applications"></a>创建 IoT Central 应用程序

若要创建的应用程序，导航到[Azure 门户](https://ms.portal.azure.com)，然后选择**创建资源**在左侧主导航菜单中。

![管理门户：导航菜单](media/howto-manage-iot-central-from-portal/image0.png)

在搜索栏中，键入“IoT Central”。

![管理门户：搜索](media/howto-manage-iot-central-from-portal/image0a1.png)

选择**IoT 中心的应用程序**在搜索结果中的行项。

![管理门户：搜索结果](media/howto-manage-iot-central-from-portal/image0b1.png)

现在选择“创建”。

![管理门户：IoT Central 资源](media/howto-manage-iot-central-from-portal/image0c1.png)

填写窗体中的所有字段。 此窗体与在 IoT Central [应用程序管理器](https://aka.ms/iotcentral)页创建应用程序时需要填写的窗体类似。 有关详细信息，请参阅[创建 IoT Central 应用程序](quick-deploy-iot-central.md)快速入门。

![管理门户：创建 IoT Central 资源](media/howto-manage-iot-central-from-portal/image1a.png)  

填写所有字段，然后选择**创建**。

## <a name="manage-existing-iot-central-applications"></a>管理现有 IoT Central 应用程序

如果已有 Azure IoT Central 应用程序，可将其删除或移动到 Azure 门户中的其他订阅或资源组。

> [!NOTE]
> Azure 门户中不会显示试用版应用程序，因为它们与订阅无关。

若要开始，选择**的所有资源**在左侧主导航菜单中。 在搜索框中键入应用程序的名称以在资源列表中将其找到。 然后选择你想要管理的 IoT Central 应用程序。

![管理门户：资源管理](media/howto-manage-iot-central-from-portal/image2a.png)

若要导航到应用程序，请选择 IoT Central 应用程序 URL。

![管理门户：资源管理](media/howto-manage-iot-central-from-portal/image3.png)

若要移动到不同的资源组应用程序，选择**更改**旁边的资源组。 在“移动资源”页上，选取要将此应用程序迁移到其中的资源组。

![管理门户：资源管理](media/howto-manage-iot-central-from-portal/image4a.png)

若要移动到其他订阅的应用程序，选择**更改**订阅旁边的链接。 在显示的对话框中，选取要将此应用程序迁移到其中的订阅。

![管理门户：资源管理](media/howto-manage-iot-central-from-portal/image5a.png)

## <a name="next-steps"></a>后续步骤

了解如何通过 Azure 门户管理 Azure IoT Central 应用程序后，建议接下来执行以下步骤：

> [!div class="nextstepaction"]
> [管理应用程序](howto-administer.md)