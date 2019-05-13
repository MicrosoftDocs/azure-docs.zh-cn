---
title: Azure 数据工厂中的集成运行时 | Microsoft Docs
description: 了解有关 Azure 数据工厂中的集成运行时的信息。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: abnarain
ms.openlocfilehash: 6a7daae90254bb4192dbaf13e1c2f9202e2d2baa
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232429"
---
# <a name="integration-runtime-in-azure-data-factory"></a>Azure 数据工厂中的集成运行时
集成运行时 (IR) 是 Azure 数据工厂用于在不同的网络环境之间提供以下数据集成功能的计算基础结构：

- **数据流**:执行[Data Flow](concepts-data-flow-overview.md)托管的 Azure 计算环境中。  
- **数据移动**：跨公用网络中的数据存储和专用网络（本地或虚拟专用网络）中的数据存储复制数据。 它提供对内置连接器、格式转换、列映射以及性能和可扩展数据传输的支持。
- **活动分派**：调度和监视在各种 Azure Databricks、 Azure HDInsight、 Azure 机器学习、 Azure SQL 数据库、 SQL Server 和的详细信息等计算服务上运行的转换活动。
- **SSIS 包执行**：在托管的 Azure 计算环境中本机执行 SQL Server 集成服务 (SSIS) 包。

在数据工厂中，活动定义要执行的操作。 链接服务定义目标数据存储或计算服务。 集成运行时提供活动和链接服务之间的桥梁。  它引用的链接的服务或活动，并提供计算环境的活动运行或分派。 这样一来，可以在最接近目标数据存储的区域中执行活动，或者，以最优性能计算服务的同时满足安全和合规性需求。

## <a name="integration-runtime-types"></a>集成运行时类型
数据工厂提供三种类型的集成运行时，应选择最能满足你所寻求的数据集成功能和网络环境需求的类型。  这三种类型是：

- Azure
- 自承载
- Azure-SSIS

下表介绍了针对其中每个集成运行时类型提供的功能和网络支持：

IR 类型 | 公用网络 | 专用网络
------- | -------------- | ---------------
Azure | 数据流<br/>数据移动<br/>活动分派 | &nbsp;
自承载 | 数据移动<br/>活动分派 | 数据移动<br/>活动分派
Azure-SSIS | SSIS 包执行 | SSIS 包执行

下图介绍了如何结合使用不同的集成运行时以提供丰富的数据集成功能和网络支持：

![不同类型的集成运行时](media/concepts-integration-runtime/different-integration-runtimes.png)

## <a name="azure-integration-runtime"></a>Azure 集成运行时
Azure 集成运行时能够：

- 在 Azure 中运行的数据流 
- 在云数据存储之间运行复制活动
- 在公用网络中分派以下转换活动：Databricks Notebook / Jar / Python 活动、 HDInsight Hive 活动、 HDInsight Pig 活动、 HDInsight MapReduce 活动、 HDInsight Spark 活动、 HDInsight Streaming 活动、 机器学习批处理执行活动，机器学习更新资源活动、 存储过程活动、 Data Lake Analytics U-SQL 活动、.NET 自定义活动、 Web 活动、 Lookup 活动和 Get Metadata 活动。

### <a name="azure-ir-network-environment"></a>Azure IR 网络环境
Azure 集成运行时支持连接到数据存储和计算服务的公共可访问的终结点。 为 Azure 虚拟网络环境使用自承载集成运行时。

### <a name="azure-ir-compute-resource-and-scaling"></a>Azure IR 计算资源和缩放
Azure 集成运行时在 Azure 中提供完全托管的无服务器计算。  无需担心基础结构配置、软件安装、修补或功能扩展。  此外，只需为实际使用时间付费。

Azure 集成运行时提供了使用安全、可靠和高性能的方式在云数据存储之间移动数据的本机计算。  可以设置在复制活动上要使用的数据集成单元的数量，相应地，Azure IR 的计算大小弹性地纵向扩展，无需显式调整 Azure 集成运行时的大小。 

活动分派是将活动路由到目标计算服务的轻型操作，因此，无需纵向扩展此方案的计算大小。

有关创建和配置 Azure IR 的信息，请参阅操作方法指南下的如何创建和配置 Azure IR。 

> [!NOTE] 
> Azure 集成运行时具有与数据流运行时，它定义将用于运行数据流的基础计算基础结构相关的属性。 

## <a name="self-hosted-integration-runtime"></a>自承载集成运行时
自承载 IR 能够：

- 在专用网络中的云数据存储和数据存储之间运行复制活动。
- 对本地或 Azure 虚拟网络中的计算资源分派以下转换活动：HDInsight Hive 活动 （BYOC 使自己的群集）、 HDInsight Pig 活动 (BYOC)、 HDInsight MapReduce 活动 (BYOC)、 HDInsight Spark 活动 (BYOC)、 HDInsight Streaming 活动 (BYOC)，机器学习批处理执行活动，机器学习更新资源活动、 存储过程活动、 Data Lake Analytics U-SQL 活动、.NET 自定义活动、 Lookup 活动和 Get Metadata 活动。

