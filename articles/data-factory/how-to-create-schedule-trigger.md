---
title: 在 Azure 数据工厂中创建计划触发器 | Microsoft Docs
description: 了解如何在 Azure 数据工厂中创建按计划运行管道的触发器。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 01/23/2018
ms.author: shlo
ms.openlocfilehash: 09f80f69857ae17a0136229fe9bf13d4f63e7096
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65151078"
---
# <a name="create-a-trigger-that-runs-a-pipeline-on-a-schedule"></a>创建按计划运行管道的触发器
本文提供有关计划触发器和创建、启动和监视计划触发器的步骤的信息。 有关其他类型的触发器，请参阅[管道执行和触发器](concepts-pipeline-execution-triggers.md)。

创建计划触发器时，请指定该触发器的计划（开始日期、重复周期、结束日期等），并将其与管道相关联。 管道和触发器具有“多对多”关系。 多个触发器可以启动单个管道。 单个触发器可以启动多个管道。

以下部分提供以不同方式创建计划触发器的步骤。 

## <a name="data-factory-ui"></a>数据工厂 UI
可以创建**计划触发器**，用于将管道计划为定期运行（每小时运行一次、每天运行一次，等等）。 

> [!NOTE]
> 有关创建管道和计划触发器、将触发器与管道相关联，以及运行和监视管道的完整演练，请参阅[快速入门：使用数据工厂 UI 创建数据工厂](quickstart-create-data-factory-portal.md)。

1. 切换到“编辑”选项卡。 

    ![切换到“编辑”选项卡](./media/how-to-create-schedule-trigger/switch-edit-tab.png)
1. 单击菜单上的“触发器”，然后单击“新建/编辑”。 

    ![“新建触发器”菜单](./media/how-to-create-schedule-trigger/new-trigger-menu.png)
2. 在“添加触发器”页中单击“选择触发器...”，然后单击“新建”。 

    ![添加触发器 - 新建触发器](./media/how-to-create-schedule-trigger/add-trigger-new-button.png)
3. 在“新建触发器”页中执行以下步骤： 

    1. 确认在“类型”中选择了“计划”。 
    2. 在“开始日期(UTC)”中指定触发器的开始日期时间。 此值默认设置为当前日期时间。 
    3. 指定触发器的“重复周期”。 从下拉列表中选择一个值（“每分钟”、“每小时”、“每日”、“每周”和“每月”）。 在文本框中输入乘数。 例如，如果希望触发器每隔 15 分钟运行一次，请选择“每分钟”，并在文本框中输入 **15**。 
    4. 对于“结束”字段，如果不想要指定触发器的结束日期时间，请选择“不结束”。 若要指定结束日期时间，请选择“日期”并指定结束日期时间，然后单击“应用”。 每次管道运行都需要付出相关成本。 若要进行测试，需确保只触发管道几次。 但是，请确保在发布时间和结束时间之间有足够的时间来运行管道。 只有在将解决方案发布到数据工厂之后，触发器才会生效，而不是在 UI 中保存触发器就会使该触发器生效。

        ![触发器设置](./media/how-to-create-schedule-trigger/trigger-settings.png)
4. 在“新建触发器”窗口中，选中“已激活”选项，然后单击“下一步”。 以后，可以使用此复选框停用该触发器。 

    ![触发器设置 -“下一步”按钮](./media/how-to-create-schedule-trigger/trigger-settings-next.png)
5. 在“新建触发器”页中查看警告消息，然后单击“完成”。

    ![触发器设置 -“完成”按钮](./media/how-to-create-schedule-trigger/new-trigger-finish.png)
6. 单击“发布”，将所做的更改发布到数据工厂。 在将更改发布到数据工厂之前，触发器不会开始触发管道运行。 

    ![发布按钮](./media/how-to-create-schedule-trigger/publish-2.png)
8. 在左侧切换到“监视”选项卡。 单击“刷新”可刷新列表。 可以看到计划的触发器触发的管道运行。 请注意“触发因素”列中的值。 如果使用“立即触发”选项，将在列表中看到手动触发器运行。 

    ![监视触发的运行](./media/how-to-create-schedule-trigger/monitor-triggered-runs.png)
