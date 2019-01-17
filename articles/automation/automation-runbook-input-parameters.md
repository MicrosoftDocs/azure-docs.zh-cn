---
title: Runbook 输入参数
description: Runbook 输入参数可让你将数据传递到启动的 Runbook，以增加 Runbook 的弹性。 本文介绍在 Runbook 中使用输入参数的不同方案。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 650effed388dde4419e2ff6aede2f0468551a959
ms.sourcegitcommit: 8d88a025090e5087b9d0ab390b1207977ef4ff7c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/21/2018
ms.locfileid: "52276680"
---
# <a name="runbook-input-parameters"></a>Runbook 输入参数

Runbook 输入参数允许在启动 Runbook 时向它传递数据，增加 Runbook 的灵活性。 这些参数可让 Runbook 操作以特定方案和环境为目标。 本文将介绍在 Runbook 中使用输入参数的不同方案。

## <a name="configure-input-parameters"></a>配置输入参数

可以在 PowerShell、PowerShell 工作流、Python 和图形 Runbook 中配置输入参数。 一个 Runbook 可以包含具有不同数据类型的多个参数，或者不包含任何参数。 输入参数可以是必需的或可选的，可以为可选参数分配默认值。 可以在通过某种可用方法启动 Runbook 时分配 Runbook 的输入参数值。 这些方法包括从门户或 Web 服务启动 Runbook。 还可以启动一个 Runbook 作为另一个 Runbook 中内联调用的子 Runbook。

## <a name="configure-input-parameters-in-powershell-and-powershell-workflow-runbooks"></a>在 PowerShell 和 PowerShell 工作流 Runbook 中配置输入参数

Azure 自动化中的 PowerShell 和 [PowerShell 工作流 Runbook](automation-first-runbook-textual.md) 支持通过以下属性定义的输入参数：  

| **属性** | **说明** |
|:--- |:--- |
| 类型 |必需。 参数值所需的数据类型。 任何 .NET 类型均有效。 |
| 名称 |必需。 参数的名称。 在 Runbook 中必须唯一，并且只能包含字母、数字或下划线字符。 必须以字母开头。 |
| 必需 |可选。 指定是否必须为该参数提供值。 如果将此项设置为 **$true**，则启动 Runbook 时必须提供一个值。 如果将此项设置为 **$false**，则值是可选的。 |
| 默认值 |可选。 指定在启动 Runbook 时未传入值的情况下要用于参数的值。 可为任何参数设置默认值，此值将使参数自动成为可选，而不管 Mandatory 设置为何。 |

Windows PowerShell 支持的输入参数属性比此处所列的多，例如验证、别名和参数集。 但是，Azure 自动化目前仅支持上述输入参数。

PowerShell 工作流 Runbook 中的参数定义采用以下常规格式，其中，多个参数必须以逗号分隔。

```powershell
Param
(
  [Parameter (Mandatory= $true/$false)]
  [Type] $Name1 = <Default value>,

  [Parameter (Mandatory= $true/$false)]
  [Type] $Name2 = <Default value>
)
```

> [!NOTE]
> 定义参数时，如果未指定 **Mandatory** 属性，则会默认将参数视为可选。 此外，如果在 PowerShell 工作流 Runbook 中设置某个参数的默认值，则 PowerShell 会将其视为可选参数，而不考虑 **Mandatory** 属性的值。
> 
> 

例如，让我们为输出有关虚拟机（可以是单个 VM 或资源组中的所有 VM）的详细信息的 PowerShell 工作流 Runbook 配置输入参数。 如以下屏幕截图中所示，此 Runbook 有两个参数：虚拟机的名称和资源组的名称。

![自动化 PowerShell 工作流](media/automation-runbook-input-parameters/automation-01-powershellworkflow.png)

在此参数定义中，**$VMName** 和 **$resourceGroupName** 参数是字符串类型的简单参数。 但是，PowerShell 和 PowerShell 工作流 Runbook 支持所有简单类型和复杂类型，例如输入参数的 **object** 或 **PSCredential**。

如果 Runbook 有 object 类型输入参数，则使用包含 (name, value) 对的 PowerShell 哈希表来传入值。 例如，如果 Runbook 中有以下参数：

```powershell
[Parameter (Mandatory = $true)]
[object] $FullName
```

则可将以下值传递到该参数：

```powershell
@{"FirstName"="Joe";"MiddleName"="Bob";"LastName"="Smith"}
```
> [!NOTE]
> 如果没有将值传递到具有的 _默认值_ `$null` 的可选 `[String]` 类型参数，则参数的值将为 _空字符串_ ，而不是 `$null`。
> 
> 

