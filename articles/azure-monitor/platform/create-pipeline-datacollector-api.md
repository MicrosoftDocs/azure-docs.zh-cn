---
title: 使用 Azure 监视器数据收集器 API 创建数据管道 |Microsoft Docs
description: 可以使用 Azure Monitor HTTP 数据收集器 API，从能够调用 REST API 的任何客户端将 POST JSON 数据添加到 Log Analytics 工作区。 本文介绍如何以自动方式上传存储在文件中的数据。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/09/2018
ms.author: magoedte
ms.openlocfilehash: 961ce4427c509142077c95a6569ad2c444ec0a52
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205949"
---
# <a name="create-a-data-pipeline-with-the-data-collector-api"></a>使用数据收集器 API 创建数据管道

使用 [Azure Monitor 数据收集器 API](data-collector-api.md)，可以将任何自定义日志数据导入到 Azure Monitor 中的 Log Analytics 工作区。 唯一要求是数据必须采用 JSON 格式并拆分成 30 MB 或更小的段。 这是一种十分灵活的机制，可以通过多种方式插入：既可通过从应用程序直接发送的数据插入，也可通过一次性即席上传插入。 本文将概述一种常见方案的某些前期要求：需定期自动上传存储在文件中的数据。 虽然此处介绍的管道不会是最有效的（或者从另一个角度来说不会是最优化的），但你可以以它为起点，在将来构建你自己的生产管道。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="example-problem"></a>示例问题
本文余下部分，我们会在 Application Insights 中检查页面视图数据。 在我们的假想方案中，我们需要将默认情况下由 Application Insights SDK 收集的地理信息与包含全球每个国家/地区人口的自定义数据关联起来，目的是确定最值得我们投入市场营销费用的地方。 