9. 单击“管道运行”旁边的向下箭头，切换到“触发器运行”视图。 

    ![监视触发器运行](./media/how-to-create-schedule-trigger/monitor-trigger-runs.png)

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

本部分介绍如何使用 Azure PowerShell 创建、启动和监视计划触发器。 若要查看此示例运行，请先查看[快速入门：使用 Azure PowerShell 创建数据工厂](quickstart-create-data-factory-powershell.md)。 然后，将以下代码添加到 main 方法，该代码用于创建并启动一个每 15 分钟运行一次的计划触发器。 该触发器与快速入门中创建的名为 **Adfv2QuickStartPipeline** 的管道相关联。

1. 在 C:\ADFv2QuickStartPSH\ 文件夹中创建一个名为 **MyTrigger.json** 的且包含以下内容的 JSON 文件：

    > [!IMPORTANT]
    > 保存 JSON 文件之前，请将 **startTime** 元素的值设置为当前 UTC 时间。 将 **endTime** 元素的值设置为比当前 UTC 时间早一小时。

    ```json   
    {
        "properties": {
            "name": "MyTrigger",
            "type": "ScheduleTrigger",
            "typeProperties": {
                "recurrence": {
                    "frequency": "Minute",
                    "interval": 15,
                    "startTime": "2017-12-08T00:00:00",
                    "endTime": "2017-12-08T01:00:00"
                }
            },
            "pipelines": [{
                    "pipelineReference": {
                        "type": "PipelineReference",
                        "referenceName": "Adfv2QuickStartPipeline"
                    },
                    "parameters": {
                        "inputPath": "adftutorial/input",
                        "outputPath": "adftutorial/output"
                    }
                }
            ]
        }
    }
    ```

    在 JSON 代码片段中：
    - 触发器的 **type** 元素设置为“ScheduleTrigger”。
    - **frequency** 元素设置为“Minute”，**interval** 元素设置为 15。 因此，触发器在开始时间与结束时间之间每 15 分钟运行一次管道。
    - **endTime** 元素比 **startTime** 元素的值晚一小时。 因此，触发器将在开始时间之后的 15 分钟、30 分钟和 45 分钟运行管道。 不要忘记将开始时间更新为当前 UTC 时间，将结束时间更新为比开始时间晚一小时。 
    - 触发器与 **Adfv2QuickStartPipeline** 管道相关联。 若要将多个管道与某个触发器相关联，请添加多个 **pipelineReference** 节。
    - 快速入门中的管道采用两个**参数**值：**inputPath** 和 **outputPath**。 因此，可以通过触发器传递这些参数的值。

2. 通过创建触发器**集 AzDataFactoryV2Trigger** cmdlet:

    ```powershell
    Set-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger" -DefinitionFile "C:\ADFv2QuickStartPSH\MyTrigger.json"
    ```

3. 确认触发器的状态是否**已停止**通过使用**Get AzDataFactoryV2Trigger** cmdlet:

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