## <a name="configure-input-parameters-in-graphical-runbooks"></a>在图形 Runbook 中配置输入参数

为了使用输入参数[配置图形 Runbook](automation-first-runbook-graphical.md)，让我们创建输出有关虚拟机（可以是单个 VM 或资源组中的所有 VM）的详细信息的图形 Runbook。 配置 Runbook 包括两个主要活动，如下所述。

[使用 Azure 运行方式帐户进行 Runbook 身份验证](automation-sec-configure-azure-runas-account.md)，以便与 Azure 进行身份验证。

通过 [**Get-AzureRmVm**](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvm) 获取虚拟机属性。

可使用 [**Write-Output**](https://technet.microsoft.com/library/hh849921.aspx) 活动输出虚拟机的名称。 **Get-AzureRmVm** 活动接受两个参数：**虚拟机名称**和**资源组名称**。 由于这些参数在每次启动 Runbook 时可能需要不同的值，因此可以将输入参数添加到 Runbook。 以下是添加输入参数的步骤：

1. 从“Runbook”边栏选项卡选择图形 Runbook，并单击[“编辑”](automation-graphical-authoring-intro.md)进行编辑。
2. 在 Runbook 编辑器中，单击“输入和输出”，打开“输入和输出”边栏选项卡。
   
    ![自动化图形 Runbook](media/automation-runbook-input-parameters/automation-02-graphical-runbok-editor.png)
3. “输入和输出”边栏选项卡显示针对 Runbook 定义的输入参数列表。 在此边栏选项卡上，可以添加新的输入参数，或编辑现有输入参数的配置。 若要为 Runbook 添加新参数，请单击“添加输入”，打开“Runbook 输入参数”边栏选项卡。 在此处可以配置以下参数：
   
   | **属性** | **说明** |
   |:--- |:--- |
   | 名称 |必需。 参数的名称。 在 Runbook 中必须唯一，并且只能包含字母、数字或下划线字符。 必须以字母开头。 |
   | Description |可选。 有关输入参数用途的说明。 |
   | 类型 |可选。 参数值的预期数据类型。 支持的参数类型包括 **String**、**Int32**、**Int64**、**Decimal**、**Boolean**、**DateTime** 和 **Object**。 如果未选择数据类型，将默认为 **String**。 |
   | 必需 |可选。 指定是否必须为该参数提供值。 如果选择“是”，则启动 Runbook 时必须提供一个值。 如果选择“否”，则启动 Runbook 时不需要提供值，并且可以设置一个默认值。 |
   | 默认值 |可选。 指定在启动 Runbook 时未传入值的情况下要用于参数的值。 对于不是必需的参数，可以设置默认值。 若要设置默认值，请选择“自定义”。 除非在 Runbook 启动时提供了其他值，否则将使用此值。 如果不想提供任何默认值，请选择“无”。 |
   
    ![添加新输入](media/automation-runbook-input-parameters/automation-runbook-input-parameter-new.png)
4. 使用以下属性，创建 **Get-AzureRmVm** 活动使用的两个参数：
   
   * **参数 1：**
     
     * 名称 - VMName
     * 类型 - String
     * 必需 - 否
   * **参数 2：**
     
     * 名称 - resourceGroupName
     * 类型 - String
     * 必需 - 否
     * 默认值 - 自定义
     * 自定义默认值 - \<包含虚拟机的资源组的名称>
5. 添加参数后，单击“确定”。 现在，可在“输入和输出”边栏选项卡中查看这些参数。 再次单击“确定”，并单击“保存”“发布”Runbook。

## <a name="configure-input-parameters-in-python-runbooks"></a>在 Python Runbook 中配置输入参数

与 PowerShell、PowerShell 工作流和图形 Runbook 不同，Python Runbook 不使用命名参数。
所有输入形参均解析为实参值数组。
通过将 `sys` 模块导入 Python 脚本然后使用 `sys.argv` 数组来访问数组。
请务必注意，数组的第一个元素 `sys.argv[0]` 是脚本的名称，因此第一个输入参数实际上是 `sys.argv[1]`。

若要获取如何在 Python Runbook 中使用输入参数的示例，请参阅[我在 Azure 自动化中的第一个 Python Runbook](automation-first-runbook-textual-python2.md)。

## <a name="assign-values-to-input-parameters-in-runbooks"></a>为 Runbook 中的输入参数赋值

在以下情况下，可以将值传递到 Runbook 中的输入参数：

### <a name="start-a-runbook-and-assign-parameters"></a>启动 Runbook 并分配参数

Runbook 有多种启动方式：通过 Azure 门户、Webhook、PowerShell cmdlet、REST API 或 SDK。 下面介绍了启动 Runbook 和分配参数的不同方法。

#### <a name="start-a-published-runbook-by-using-the-azure-portal-and-assign-parameters"></a>使用 Azure 门户启动已发布的 Runbook 并分配参数

[启动 Runbook](automation-starting-a-runbook.md#starting-a-runbook-with-the-azure-portal) 时，“启动 Runbook”边栏选项卡会打开，可以为所创建的参数输入值。

![使用门户启动](media/automation-runbook-input-parameters/automation-04-startrunbookusingportal.png)

在输入框下面的标签中，可以查看为参数设置的属性。 属性包括必需或可选状态、类型和默认值。 在参数名称旁边的帮助气球中，可以查看做出参数输入值相关决策时所需的所有关键信息。 此信息包括参数是必需还是可选的。 此外还包括类型和默认值（如果有）及其他有用的说明。

> [!NOTE]
> 字符串类型参数支持**空**字符串值。  在输入参数框中输入 **[EmptyString]** 将向参数传递空字符串。 另外，字符串类型参数不支持传递 **Null** 值。 如果未向字符串参数传递任何值，PowerShell 会将值解释为 NULL。
> 
> 

#### <a name="start-a-published-runbook-by-using-powershell-cmdlets-and-assign-parameters"></a>使用 PowerShell cmdlet 启动已发布的 Runbook 并分配参数

* **Azure 资源管理器 cmdlet**：可使用 [Start-AzureRmAutomationRunbook](https://docs.microsoft.com/powershell/module/azurerm.automation/start-azurermautomationrunbook) 启动在资源组中创建的自动化 Runbook。
  
  **示例：**
  
  ```powershell
  $params = @{“VMName”=”WSVMClassic”;”resourceGroupeName”=”WSVMClassicSG”}
  
  Start-AzureRmAutomationRunbook -AutomationAccountName “TestAutomation” -Name “Get-AzureVMGraphical” –ResourceGroupName $resourceGroupName -Parameters $params
  ```
* **Azure 经典部署模型 cmdlet**：可以使用 [Start-AzureAutomationRunbook](https://docs.microsoft.com/powershell/module/servicemanagement/azure/start-azureautomationrunbook) 启动在默认资源组中创建的自动化 Runbook。
  
  **示例：**
  
  ```powershell
  $params = @{“VMName”=”WSVMClassic”; ”ServiceName”=”WSVMClassicSG”}
  
  Start-AzureAutomationRunbook -AutomationAccountName “TestAutomation” -Name “Get-AzureVMGraphical” -Parameters $params
  ```

> [!NOTE]
> 使用 PowerShell cmdlet 启动 Runbook 时，将创建值为 **PowerShell** 的默认参数 **MicrosoftApplicationManagementStartedBy**。 可在“作业详细信息”边栏选项卡中查看此参数。  
> 
> 

#### <a name="start-a-runbook-by-using-an-sdk-and-assign-parameters"></a>使用 SDK 启动 Runbook 并分配参数

* **Azure 资源管理器方法：** 可使用编程语言的 SDK 来启动 Runbook。 以下 C# 代码段用于在自动化帐户中启动 Runbook。 可以在 [GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/blob/master/src/ResourceManagement/Automation/Automation.Tests/TestSupport/AutomationTestBase.cs)中查看完整代码。  
  
  ```csharp
   public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
      {
        var response = AutomationClient.Jobs.Create(resourceGroupName, automationAccount, new JobCreateParameters
         {
            Properties = new JobCreateProperties
             {
                Runbook = new RunbookAssociationProperty
                 {
                   Name = runbookName
                 },
                   Parameters = parameters
             }
         });
      return response.Job;
      }
  ```
* **Azure 经典部署模型方法：** 可使用编程语言的 SDK 启动 Runbook。 以下 C# 代码段用于在自动化帐户中启动 Runbook。 可以在 [GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/blob/master/src/ServiceManagement/Automation/Automation.Tests/TestSupport/AutomationTestBase.cs)中查看完整代码。
  
  ```csharp
  public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
    {
      var response = AutomationClient.Jobs.Create(automationAccount, new JobCreateParameters
    {
      Properties = new JobCreateProperties
         {
           Runbook = new RunbookAssociationProperty
         {
           Name = runbookName
              },
                Parameters = parameters
              }
       });
      return response.Job;
    }
  ```
  
  若要启动此方法，请创建一个字典来存储 Runbook 参数（**VMName** 和 **resourceGroupName**）及其值。 然后启动 Runbook。 以下 C# 代码段用于调用上面定义的方法。
  
  ```csharp
  IDictionary<string, string> RunbookParameters = new Dictionary<string, string>();
  
  // Add parameters to the dictionary.
  RunbookParameters.Add("VMName", "WSVMClassic");
  RunbookParameters.Add("resourceGroupName", "WSSC1");
  
  //Call the StartRunbook method with parameters
  StartRunbook(“Get-AzureVMGraphical”, RunbookParameters);
  ```

#### <a name="start-a-runbook-by-using-the-rest-api-and-assign-parameters"></a>使用 REST API 启动 Runbook 并分配参数
可在以下请求 URI 中使用 PUT 方法通过 Azure 自动化 REST API 来创建并启动 Runbook 作业： https://management.core.windows.net/<subscription-id>/cloudServices/<cloud-service-name>/resources/automation/~/automationAccounts/<automation-account-name>/jobs/<job-id>?api-version=2014-12-08`


在请求 URI 中替换以下参数：

* **subscription-id：** Azure 订阅 ID。  
* **cloud-service-name：** 请求所要发送到的云服务的名称。  
* **automation-account-name：** 托管在指定云服务中的自动化帐户的名称。  
* **job-id：** 作业的 GUID。 使用 **[GUID]::NewGuid().ToString()** 命令可以创建 PowerShell 中的 GUID。

要将参数传递到 Runbook 作业，请使用请求正文。 它采用两个以 JSON 格式提供的属性：

* **名称：** 必需。 作业要启动的 Runbook 的名称。  
* **Runbook 参数：** 可选。 参数列表的字典；列表必须采用 (名称, 值) 格式，其中的名称应为字符串类型，值可以是任何有效的 JSON 值。

如果想要启动之前以 **VMName** 和 **resourceGroupName** 作为参数创建的 **Get-AzureVMTextual Runbook**，请使用以下 JSON 格式的请求正文。

   ```json
    {
      "properties":{
        "runbook":{
        "name":"Get-AzureVMTextual"},
      "parameters":{
         "VMName":"WSVMClassic",
         "resourceGroupName":”WSCS1”}
        }
    }
   ```

如果成功创建了作业，将返回 HTTP 状态代码 201。 有关响应标头和响应正文的详细信息，请参阅有关如何[使用 REST API 创建 Runbook 作业](https://msdn.microsoft.com/library/azure/mt163849.aspx)的文章。

### <a name="test-a-runbook-and-assign-parameters"></a>测试 Runbook 并分配参数
使用测试选项[测试 Runbook 的草稿版本](automation-testing-runbook.md)时，将打开“测试”页，可在其中为所创建的参数配置值。

![测试并分配参数](media/automation-runbook-input-parameters/automation-06-testandassignparameters.png)

### <a name="link-a-schedule-to-a-runbook-and-assign-parameters"></a>将计划链接到 Runbook 并分配参数
可[将计划链接](automation-schedules.md)到 Runbook，以便在特定的时间启动 Runbook。 创建计划时将指定输入参数，Runbook 在按计划启动时，将使用这些值。 只有在提供所有必需参数值之后，才可以保存计划。

![计划并分配参数](media/automation-runbook-input-parameters/automation-07-scheduleandassignparameters.png)

### <a name="create-a-webhook-for-a-runbook-and-assign-parameters"></a>创建 Runbook 的 Webhook 并分配参数
可为 Runbook 创建 [Webhook](automation-webhooks.md) 并配置 Runbook 输入参数。 只有在提供所有必需参数值之后，才可以保存 Webhook。

![创建 webhook 并分配参数](media/automation-runbook-input-parameters/automation-08-createwebhookandassignparameters.png)

使用 Webhook 执行 Runbook 时，会发送预定义的输入参数 **[Webhookdata](automation-webhooks.md#details-of-a-webhook)** 定义的输入参数。 可单击 **WebhookData** 参数将它展开，以查看更多详细信息。

![WebhookData 参数](media/automation-runbook-input-parameters/automation-09-webhook-data-parameters.png)

## <a name="next-steps"></a>后续步骤
* 有关 Runbook 输入和输出的详细信息，请参阅 [Azure Automation: runbook input, output, and nested runbooks](https://azure.microsoft.com/blog/azure-automation-runbook-input-output-and-nested-runbooks/)（Azure 自动化：Runbook 输入、输出和嵌套 Runbook）。
* 有关以不同方式启动 Runbook 的详细信息，请参阅[启动 Runbook](automation-starting-a-runbook.md)。
* 若要编辑文本 Runbook，请参阅[编辑文本 Runbook](automation-edit-textual-runbook.md)。
* 若要编辑图形 Runbook，请参阅 [Azure 自动化中的图形创作](automation-graphical-authoring-intro.md)。

