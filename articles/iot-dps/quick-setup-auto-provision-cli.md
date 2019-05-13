---
title: 使用 Azure CLI 设置设备预配服务 | Microsoft Docs
description: Azure 快速入门 - 使用 Azure CLI 设置 Azure IoT 中心设备预配服务
author: wesmc7777
ms.author: wesmc
ms.date: 02/26/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 3062fb640985498ba35e23f6310828a2bd59bfed
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60363710"
---
# <a name="set-up-the-iot-hub-device-provisioning-service-with-azure-cli"></a>使用 Azure CLI 设置 IoT 中心设备预配服务

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本快速入门详述了如何使用 Azure CLI 创建 IoT 中心和 IoT 中心设备预配服务并将两个服务链接到一起。 

如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

> [!IMPORTANT]
> 在本快速入门中创建的 IoT 中心和预配服务将会充当可以公开发现的 DNS 终结点。 如果决定更改用于这些资源的名称，请确保不使用任何敏感信息。
>



## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](/cli/azure/group#az-group-create) 命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

以下示例在 *westus* 位置创建名为 *my-sample-resource-group* 的资源组。

```azurecli-interactive 
az group create --name my-sample-resource-group --location westus
```

> [!TIP]
> 此示例在“美国西部”位置创建资源组。 可运行 `az account list-locations -o table` 命令，查看可用位置的列表。
>
>

## <a name="create-an-iot-hub"></a>创建 IoT 中心

使用 [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create) 命令创建 IoT 中心。

以下示例在 *westus* 位置创建名为 *my-sample-hub* 的 IoT 中心。  

```azurecli-interactive 
az iot hub create --name my-sample-hub --resource-group my-sample-resource-group --location westus
```

## <a name="create-a-provisioning-service"></a>创建预配服务

使用 [az iot dps create](/cli/azure/iot/dps#az-iot-dps-create) 命令创建预配服务。 

以下示例在 *westus* 位置创建名为 *my-sample-dps* 的预配服务。  

```azurecli-interactive 
az iot dps create --name my-sample-dps --resource-group my-sample-resource-group --location westus
```

> [!TIP]
> 此示例在“美国西部”位置创建预配服务。 若要查看可用位置的列表，可以运行 `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table` 命令，也可以转到[Azure 状态](https://azure.microsoft.com/status/)页，在其中搜索“设备预配服务”。 在命令中，可以使用一个单词或多个单词的格式来指定位置，例如：westus、West US、WEST US，等等。该值不区分大小写。 如果使用多个单词的格式来指定位置，请将值置于引号中，例如 `-- location "West US"`。
>


## <a name="get-the-connection-string-for-the-iot-hub"></a>获取 IoT 中心的连接字符串

需要提供 IoT 中心的连接字符串才能将其与设备预配服务链接到一起。 使用 [az iot hub show-connection-string](/cli/azure/iot/hub#az-iot-hub-show-connection-string) 命令获取连接字符串，并使用其输出设置一个变量。链接这两个资源时，需要用到该变量。 

以下示例将 *hubConnectionString* 变量设置为中心的 *iothubowner* 策略的主键连接字符串值。 可以使用 `--policy-name` 参数指定另一策略。 此命令使用 Azure CLI [查询](/cli/azure/query-azure-cli)和[输出](/cli/azure/format-output-azure-cli#tsv-output-format)选项从命令输出提取连接字符串。

```azurecli-interactive 
hubConnectionString=$(az iot hub show-connection-string --name my-sample-hub --key primary --query connectionString -o tsv)
```

可以使用 `echo` 命令查看连接字符串。

```azurecli-interactive 
echo $hubConnectionString
```

> [!NOTE]
> 这两个命令适用于在 Bash 下运行的主机。 如果使用本地 Windows/CMD shell 或 PowerShell 主机，则需修改这些命令，以便使用该环境的正确语法。
>

## <a name="link-the-iot-hub-and-the-provisioning-service"></a>将 IoT 中心和预配服务相链接

使用 [az iot dps linked-hub create](/cli/azure/iot/dps/linked-hub#az-iot-dps-linked-hub-create) 命令将 IoT 中心与预配服务相链接。 

以下示例将 *westus* 位置中名为 *my-sample-hub* 的 IoT 中心与名为 *my-sample-dps* 的设备预配服务相链接。 它使用在上一步的 *hubConnectionString* 变量中存储的 *my-sample-hub* 的连接字符串。

```azurecli-interactive 
az iot dps linked-hub create --dps-name my-sample-dps --resource-group my-sample-resource-group --connection-string $hubConnectionString --location westus
```

## <a name="verify-the-provisioning-service"></a>验证预配服务

使用 [az iot dps show](/cli/azure/iot/dps#az-iot-dps-show) 命令获取预配服务的详细信息。

以下示例获取名为 *my-sample-dps* 的预配服务的详细信息。 链接的 IoT 中心显示在 *properties.iotHubs* 集合中。

```azurecli-interactive
az iot dps show --name my-sample-dps
```

## <a name="clean-up-resources"></a>清理资源

本教程系列中的其他快速入门教程是在本文的基础上制作的。 如果打算继续学习后续的快速入门或相关教程，请不要清理在本快速入门中创建的资源。 如果不打算继续，可以使用以下命令删除预配服务、IoT 中心或资源组及其所有资源。

若要删除预配服务，请运行 [az iot dps delete](/cli/azure/iot/dps#az-iot-dps-delete) 命令：

```azurecli-interactive
az iot dps delete --name my-sample-dps --resource-group my-sample-resource-group
```
若要删除 IoT 中心，请运行 [az iot hub delete](/cli/azure/iot/hub#az-iot-hub-delete) 命令：

```azurecli-interactive
az iot hub delete --name my-sample-hub --resource-group my-sample-resource-group
```

若要删除资源组及其所有资源，请运行 [az group delete](/cli/azure/group#az-group-delete) 命令：

```azurecli-interactive
az group delete --name my-sample-resource-group
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已部署 IoT 中心和设备预配服务实例，并链接了这两个资源。 若要了解如何使用此设置来预配模拟设备，请继续学习本快速入门中关于如何创建模拟设备的内容。

> [!div class="nextstepaction"]
> [创建模拟设备快速入门](./quick-create-simulated-device.md)