4. 通过使用启动触发器**开始 AzDataFactoryV2Trigger** cmdlet:

    ```powershell
    Start-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

5. 确认触发器的状态是否**Started**通过使用**Get AzDataFactoryV2Trigger** cmdlet:

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

6.  获取触发器运行在 Azure PowerShell 中使用**Get AzDataFactoryV2TriggerRun** cmdlet。 若要获取有关触发器运行的信息，请定期执行以下命令。 更新 **TriggerRunStartedAfter** 和 **TriggerRunStartedBefore** 值以匹配触发器定义中的值：

    ```powershell
    Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-08T00:00:00" -TriggerRunStartedBefore "2017-12-08T01:00:00"
    ```
    
    若要在 Azure 门户中监视触发器运行和管道运行，请参阅[监视管道运行](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline)。


## <a name="net-sdk"></a>.NET SDK
本部分介绍如何使用 .NET SDK 创建、启动和监视触发器。 若要查看此示例运行，请先查看[快速入门：使用 .NET SDK 创建数据工厂](quickstart-create-data-factory-dot-net.md)。 然后，将以下代码添加到 main 方法，该代码用于创建并启动一个每 15 分钟运行一次的计划触发器。 该触发器与快速入门中创建的名为 **Adfv2QuickStartPipeline** 的管道相关联。

若要创建并启动一个每 15 分钟运行一次的计划触发器，请将以下代码添加到 main 方法：

```csharp
            // Create the trigger
            Console.WriteLine("Creating the trigger");

            // Set the start time to the current UTC time
            DateTime startTime = DateTime.UtcNow;

            // Specify values for the inputPath and outputPath parameters
            Dictionary<string, object> pipelineParameters = new Dictionary<string, object>();
            pipelineParameters.Add("inputPath", "adftutorial/input");
            pipelineParameters.Add("outputPath", "adftutorial/output");

            // Create a schedule trigger
            string triggerName = "MyTrigger";
            ScheduleTrigger myTrigger = new ScheduleTrigger()
            {
                Pipelines = new List<TriggerPipelineReference>()
                {
                    // Associate the Adfv2QuickStartPipeline pipeline with the trigger
                    new TriggerPipelineReference()
                    {
                        PipelineReference = new PipelineReference(pipelineName),
                        Parameters = pipelineParameters,
                    }
                },
                Recurrence = new ScheduleTriggerRecurrence()
                {
                    // Set the start time to the current UTC time and the end time to one hour after the start time
                    StartTime = startTime,
                    TimeZone = "UTC",
                    EndTime = startTime.AddHours(1),
                    Frequency = RecurrenceFrequency.Minute,
                    Interval = 15,
                }
            };

            // Now, create the trigger by invoking the CreateOrUpdate method
            TriggerResource triggerResource = new TriggerResource()
            {
                Properties = myTrigger
            };
            client.Triggers.CreateOrUpdate(resourceGroup, dataFactoryName, triggerName, triggerResource);

            // Start the trigger
            Console.WriteLine("Starting the trigger");
            client.Triggers.Start(resourceGroup, dataFactoryName, triggerName);
```

若要监视触发器运行，请在示例中的最后一个 `Console.WriteLine` 语句之前添加以下代码：

```csharp
            // Check that the trigger runs every 15 minutes
            Console.WriteLine("Trigger runs. You see the output every 15 minutes");

            for (int i = 0; i < 3; i++)
            {
                System.Threading.Thread.Sleep(TimeSpan.FromMinutes(15));
                List<TriggerRun> triggerRuns = client.Triggers.ListRuns(resourceGroup, dataFactoryName, triggerName, DateTime.UtcNow.AddMinutes(-15 * (i + 1)), DateTime.UtcNow.AddMinutes(2)).ToList();
                Console.WriteLine("{0} trigger runs found", triggerRuns.Count);

                foreach (TriggerRun run in triggerRuns)
                {
                    foreach (KeyValuePair<string, string> triggeredPipeline in run.TriggeredPipelines)
                    {
                        PipelineRun triggeredPipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, triggeredPipeline.Value);
                        Console.WriteLine("Pipeline run ID: {0}, Status: {1}", triggeredPipelineRun.RunId, triggeredPipelineRun.Status);
                        List<ActivityRun> runs = client.ActivityRuns.ListByPipelineRun(resourceGroup, dataFactoryName, triggeredPipelineRun.RunId, run.TriggerRunTimestamp.Value, run.TriggerRunTimestamp.Value.AddMinutes(20)).ToList();
                    }
                }
            }