为此，我们使用公开的数据源，例如 [UN World Population Prospects](https://esa.un.org/unpd/wpp/)（联合国世界人口展望）。 数据将采用以下简单架构：

![简单架构示例](./media/create-pipeline-datacollector-api/example-simple-schema-01.png)

在示例中，假定我们要上传的新文件包含最近一年的数据（前提是已发布）。

## <a name="general-design"></a>常规设计
我们将使用经典的 ETL 类型逻辑来设计管道。 体系结构将如下所示：

![数据集合管道体系结构](./media/create-pipeline-datacollector-api/data-pipeline-dataflow-architecture.png)

本文不会介绍如何创建数据或[将其上传到 Azure Blob 存储帐户](../../storage/blobs/storage-upload-process-images.md)， 但会介绍如何在在新文件上传到 Blob 时立刻拾取流。 在这里：

1. 会通过一个进程来检测是否有新数据上传。  我们的示例使用 [Azure 逻辑应用](../../logic-apps/logic-apps-overview.md)，其提供的触发器可以检测是否有新数据上传到 Blob。

2. 一个处理器将读取此新数据并将其转换为 JSON，这是 Azure Monitor 所需的格式。在此示例中，我们使用 [Azure Function](../../azure-functions/functions-overview.md) 作为一种轻型且经济高效的方式来执行处理代码。 启动此函数的逻辑应用是我们用来检测新数据的。

3. 最后，一旦 JSON 对象可用，就会将它发送到 Azure Monitor。 同一逻辑应用使用内置的 Log Analytics 数据收集器活动将数据发送到 Azure Monitor。

虽然我们未在本文中概述 Blob 存储、逻辑应用或 Azure Function 的详细设置，但在具体产品的页面上提供了详细说明。

为了监视此管道，我们使用 Application Insights 来监视 Azure Function（[详见此文](../../azure-functions/functions-monitoring.md)），使用 Azure Monitor 来监视逻辑应用（[详见此文](../../logic-apps/logic-apps-monitor-your-logic-apps-oms.md)）。 

## <a name="setting-up-the-pipeline"></a>设置管道
若要设置管道，请先确保创建并配置 Blob 容器。 同样，请确保创建 Log Analytics 工作区，以便将数据发送到其中。

## <a name="ingesting-json-data"></a>引入 JSON 数据
使用逻辑应用引入 JSON 数据是很简单的操作。由于不需进行转换，因此可以将整个管道包装在单个逻辑应用中。 配置 Blob 容器和 Log Analytics 工作区以后，请创建新的逻辑应用并对其进行配置，如下所示：

![逻辑应用工作流示例](./media/create-pipeline-datacollector-api/logic-apps-workflow-example-01.png)

保存逻辑应用，然后对其进行测试。

## <a name="ingesting-xml-csv-or-other-formats-of-data"></a>引入 XML、CSV 或其他格式的数据
现在的逻辑应用没有内置的功能可以将 XML、CSV 或其他类型轻松转换为 JSON 格式。 因此，需使用其他方式来完成该转换。 就本文来说，我们使用 Azure Functions 的无服务器计算功能来这样做，十分轻便且经济高效。 

在此示例中，我们分析 CSV 文件，但任何其他的文件类型也可以进行类似的处理。 请直接修改 Azure Function 的反序列化部分，使之反映特定数据类型的正确逻辑。

1.  在系统提示时使用 Function 运行时 v1 根据使用情况创建新的 Azure Function。  选择以 C# 为目标对象的“HTTP 触发器”模板作为起点，根据我们的要求配置绑定。 
2.  在右窗格的“查看文件”选项卡中，创建名为 **project.json** 的新文件，然后粘贴以下代码（来自我们正使用的 NuGet 包）：

    ![Azure Functions 示例项目](./media/create-pipeline-datacollector-api/functions-example-project-01.png)
    
    ``` JSON
    {
      "frameworks": {
        "net46":{
          "dependencies": {
            "CsvHelper": "7.1.1",
            "Newtonsoft.Json": "11.0.2"
          }  
        }  
       }  
     }  
    ```

3. 在右窗格中切换到 **run.csx**，将默认代码替换为以下代码： 

    >[!NOTE]
    >就你的项目来说，必须将记录模型（“PopulationRecord”类）替换为你自己的数据架构。
    >

    ```   
    using System.Net;
    using Newtonsoft.Json;
    using CsvHelper;
    
    class PopulationRecord
    {
        public String Location { get; set; }
        public int Time { get; set; }
        public long Population { get; set; }
    }

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        string filePath = await req.Content.ReadAsStringAsync(); //get the CSV URI being passed from Logic App
        string response = "";

        //get a stream from blob
        WebClient wc = new WebClient();
        Stream s = wc.OpenRead(filePath);         

        //read the stream
        using (var sr = new StreamReader(s))
        {
            var csvReader = new CsvReader(sr);
    
            var records = csvReader.GetRecords<PopulationRecord>(); //deserialize the CSV stream as an IEnumerable
    
            response = JsonConvert.SerializeObject(records); //serialize the IEnumerable back into JSON
        }    

        return response == null
            ? req.CreateResponse(HttpStatusCode.BadRequest, "There was an issue getting data")
            : req.CreateResponse(HttpStatusCode.OK, response);
     }  
    ```

4. 保存函数。
5. 测试函数，确保代码正常运行。 切换到右窗格中的“测试”选项卡，对测试进行配置，如下所示。 将 Blob 链接和示例数据置于“请求正文”文本框中。 单击“运行”以后，会在“输出”框中看到 JSON 输出：

    ![Function App 测试代码](./media/create-pipeline-datacollector-api/functions-test-01.png)

现在需返回去修改此前已开始生成的逻辑应用，使之包括已引入并转换为 JSON 格式的数据。  使用视图设计器进行如下所示的配置，然后保存逻辑应用：

![逻辑应用工作流完整示例](./media/create-pipeline-datacollector-api/logic-apps-workflow-example-02.png)

## <a name="testing-the-pipeline"></a>测试管道
现在可以将一个新文件上传到此前配置的 Blob，让逻辑应用对其进行监视。 很快就会看到逻辑应用的新实例启动并调用 Azure Function，然后成功地将数据发送到 Azure Monitor。 

>[!NOTE]
>首次发送新数据类型时，数据显示在 Azure Monitor 中可能需要长达 30 分钟的时间。


## <a name="correlating-with-other-data-in-log-analytics-and-application-insights"></a>与 Log Analytics 和 Application Insights 中的其他数据相关联
若要完成将 Application Insights 页面视图数据与我们从自定义数据源引入的人口数据相关联的目标，请在 Application Insights Analytics 窗口或 Log Analytics 工作区中运行以下查询：

``` KQL
app("fabrikamprod").pageViews
| summarize numUsers = count() by client_CountryOrRegion
| join kind=leftouter (
   workspace("customdatademo").Population_CL
) on $left.client_CountryOrRegion == $right.Location_s
| project client_CountryOrRegion, numUsers, Population_d
```

输出会显示两个数据源现在已联接。  

![关联搜索结果示例中的非联接数据](./media/create-pipeline-datacollector-api/correlating-disjoined-data-example-01.png)

## <a name="suggested-improvements-for-a-production-pipeline"></a>针对生产管道建议的改进
本文介绍了一个工作原型，其背后的逻辑可以应用于实际的生产级解决方案。 对于这样的生产级解决方案，建议进行以下改进：

* 在逻辑应用和 Function 中添加错误处理和重试逻辑。
* 添加逻辑，确保不超出 30MB/单次 Log Analytics 引入 API 调用限制。 根据需要将数据拆分成较小的段。
* 设置针对 Blob 存储的清理策略。 将数据成功发送到 Log Analytics 工作区以后，除非需要保留原始数据进行存档，否则没有理由继续存储这些数据。 
* 验证是否已在整个管道中启用监视功能，根据需要添加跟踪点和警报。
* 利用源代码管理对函数和逻辑应用的代码进行管理。
* 确保遵循适当的更改管理策略，在架构更改时对函数和逻辑应用进行相应的修改。
* 若要上传多个不同的数据类型，请将其隔离到 Blob 容器的单个文件夹中并创建逻辑，以便根据数据类型来扇出逻辑。 


## <a name="next-steps"></a>后续步骤
详细了解如何使用[数据收集器 API](data-collector-api.md) 将数据从任何 REST API 客户端写入到 Log Analytics 工作区。
