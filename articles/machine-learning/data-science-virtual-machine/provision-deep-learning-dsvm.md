---
title: 创建深度学习 Data Science Virtual Machine
titleSuffix: Azure
description: 在 Azure 上配置和创建深度学习数据科学虚拟机，用于进行分析和机器学习。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.custom: seodec18
ms.assetid: e1467c0f-497b-48f7-96a0-7f806a7bec0b
ms.service: machine-learning
ms.subservice: data-science-vm
ms.workload: data-services
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: 318df03c7c4447d051dfa396098462c0f8bbf423
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65410441"
---
# <a name="provision-a-deep-learning-virtual-machine-on-azure"></a>在 Azure 上预配深度学习虚拟机 

深度学习虚拟机 (DLVM) 是常用的[数据科学虚拟机](https://aka.ms/dsvm) (DSVM) 的特别配置变体，让你更轻松地使用基于 GPU 的 VM 实例快速训练深度学习模型。 Windows 2016 或作为基础的 Ubuntu DSVM 均支持它。 DLVM 共享 DSVM 上提供的相同核心 VM 映像，因此也共享了 DSVM 上所有丰富的工具集。 

DLVM 包含 AI 的多个工具，包括 GPU 版本的常用深度学习框架（如 Microsoft 认知工具包、TensorFlow、Keras、Caffe2、Chainer）、用于获取和预处理映像、文本数据的工具、用于数据科学建模和开发活动的工具（例如 Microsoft R Server Developer Edition、Anaconda Python、用于 Python 和 R 的 Jupyter Notebook、用于 Python 和 R 的 IDE、SQL 数据库以及许多其他数据科学和 ML 工具）。 

## <a name="create-your-deep-learning-virtual-machine"></a>创建深度学习虚拟机
以下是创建深度学习虚拟机实例的步骤： 

1. 在 [Azure 门户](https://portal.azure.com/#create/microsoft-ads.dsvm-deep-learningtoolkit
)中导航到虚拟机列表。
2. 选择底部的“创建”按钮进入向导。![create-dlvm](./media/dlvm-provision-wizard.PNG)
3. 用于创建 DLVM 的向导要求针对上图右侧列举的每个步骤（共有**四个步骤**）提供**输入**。 以下是配置每个步骤所需的输入：

   <a name="basics"></a>   
   1. **基础知识**
      
      1. **名称**：要创建的数据科学服务器的名称。
      2. **选择深度学习 VM 的 OS 类型**：选择 Windows 或 Linux（适用于基于 Windows 2016 和 Ubuntu Linux 的 DSVM）
      2. **用户名**：管理员帐户登录 ID。
      3. **密码**：管理员帐户密码。
      4. **订阅**：如果有多个订阅，请选择要在其上创建虚拟机并对其计费的订阅。
      5. **资源组**：可以创建一个新资源组，也可以使用订阅中现有的空 Azure 资源组。
      6. **位置**：选择最合适的数据中心。 通常，最合适的数据中心应拥有大部分数据，或者最接近实际位置以实现最快的网络访问。 
      
      > [!NOTE]
      > DLVM 支持所有 NC 和 ND 系列 GPU VM 实例。 预配 DLVM 时，必须在 Azure 中选择一个具有 GPU 的位置。 在 [Azure 产品（按区域）](https://azure.microsoft.com/regions/services/)页中查看是否有可用位置，并在“计算”下查找 **NC 系列**、**NCv2 系列**、**NCv3 系列**或 **ND 系列**。 

   1. **设置**：选择 NC 系列（NC、NCv2、NCv3）或 ND 系列中满足功能要求和成本约束的 GPU 虚拟机大小之一。 为 VM 创建存储帐户。  ![dlvm 设置](./media/dlvm-provision-step-2.PNG)
   
   1. **汇总**：验证输入的所有信息是否正确。

   1. **购买**：单击“购买”开始预配。 此时会显示交易条款的链接。 除计算**大小**步骤中选择的服务器大小所产生的费用外，VM 没有任何其他费用。 

> [!NOTE]
> 预配约需要 10 到 20 分钟。 预配的状态在 Azure 门户上显示。
> 


## <a name="how-to-access-the-deep-learning-virtual-machine"></a>如何访问深度学习虚拟机

### <a name="windows-edition"></a>Windows 版
创建 VM 后，可以使用在前面的**基本信息**部分中配置的管理员帐户凭据从远程桌面登录 VM。 

### <a name="linux-edition"></a>Linux 版

创建 VM 后，可使用 SSH 登录。 使用你在中创建的帐户凭据[**基础知识**](#basics)部分步骤 3 为文本 shell 接口。 连接到 Azure Vm 的 SSH 连接有关的详细信息，请参阅[安装和配置远程桌面以连接到 Azure 中的 Linux VM](/azure/virtual-machines/linux/use-remote-desktop)。 在 Windows 客户端，您可以下载之类的 SSH 客户端工具[Putty](https://www.putty.org)。 如果喜欢图形桌面（X Windows系统），可以在 Putty 上使用 X11 转发或安装 X2Go 客户端。 

> [!NOTE]
> 在我们的测试中，X2Go 客户端的性能优于 X11 转发。 建议对图形桌面界面使用 X2Go 客户端。
> 
> 

#### <a name="installing-and-configuring-x2go-client"></a>安装和配置 X2Go 客户端
Linux DLVM 已通过 X2Go 服务器进行预配并且可接受客户端连接。 若要连接到 Linux VM 图形桌面，请在客户端上完成以下过程：

1. 从 [X2Go ](https://wiki.x2go.org/doku.php/doc:installation:x2goclient) 为客户端平台下载并安装 X2Go 客户端。    
2. 运行 X2Go 客户端，并选择“新建会话”。 这会打开具有多个选项卡的配置窗口。 输入下列配置参数:
   * **会话选项卡**：
     * **主机**：Linux Data Science VM 的主机名或 IP 地址。
     * **登录名**：Linux VM 上的用户名。
     * **SSH 端口**：保留默认值 22。
     * **会话类型**：将值更改为“XFCE”。 Linux DSVM 目前仅支持 XFCE 桌面。
   * **媒体选项卡**：如果无需使用声音支持和客户端打印功能，可将其关闭。
   * **共享文件夹**：如果希望将目录从客户端计算机装入 Linux VM，则在此选项卡上添加要与 VM 共享的客户端计算机目录。

通过 X2Go 客户端使用 SSH 客户端或 XFCE 图形桌面登录 VM 后，即可开始使用 VM 上安装和配置的工具。 在 XFCE 上，可看到许多工具的应用程序菜单快捷方式和桌面图标。

创建并预配 VM 后，便可以开始使用其上安装并配置的工具。 许多工具带有开始菜单磁贴和桌面图标。 