```

若要在 Azure 门户中监视触发器运行和管道运行，请参阅[监视管道运行](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline)。


## <a name="python-sdk"></a>Python SDK
本部分介绍如何使用 Python SDK 创建、启动和监视触发器。 若要查看此示例运行，请先查看[快速入门：使用 Python SDK 创建数据工厂](quickstart-create-data-factory-python.md)。 然后，在 Python 脚本中的“monitor the pipeline run”代码块之后添加以下代码块。 此代码将创建在指定的开始时间和结束时间之间每 15 分钟运行一次的计划触发器。 请将 **start_time** 变量更新为当前 UTC 时间，将 **end_time** 变量更新为比当前 UTC 时间晚一小时。

```python
    # Create a trigger
    tr_name = 'mytrigger'
    scheduler_recurrence = ScheduleTriggerRecurrence(frequency='Minute', interval='15',start_time='2017-12-12T04:00:00', end_time='2017-12-12T05:00:00', time_zone='UTC')
    pipeline_parameters = {'inputPath':'adftutorial/input', 'outputPath':'adftutorial/output'}
    pipelines_to_run = []
    pipeline_reference = PipelineReference('copyPipeline')
    pipelines_to_run.append(TriggerPipelineReference(pipeline_reference, pipeline_parameters))
    tr_properties = ScheduleTrigger(description='My scheduler trigger', pipelines = pipelines_to_run, recurrence=scheduler_recurrence)    
    adf_client.triggers.create_or_update(rg_name, df_name, tr_name, tr_properties)

    # Start the trigger
    adf_client.triggers.start(rg_name, df_name, tr_name)