> [!NOTE] 
> 使用自承载集成运行时支持需要自带驱动程序（如 SAP Hana、MySQL 等）的数据存储。有关详细信息，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。

### <a name="self-hosted-ir-network-environment"></a>自承载 IR 网络环境
如果想要在专用网络环境中安全地执行数据集成（不在公有云环境中建立直通连接），可以在企业防火墙后的本地环境中或虚拟专用网络内安装自承载 IR。  自承载集成运行时仅进行基于出站 HTTP 的连接，以打开 Internet。

### <a name="self-hosted-ir-compute-resource-and-scaling"></a>自承载 IR 计算资源和缩放
需要在本地计算机或专用网络中的虚拟机上安装自承载 IR。 目前，仅支持在 Windows 操作系统上运行自承载 IR。  

为了获得高可用性和可伸缩性，可以通过在主动-主动模式中将逻辑实例与多个本地计算机相关联来向外扩展自承载 IR。  有关详细信息，请参阅操作方法指南下的“如何创建和配置自承载 IR”一文。

## <a name="azure-ssis-integration-runtime"></a>Azure-SSIS 集成运行时
若要提升和切换现有 SSIS 工作负荷，可以创建 Azure-SSIS IR 以本机执行 SSIS 包。

### <a name="azure-ssis-ir-network-environment"></a>Azure-SSIS IR 网络环境
可以在公用网络或专用网络中配置 Azure-SSIS IR。  通过将 Azure-SSIS IR 加入连接到本地网络的虚拟网络来支持本地数据访问。  

### <a name="azure-ssis-ir-compute-resource-and-scaling"></a>Azure-SSIS IR 计算资源和缩放
Azure-SSIS IR 是完全托管的 Azure VM 群集，专用于运行 SSIS 包。 可使用自己的 Azure SQL 数据库或托管实例服务器托管要附加到该服务器的 SSIS 项目/包 (SSISDB) 的目录。 可以通过指定节点大小纵向扩展计算能力并通过指定群集中的节点数对其进行横向扩展。 可以在认为合适时停止和启动 Azure-SSIS 集成运行时以管理运行的成本。

有关详细信息，请参阅操作方法指南下的“如何创建和配置 Azure-SSIS IR”一文。  创建后，即可使用熟悉的工具（如 SQL Server 数据工具 (SSDT) 和 SQL Server Management Studio (SSMS)）部署和管理现有 SSIS 包，无需对其更改或仅做少量更改。

有关 Azure-SSIS 运行时的详细信息，请参阅以下文章： 

- [教程：将 SSIS 包部署到 Azure](tutorial-create-azure-ssis-runtime-portal.md)。 此文提供有关创建 Azure-SSIS IR，并使用 Azure SQL 数据库来承载 SSIS 目录的分步说明。 
- [如何：创建 Azure-SSIS 集成运行时](create-azure-ssis-integration-runtime.md)。 本文是教程的拓展延伸，介绍了如何使用 Azure SQL 数据库托管实例以及如何将 IR 加入虚拟网络。 
- [监视 Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime)。 此文介绍如何检索有关 Azure-SSIS IR 的信息，以及返回的信息中的状态说明。 
- [管理 Azure-SSIS IR](manage-azure-ssis-integration-runtime.md)。 此文介绍如何停止、启动或删除 Azure-SSIS IR。 此外，介绍如何通过在 Azure-SSIS IR 中添加更多节点来扩展 IR。 
- [将 Azure-SSIS IR 加入虚拟网络](join-azure-ssis-integration-runtime-virtual-network.md)。 此文提供有关将 Azure-SSIS IR 加入 Azure 虚拟网络的概念性信息。 此外，还介绍可以执行哪些步骤来使用 Azure 门户配置虚拟网络，以便 Azure-SSIS IR 能够加入虚拟网络。 