```

若要在 Azure 门户中监视触发器运行和管道运行，请参阅[监视管道运行](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline)。

## <a name="azure-resource-manager-template"></a>Azure 资源管理器模板
可以使用 Azure 资源管理器模板创建触发器。 有关分步说明，请参阅[使用资源管理器模板创建 Azure 数据工厂](quickstart-create-data-factory-resource-manager-template.md)。  

## <a name="pass-the-trigger-start-time-to-a-pipeline"></a>将触发器开始时间传递给管道
Azure 数据工厂版本 1 支持使用以下系统变量读取或写入分区的数据：SliceStart、SliceEnd、WindowStart 和 WindowEnd。 在 Azure 数据工厂的当前版本中，可以使用管道参数实现此行为。 触发器的开始时间和计划时间设置为管道参数的值。 在以下示例中，触发器的计划时间作为值传递给 **scheduledRunTime** 管道参数：

```json
"parameters": {
    "scheduledRunTime": "@trigger().scheduledTime"
}
```    

有关详细信息，请参阅[如何读取或写入分区的数据](how-to-read-write-partitioned-data.md)中的说明。

## <a name="json-schema"></a>JSON 架构
以下 JSON 定义演示如何创建一个带有计划和重复周期的计划触发器：

```json
{
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": <<Minute, Hour, Day, Week, Month>>,
        "interval": <<int>>,             // Optional, specifies how often to fire (default to 1)
        "startTime": <<datetime>>,
        "endTime": <<datetime - optional>>,
        "timeZone": "UTC"
        "schedule": {                    // Optional (advanced scheduling specifics)
          "hours": [<<0-23>>],
          "weekDays": [<<Monday-Sunday>>],
          "minutes": [<<0-59>>],
          "monthDays": [<<1-31>>],
          "monthlyOccurrences": [
               {
                    "day": <<Monday-Sunday>>,
                    "occurrence": <<1-5>>
               }
           ]
        }
      }
    },
   "pipelines": [
            {
                "pipelineReference": {
                    "type": "PipelineReference",
                    "referenceName": "<Name of your pipeline>"
                },
                "parameters": {
                    "<parameter 1 Name>": {
                        "type": "Expression",
                        "value": "<parameter 1 Value>"
                    },
                    "<parameter 2 Name>" : "<parameter 2 Value>"
                }
           }
      ]
  }
}
```

> [!IMPORTANT]
>  **parameters** 属性是 **pipelines** 元素的必需属性。 如果管道不使用任何参数，则必须包括一个空的针对 **parameters** 属性的 JSON 定义。


### <a name="schema-overview"></a>架构概述
下表概述了与触发器的定期触发和计划相关的主要架构元素：

| JSON 属性 | 描述 |
|:--- |:--- |
| **startTime** | 一个日期/时间值。 对于简单的计划，**startTime** 属性的值适用于第一个匹配项。 对于复杂的计划，触发器的启动时间不早于指定的 **startTime** 值。 |
| **endTime** | 触发器的结束日期和时间。 该触发器不在指定的结束日期和时间之后执行。 属性的值不能是过去的时间。 此属性是可选的。 |
| **timeZone** | 时区。 目前仅支持 UTC 时区。 |
| **recurrence** | 一个 recurrence 对象，指定触发器的定期触发规则。 recurrence 对象支持 **frequency**、**interval**、**endTime**、**count** 和 **schedule** 元素。 定义 recurrence 对象时，**frequency** 元素是必需的。 recurrence 对象的其他元素为可选元素。 |
| **frequency** | 触发器定期触发的频率单位。 支持的值包括 "minute"、"hour"、"day"、"week" 和 "month"。 |
| **interval** | 一个正整数，表示 **frequency** 值对应的时间间隔，决定了触发器的运行频率。 例如，如果 **interval** 为 3，**frequency** 为“week”，则触发器每 3 周定期触发一次。 |
| **schedule** | 触发器的定期触发计划。 指定了 **frequency** 值的触发器会根据定期触发计划更改其定期触发。 **schedule** 属性包含对定期触发的修改，这些修改是按分钟、小时、工作日、月份日期和周次进行的。


### <a name="schema-defaults-limits-and-examples"></a>架构默认值、限制和示例

| JSON 属性 | Type | 需要 | 默认值 | 有效值 | 示例 |
|:--- |:--- |:--- |:--- |:--- |:--- |
| **startTime** | String | 是 | 无 | ISO-8601 日期时间 | `"startTime" : "2013-01-09T09:30:00-08:00"` |
| **recurrence** | Object | 是 | 无 | Recurrence 对象 | `"recurrence" : { "frequency" : "monthly", "interval" : 1 }` |
| **interval** | Number | 否 | 第 | 1 到 1,000 | `"interval":10` |
| **endTime** | String | 是 | 无 | 代表将来某个时间的日期/时间值。 | `"endTime" : "2013-02-09T09:30:00-08:00"` |
| **schedule** | Object | 否 | 无 | Schedule 对象 | `"schedule" : { "minute" : [30], "hour" : [8,17] }` |

### <a name="starttime-property"></a>startTime 属性
下表说明了 **startTime** 属性如何控制触发器运行：

| startTime 值 | 不按计划循环 | 按计划循环 |
|:--- |:--- |:--- |
| 开始时间在过去 | 计算开始时间后的第一个将来执行时间，并在该时间运行。<br/><br/>根据上次执行时间进行计算后，运行后续执行。<br/><br/>请参阅此表后面的示例。 | 触发器不会早于指定的开始时间启动。 第一次执行取决于根据开始时间计算的计划。<br/><br/>后续执行的运行取决于定期触发计划。 |
| 开始时间在将来或当前 | 在指定的开始时间运行一次。<br/><br/>根据上次执行时间进行计算后，运行后续执行。 | 触发器不会早于指定的开始时间启动。 第一次执行取决于根据开始时间计算的计划。<br/><br/>后续执行的运行取决于定期触发计划。 |

让我们查看一个示例，了解在开始时间是过去的时间，并且指定了定期触发但未指定计划的情况下，会发生什么事情。 假设当前时间为 `2017-04-08 13:00`，开始时间为 `2017-04-07 14:00`，定期触发为每两天触发一次。 （定义 **recurrence** 值时，可将 **frequency** 属性设置为 "day"，**interval** 属性设置为 2。）请注意，**startTime** 值为过去的时间，发生在当前时间之前。

在上述情况下，首次执行时间为 `2017-04-09 at 14:00`。 计划程序引擎从开始时间计算执行循环。 过去的所有实例会被丢弃。 引擎将使用将来发生的下一个实例。 在本方案中，开始时间为 `2017-04-07 at 2:00pm`，因此下一个实例为从该时间算起的 2 天，即 `2017-04-09 at 2:00pm`。

首次执行时间是相同的，即使 **startTime** 值为 `2017-04-05 14:00` 或 `2017-04-01 14:00`。 在首次执行后，使用该计划计算后续执行。 因此，后续执行的时间分别为 `2017-04-11 at 2:00pm`、`2017-04-13 at 2:00pm`、`2017-04-15 at 2:00pm`，依此类推。

最后，如果没有在触发器的计划中设置小时或分钟，则会将第一次执行时对应的小时或分钟值用作默认值。

### <a name="schedule-property"></a>schedule 属性
一方面，使用 schedule 可以限制触发器执行的次数。 例如，如果触发器的频率为按月，根据计划仅在第 31 天运行，则该触发器仅在有 31 日的月份运行。

然后，计划还可以增加触发器执行的次数。 例如，如果触发器的频率为按月，根据计划在每月的第 1 天和第 2 天运行，则该触发器会在当月的第 1 天和第 2 天运行，而不是每月运行一次。

如果指定了多个 **schedule** 元素，则求值时会根据计划设置按从大到小的顺序进行。 求值从周次开始，然后是月份日期、工作日、小时，最后是分钟。

下表详细描述了 **schedule** 元素：


| JSON 元素 | 描述 | 有效值 |
|:--- |:--- |:--- |
| **分钟数** | 运行触发器的小时中的分钟。 | <ul><li>Integer</li><li>整数数组</li></ul>
| **小时数** | 运行触发器的日期中的小时。 | <ul><li>Integer</li><li>整数数组</li></ul> |
| **工作日** | 运行触发器的工作日。 此值只能使用与星期相关的频率来指定。 | <ul><li>星期一、星期二、星期三、星期四、星期五、星期六、星期日</li><li>星期值的数组（最大数组值为 7）</li><li>星期值不区分大小写</li></ul> |
| **monthlyOccurrences** | 运行触发器的月份日期。 此值只能使用与月份相关的频率来指定。 | <ul><li>MonthlyOccurrence 对象的数组：`{ "day": day,  "occurrence": occurrence }`。</li><li>**day** 属性表示运行触发器那天为星期几。 例如，如果 **monthlyOccurrences** 属性的 **day** 值为 `{Sunday}`，则表示在当月的每个星期日运行触发器。 **day** 属性是必需的。</li><li>**occurrence** 属性是指定的 **day** 在当月的匹配项。 例如，如果 **monthlyOccurrences** 属性的 **day** 和 **occurrence** 值为 `{Sunday, -1}`，则表示在当月的最后一个星期日运行触发器。 **occurrence** 属性是可选的。</li></ul> |
| **monthDays** | 运行触发器的月份日期。 此值只能使用与月份相关的频率来指定。 | <ul><li><= -1 且 >= -31 的任何值</li><li>>= 1 且 <= 31 的任何值</li><li>值组成的数组</li></ul> |


## <a name="examples-of-trigger-recurrence-schedules"></a>触发器定期触发计划示例
本部分提供定期触发计划的示例，着重于 **schedule** 对象及其元素。

这些示例假定 **interval** 值为 1，且根据计划定义，**frequency** 值是正确的。 例如，不能在 **frequency** 值为 "day" 的同时，在 **schedule** 对象中有一个 "monthDays" 修改项。 上一部分的表中提到了此类限制。

| 示例 | 描述 |
|:--- |:--- |
| `{"hours":[5]}` | 在每天早晨 5:00 运行。 |
| `{"minutes":[15], "hours":[5]}` | 在每天早晨 5:15 运行。 |
| `{"minutes":[15], "hours":[5,17]}` | 在每天早晨 5:15 和下午 5:15 运行。 |
| `{"minutes":[15,45], "hours":[5,17]}` | 在每天早晨 5:15、5:45 和下午 5:15、5:45 运行。 |
| `{"minutes":[0,15,30,45]}` | 每 15 分钟运行一次。 |
| `{hours":[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]}` | 每小时运行一次。 此触发器每小时运行一次。 如果指定了值，则分钟由 **startTime** 值控制。 如果未指定值，则分钟由创建时间控制。 例如，如果开始时间或创建时间（以适用的为准）为中午 12:25，则触发器会在 00:25、01:25、02:25 ... 23:25 运行。<br/><br/>该计划相当于为触发器指定了 **frequency** 值 "hour"、**interval** 值 1，无 **schedule**。  可以使用 **frequency** 和 **interval** 值不同于此的该计划来创建其他触发器。 例如，如果 **frequency** 值为 "month"，则此计划每月仅运行一次，而不是每天运行一次（如果 **frequency** 值为 "day"）。 |
| `{"minutes":[0]}` | 每小时整点运行。 此触发器每小时整点运行，开始时间为晚上 12:00、凌晨 1:00、凌晨 2:00，依此类推。<br/><br/>此计划相当于 **frequency** 值为 "hour"，**startTime** 值为零分钟的触发器，或者没有 **schedule** 但 **frequency** 值为 "day" 的触发器。 如果 **frequency** 值为 "week" 或 "month"，则此计划会相应地在每周的某一天或每月的某一天执行。 |
| `{"minutes":[15]}` | 在整点的 15 分钟过后运行。 此触发器每小时在整点过后 15 分钟运行，开始时间为凌晨 00:15、凌晨 1:15、凌晨 2:15，依此类推，结束时间为晚上 11:15。 |
| `{"hours":[17], "weekDays":["saturday"]}` | 在每周星期六的下午 5:00 运行。 |
| `{"hours":[17], "weekDays":["monday", "wednesday", "friday"]}` | 在每周星期一、星期三和星期五的下午 5:00 运行。 |
| `{"minutes":[15,45], "hours":[17], "weekDays":["monday", "wednesday", "friday"]}` | 在每周星期一、星期三和星期五的下午 5:15 和下午 5:45 运行。 |
| `{"minutes":[0,15,30,45], "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` | 在工作日每 15 分钟运行一次。 |
| `{"minutes":[0,15,30,45], "hours": [9, 10, 11, 12, 13, 14, 15, 16] "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` | 在工作日的早晨 9:00 到下午 4:45 之间每 15 分钟运行一次。 |
| `{"weekDays":["tuesday", "thursday"]}` | 在星期二和星期四的指定开始时间运行。 |
| `{"minutes":[0], "hours":[6], "monthDays":[28]}` | 在每月 28 日的早晨 6:00 运行（假设 **frequency** 值为 "month"）。 |
| `{"minutes":[0], "hours":[6], "monthDays":[-1]}` | 在当月最后一天的早晨 6:00 运行。 若要在某个月的最后一天运行触发器，请使用 -1 而不是 28 日、29 日、30 日或 31 日。 |
| `{"minutes":[0], "hours":[6], "monthDays":[1,-1]}` | 在每月第一天和最后一天的早晨 6:00 运行。 |
| `{monthDays":[1,14]}` | 在每月第一天和第 14 天的指定开始时间运行。 |
| `{"minutes":[0], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` | 在每月第一个星期五的早晨 5:00 运行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` | 在每月第一个星期五的指定开始时间运行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":-3}]}` | 在每月从月末算起的第三个星期五的指定开始时间运行。 |
| `{"minutes":[15], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` | 在每月第一个和最后一个星期五的早晨 5:15 运行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` | 在每月第一个和最后一个星期五的指定开始时间运行。 |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":5}]}` | 在每月第五个星期五的指定开始时间运行。 如果月份中没有第五个星期五，则管道不运行，因为该管道按计划只在第五个星期五运行。 若要在月份的最后一个星期五运行触发器，可以考虑使用 -1 而不是 5 作为 **occurrence** 值。 |
| `{"minutes":[0,15,30,45], "monthlyOccurrences":[{"day":"friday", "occurrence":-1}]}` | 在月份的最后一个星期五每 15 分钟运行一次。 |
| `{"minutes":[15,45], "hours":[5,17], "monthlyOccurrences":[{"day":"wednesday", "occurrence":3}]}` | 在每月第三个星期三的早晨 5:15、早晨 5:45、下午 5:15、下午 5:45 运行。 |


## <a name="next-steps"></a>后续步骤
有关触发器的详细信息，请参阅[管道执行和触发器](concepts-pipeline-execution-triggers.md#triggers)。