## <a name="integration-runtime-location"></a>集成运行时位置
数据工厂位置是存储数据工厂元数据和启动管道触发所在的位置。 同时，数据工厂可以访问其他 Azure 区域的数据存储和计算数据，在数据存储之间移动数据或使用计算服务处理数据。 此行为通过[全局可用 IR 来实现](https://azure.microsoft.com/global-infrastructure/services/)，以确保数据的符合性、有效性并减少网络对外费用。

IR 位置定义其后端计算的位置，尤其是执行数据移动、活动分派和 SSIS 包执行的位置。 IR 位置可能与数据工厂所属的位置不同。 

### <a name="azure-ir-location"></a>Azure IR 位置
可以设置 Azure IR 的特定位置，这样数据移动或活动调度就会发生在该特定区域。 

如果选择使用默认的自动解析 Azure IR，则会出现以下情况： 

- 对于复制活动，ADF 会尽量自动检测接收器和源数据存储，以便在可用的情况下选择同一区域的最佳位置，或者选择同一地理位置的最近位置，或者在不可检测的情况下使用数据工厂区域作为替代。

- 若要执行查找/GetMetadata 活动和调度转换活动，ADF 会使用数据工厂区域中的 IR。

- 数据流中，对于 ADF 将数据工厂区域中使用红外线 （ir）。 

  > [!TIP] 
  > 较好的做法是确保数据流 （如果可能） 运行在与相应的数据存储位于同一区域中。 您可以实现此目的通过自动解决 Azure IR （如果数据存储区位置是与数据工厂的位置相同），或通过在与您的数据存储位于同一区域中创建新的 Azure IR 实例，然后在其上执行数据流。 

可以在 UI 或活动监视有效负载的管道活动监视视图中监视哪个 IR 位置在活动执行期间生效。

>[!TIP]
>如果有严格的数据符合性要求，并需确保数据不离开特定的地域，则可在特定区域显式创建一个 Azure IR，然后使用 ConnectVia 属性将链接服务指向该 IR。 例如，若需将数据从英国南部的 Blob 复制到英国南部的 SQL DW，并且需确保数据不离开英国，请在英国南部创建一个 Azure IR，然后将两个链接服务均链接到该 IR。

### <a name="self-hosted-ir-location"></a>自承载 IR 位置
自承载 IR 逻辑上注册到数据工厂，用于支持其功能的计算由你提供。 因此，没有适用于自承载 IR 的显式位置属性。 

用于执行数据移动时，自承载 IR 从源提取数据并写入到目标。

### <a name="azure-ssis-ir-location"></a>Azure-SSIS IR 位置
为你的 Azure-SSIS IR 选择正确的位置对在提取-转换-加载 (ETL) 工作流中实现高性能至关重要。

- Azure-SSIS IR 的位置无需与数据工厂的位置相同，但应与你自己的需要托管 SSISDB 的 Azure SQL 数据库/托管实例服务器的位置相同。 这样一来，Azure-SSIS 集成运行时可以轻松地访问 SSISDB，且不会在不同位置之间产生过多的流量。
- 如果没有托管 SSISDB 的现有 Azure SQL 数据库/托管实例服务器，但有本地数据源/目标，则应在连接到本地网络的虚拟网络所在位置中创建新的 Azure SQL 数据库/托管实例服务器。  这样，即可使用新的 Azure SQL 数据库/托管实例服务器并加入该虚拟网络来创建 Azure-SSIS IR，从而将不同位置间的数据移动降至最低 - 所有操作在同一位置进行。
- 如果托管 SSISDB 的现有 Azure SQL 数据库/托管实例服务器的位置与连接到本地网络的虚拟网络的位置不同，请先使用现有 Azure SQL 数据库/托管实例服务器并在同一位置加入其他虚拟网络来创建 Azure-SSIS IR，然后配置不同位置之间的虚拟网络到虚拟网络连接。

下图显示了数据工厂及其集成运行时的位置设置：

![集成运行时位置](media/concepts-integration-runtime/integration-runtime-location.png)

## <a name="determining-which-ir-to-use"></a>确定要使用哪个 IR

### <a name="copy-activity"></a>复制活动

对于复制活动，它需要使用源和接收器链接服务，以定义数据流的方向。 以下逻辑用于确定执行复制所使用的集成运行时实例的类型： 

- **在两个云数据源之间复制**：当源和接收器链接服务都使用 Azure IR 时，ADF 会使用区域性的 Azure IR（如果已指定），或者自动确定 Azure IR 的位置，前提是你根据[集成运行时位置](#integration-runtime-location)部分的说明选择自动解析 IR（默认设置）。
- **在云数据源和专用网络中的数据源之间复制**：如果源或接收器链接服务指向自承载 IR，则在该自承载集成运行时上执行复制活动。
- **在专用网络中的两个数据源之间复制**：源和接收器链接服务必须同时指向同一集成运行时实例，且该集成运行时用于执行复制活动。

### <a name="lookup-and-getmetadata-activity"></a>查找和 GetMetadata 活动

查找和 GetMetadata 活动在关联到数据存储链接服务的集成运行时上执行。

### <a name="transformation-activity"></a>转换活动

每个转换活动都有一个目标计算链接服务，该服务指向集成运行时。 该集成运行时实例是分派转换活动的实例。

### <a name="data-flow-activity"></a>数据流活动

与之关联的集成运行时中执行数据流活动。 

## <a name="next-steps"></a>后续步骤
请参阅以下文章：

- [创建 Azure 集成运行时](create-azure-integration-runtime.md)
- [创建自承载集成运行时](create-self-hosted-integration-runtime.md)
- [创建 Azure-SSIS 集成运行时](create-azure-ssis-integration-runtime.md)。 本文是教程的拓展延伸，介绍了如何使用 Azure SQL 数据库托管实例以及如何将 IR 加入虚拟网络。 
