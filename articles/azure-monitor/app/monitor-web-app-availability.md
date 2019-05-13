---
title: 监视任何网站的可用性和响应能力 | Microsoft Docs
description: 在 Application Insights 中设置 Web 测试。 当网站不可用或响应速度缓慢时接收警报。
services: application-insights
documentationcenter: ''
author: lgayhardt
manager: carmonm
ms.assetid: 46dc13b4-eb2e-4142-a21c-94a156f760ee
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 01/22/2019
ms.reviewer: sdash
ms.author: lagayhar
ms.openlocfilehash: 6cd5413d64be2117cc5f64202ecdaaf40f35db4b
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205379"
---
# <a name="monitor-availability-and-responsiveness-of-any-web-site"></a>监视任何网站的可用性和响应能力
将 Web 应用或网站部署到任何服务器之后，可以设置测试来监视其可用性和响应能力。 [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) 将来自全球各地的 Web 请求定期发送到应用程序。 如果应用程序无响应或响应太慢，则会发出警报。

对于可以从公共 Internet 访问的任何 HTTP 或 HTTPS 终结点，均可设置可用性测试。 无需将任何内容添加到要测试的网站。 它甚至不一定是站点：可以测试你所依赖的 REST API 服务。

有两种类型的可用性测试：

* [URL ping 测试](#create)：可以在 Azure 门户中创建的简单测试。
* [多步骤 Web 测试](#multi-step-web-tests)：可以在 Visual Studio Enterprise 中创建并上传到门户的测试。

对于每个应用程序资源，最多可以创建 100 个可用性测试。


## <a name="create"></a>为可用性测试报告打开资源

**如果已配置 Application Insights**（针对 Web 应用），请在 [Azure 门户](https://portal.azure.com)中打开 Application Insights 资源。

**或者，若要在新资源中查看报告**，请转到 [Azure 门户](https://portal.azure.com)，并创建 Application Insights 资源。

![创建资源 > 开发人员工具 > Application Insights](./media/monitor-web-app-availability/1create-resource-appinsights.png)

单击“所有资源”，打开新资源的“概述”边栏选项卡。

## <a name="setup"></a>创建 URL ping 测试
打开“可用性”边栏选项卡，并添加一个测试。

![至少填写网站的 URL](./media/monitor-web-app-availability/2addtest-url.png)

* **URL** 可以是要测试的任何网页，但必须在公共 Internet 中可见。 该 URL 可以包括查询字符串。 因此，例如，可以稍微训练一下数据库。 如果 URL 解析为重定向，最多可以跟踪 10 个重定向。
* **分析从属请求**：如果选中此选项，则测试将请求图像、脚本、样式文件以及其他属于受测网页的文件。 记录的响应时间包括获取这些文件所耗费的时间。 如果无法在超时期限内为整个测试成功下载所有这些资源，测试会失败。 如果不选中此选项，则测试只请求指定 URL 的文件。

* **启用重试**：如果选中此选项，则测试失败时，会在短时间后重试。 仅当连续三次尝试失败时，才报告失败。 然后，将按照一般的测试频率执行后续测试。 重试会暂停，直到下次成功为止。 可在每个测试位置单独应用此规则。 建议使用此选项。 平均大约有 80% 的失败可在重试后消除。

* **测试频率**：设置从每个测试位置运行测试的频率。 如果有五个测试位置，且默认频率为五分钟，则平均每隔一分钟测试站点一次。

* **测试位置** 是服务器将 Web 请求发送到的 URL 位置。 建议最低测试位置数目为 5，以确保可以将网站中的问题与网络问题区分开来。 最多可以选择 16 个位置。

> [!NOTE]
> * 强烈建议至少在 5 个地点进行测试。 这是为了防止可能由特定位置的暂时性问题导致的虚假警报。 此外，我们发现最佳配置是使测试位置的数目等于警报位置阈值 + 2。 
> * 启用“分析从属请求”选项会导致执行更严格的检查。 对于手动浏览站点时可能不明显的情况，测试可能会失败。

* **成功准则**：

    **测试超时**：减少此值可以接收有关响应变慢的警报。 如果未在这段时间内收到站点的响应，则将测试视为失败。 如果选择了“分析依赖请求”，则必须在这段时间内收到所有图像、样式文件、脚本和其他依赖资源。

    **HTTP 响应**：视为成功的返回状态代码。 代码 200 指示返回了正常网页。

    **内容匹配**：类似于“欢迎!”的字符串。 我们测试区分大小写的匹配项是否出现在每个响应中。 它必须是不带通配符的纯字符串。 别忘了，如果页面内容更改，可能需要更新。 **内容匹配当前支持只能使用英文字符。** 

* **警报位置阈值**：建议最少 3/5 个位置。 警报位置阈值和测试位置数目之间的最佳关系是警报位置阈值  =  测试位置数 - 2，至少有 5 个测试位置。

## <a name="multi-step-web-tests"></a>多步骤 Web 测试
可以监视涉及一连串 URL 的方案。 例如，如果正在监视销售网站，可以测试是否能够正常地将商品添加购物车。

> [!NOTE]
> 对多步骤 Web 测试要收取费用。 [定价方案](https://azure.microsoft.com/pricing/details/application-insights/)。
> 

若要创建多步骤测试，可以使用 Visual Studio Enterprise 来录制方案，然后将录制内容上传到 Application Insights。 Application Insights 将按特定间隔重放该方案，并验证响应。

> [!NOTE]
> * 不能在测试中使用编码的函数或循环。 测试必须完全包含在 .webtest 脚本中。 但是，可以使用标准插件。
> * 仅多步骤 Web 测试支持使用英文字符。 如果在其他语言中使用 Visual Studio，请更新 Web 测试定义文件以转换/排除非英文字符。
>

#### <a name="1-record-a-scenario"></a>1.录制方案
使用 Visual Studio Enterprise 录制 Web 会话。

1. 创建 Web 性能测试项目。

    ![在 Visual Studio Enterprise 版中，基于“Web 性能测试和负载测试”模板创建项目。](./media/monitor-web-app-availability/appinsights-71webtest-multi-vs-create.png)

 * *看不到 Web 性能与负载测试模板？* - 关闭 Visual Studio Enterprise。 打开 **Visual Studio 安装程序**，修改 Visual Studio Enterprise 安装。 在“各个组件”下，选择“Web 性能和负载测试工具”。

2. 打开 .webtest 文件并开始录制。

    ![打开 .webtest 文件并单击“录制”。](./media/monitor-web-app-availability/appinsights-71webtest-multi-vs-start.png)
3. 执行要在测试中模拟的用户操作：打开网站、将产品加入购物车，等等。 然后停止测试。

    ![Web 测试录制程序在 Internet Explorer 中运行。](./media/monitor-web-app-availability/appinsights-71webtest-multi-vs-record.png)

    不要录制太长的方案。 以 100 个步骤和 2 分钟为限。
4. 编辑测试：

   * 添加验证，检查收到的文本和响应代码。
   * 删除所有多余的交互。 也可以删除图片、广告或跟踪站点的依赖请求。

     请记住，只能编辑测试脚本 - 不能添加自定义代码或调用其他 Web 测试。 不要在测试中插入循环。 可以使用标准 Web 测试插件。
5. 在 Visual Studio 中运行测试，确保它可以正常工作。

    Web 测试运行器将打开 Web 浏览器，并重复录制的操作。 请确保它按预期工作。

    ![在 Visual Studio 中，打开 .webtest 文件并单击“运行”。](./media/monitor-web-app-availability/appinsights-71webtest-multi-vs-run.png)

#### <a name="2-upload-the-web-test-to-application-insights"></a>2.将 Web 测试上传到 Application Insights
1. 在 Application Insights 门户中，在“可用性”边栏选项卡上单击“添加测试”。

    ![在“可用性”边栏选项卡上，选择“添加测试”。](./media/monitor-web-app-availability/3addtest-web.png)
2. 选择多步骤测试并上传 .webtest 文件。

    像设置 ping 测试一样设置测试位置、频率和警报参数。

### <a name="plugging-time-and-random-numbers-into-your-multi-step-test"></a>将时间和随机数插入多步骤测试
假设要测试的工具从外部源获取与时间相关的数据（例如股票）。 录制 Web 测试时，必须使用具体的时间，但要将它们设置为测试参数：StartTime 和 EndTime。

![带参数的 Web 测试。](./media/monitor-web-app-availability/appinsights-72webtest-parameters.png)

运行测试时，EndTime 应该始终为当前时间，StartTime 在 15 分钟前。

Web 测试插件提供时间参数化方式。

1. 针对所需的每个变量参数值添加一个 Web 测试插件。 在 Web 测试工具栏中，选择“添加 Web 测试插件”。

    ![选择“添加 Web 测试插件”并选择类型。](./media/monitor-web-app-availability/appinsights-72webtest-plugins.png)

    本示例使用两个日期时间插件实例。 一个实例设置为“15 分钟前”，另一个实例设置为“现在”。
2. 打开每个插件的属性。 为插件命名，然后将它设置为使用当前时间。 对于其中一个插件，将“添加分钟”设置为 -15。

    ![设置名称，“使用当前时间”和“添加分钟”。](./media/monitor-web-app-availability/appinsights-72webtest-plugin-parameters.png)
3. 在 Web 测试参数中，使用 {{plug-in name}} 来引用插件名称。

    ![在测试参数中使用 {{plug-in name}}。](./media/monitor-web-app-availability/appinsights-72webtest-plugin-name.png)

现在，将测试上传到门户。 每次运行测试时，将使用动态值。


## <a name="monitor"></a>查看可用性测试结果

概述选项卡显示测试的成功率，而详细信息选项卡则显示特定详细信息的散点图和网格。

几分钟之后，单击“刷新”即可查看测试结果。

![详细信息边栏选项卡上的散点图](./media/monitor-web-app-availability/4refresh.png)

散点图显示其中都诊断测试步骤详细信息的测试结果示例。 测试引擎存储已失败的测试的诊断详细信息。 对于成功的测试，将存储执行子集的诊断详细信息。 将鼠标悬停在任何绿点/红点上，可查看测试时间戳、测试持续时间、位置和测试名称。 单击散点图中的任何点可查看测试结果的详细信息。  

选择特定测试、位置或减少时间段，可查看围绕感兴趣的时间段的更多结果。 使用搜索资源管理器以查看所有执行结果，或者使用分析查询以针对此数据运行自定义报告。

除了原始结果外，指标资源管理器中还有两个可用性指标： 

1. 可用性：已成功的测试占执行的所有测试的百分比。 
2. 测试持续时间：执行的所有测试的平均测试持续时间。

可以将筛选器应用于测试名称、位置以分析特定测试和/或位置的趋势。

## <a name="edit"></a> 检查和编辑测试

从详细信息选项卡上，在特定测试中选择最右侧的省略号以编辑、临时禁用、删除或下载 Web 测试。 可能需要 20 分钟的时间进行配置更改传播。

从特定测试中选择“查看测试详细信息”以查看其散点图和特定的测试位置详细信息。

![查看测试详细信息，编辑以及禁用 Web 测试](./media/monitor-web-app-availability/5viewdetails.png)

对服务执行维护时，可能需要禁用可用性测试或与这些测试关联的警报规则。

![禁用 Web 测试](./media/monitor-web-app-availability/6disable.png)
![编辑测试](./media/monitor-web-app-availability/8edittest.png)

## <a name="failures"></a>如果看到失败
单击红点。

![单击红点](./media/monitor-web-app-availability/open-instance-3.png)

从可用性测试结果中，可以看到所有组件的事务详细信息。 在此门户中，可以：

* 检查从服务器收到的响应。
* 使用在处理失败的可用性测试时收集的相关服务器端遥测数据进行故障诊断。
* 在 Git 或 Azure Boards 中记录问题或工作项以跟踪问题。 Bug 中将包含转至此事件的链接。
* 在 Visual Studio 中打开 Web 测试结果。

从[此处](../../azure-monitor/app/transaction-diagnostics.md)详细了解端到端事务诊断体验。

单击异常行可查看导致综合可用性测试失败的服务器端异常的详细信息。 还可以获取[调试快照](../../azure-monitor/app/snapshot-debugger.md)，进行更丰富的代码级诊断。

![服务器端诊断](./media/monitor-web-app-availability/open-instance-4.png)

## <a name="alerts"></a> 可用性警报
使用经典警报体验可以获得有关可用性数据的以下类型的警报规则：
1. 在某段时间内，Y 个位置中有 X 个报告失败
2. 聚合可用性百分比低于阈值
3. 平均测试持续时间超过阈值

### <a name="alert-on-x-out-of-y-locations-reporting-failures"></a>当 Y 个位置中有 X 个报告失败时发出警报
创建新的可用性测试时，会在[新的统一警报体验](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)中默认启用“Y 个位置中的 X 个”警报规则。 可通过选择“经典”选项或选择禁用该警报规则来选择退出。

![创建体验](./media/monitor-web-app-availability/appinsights-71webtestUpload.png)

> [!NOTE]
>  使用[新的统一警报](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)时，必须在警报体验中配置预警规则严重性和[操作组](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups)的通知首选项。 如果不执行以下步骤，则只会收到门户内通知。

1. 保存可用性测试后，在详细信息选项卡上单击你刚才所做的测试旁边的省略号。 单击“编辑警报”。
![保存后编辑](./media/monitor-web-app-availability/9editalert.png)

2. 设置所需的严重性级别、规则说明，最重要的是设置具有要用于此警报规则的通知首选项的操作组。
![保存后编辑](./media/monitor-web-app-availability/10editalert.png)


> [!NOTE]
> * 通过执行上述步骤，将操作组配置为在警报触发时接收通知。 如果不执行此步骤，则在规则触发时只会收到门户内通知。
>

### <a name="alert-on-availability-metrics"></a>根据可用性指标发出警报
使用[新的统一警报](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)时，可以根据分段聚合可用性发出警报以及测试持续时间指标：

1. 在指标体验中选择 Application Insights 资源，然后选择可用性指标：![可用性指标选择](./media/monitor-web-app-availability/selectmetric.png)

2. 从菜单中配置警报选项将转到新体验，可在其中选择特定测试或位置以将警报规则设置为打开。 还可以在此处配置此警报规则的操作组。
    ![可用性警报配置](./media/monitor-web-app-availability/availabilitymetricalert.png)

### <a name="alert-on-custom-analytics-queries"></a>根据自定义分析查询发出警报
使用[新的统一警报](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)时，可以根据[自定义日志查询](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-log)发出警报。 借助自定义查询，可以在有助于获得最可靠的可用性问题信号的任意条件下发出警报。 如果使用 TrackAvailability SDK 发送自定义可用性结果，这也特别适用。 

> [!Tip]
> * 可用性数据的指标包括可能通过调用我们的 TrackAvailability SDK 提交的任何自定义可用性结果。 可以使用“根据指标发出警报”支持根据自定义可用性结果发出警报。
>

## <a name="dealing-with-sign-in"></a>处理登录
如果用户登录应用，可以使用许多选项来模拟登录，以便可以在登录后测试页面。 使用的方法取决于应用提供的安全性类型。

在所有情况下，应该只针对测试目的在应用程序中创建帐户。 如果可能，请限制此测试帐户的权限，以便 Web 测试不会影响实际用户。

### <a name="simple-username-and-password"></a>简单的用户名和密码
以普通方式录制 Web 测试。 先删除 Cookie。

### <a name="saml-authentication"></a>SAML 身份验证
使用可用于 Web 测试的 SAML 插件。

### <a name="client-secret"></a>客户端机密
如果应用的某个登录路由涉及到客户端机密，请使用该路由。 例如，Azure Active Directory (AAD) 就是提供客户端机密登录的服务。 在 AAD 中，客户端机密是应用密钥。

下面是使用应用密钥的 Azure Web 应用的 Web 测试示例：

![客户端机密示例](./media/monitor-web-app-availability/110.png)

1. 使用客户端机密 (AppKey) 从 AAD 获取令牌。
2. 从响应中提取持有者令牌。
3. 使用授权标头中的持有者令牌调用 API。

确保 Web 测试是实际客户端 - 即，在 AAD 中有自身的应用 - 并使用其 clientId 和 appkey。 测试中的服务在 AAD 中也有自身的应用：此应用的 appID URI 反映在“resource”字段中的 Web 测试内。

### <a name="open-authentication"></a>开放身份验证
开放身份验证的示例包括使用 Microsoft 或 Google 帐户登录。 许多使用 OAuth 的应用提供替代的客户端机密，因此第一个技巧就是调查这种可能性。

如果测试必须使用 OAuth 登录，则常规方法是：

* 使用 Fiddler 等工具检查 Web 浏览器、身份验证站点与应用之间的流量。
* 使用不同的计算机或浏览器或者以较长的间隔执行两次以上的登录（使令牌过期）。
* 通过比较不同的会话，识别从身份验证站点返回的令牌，然后在登录后将此令牌传递给应用服务器。
* 使用 Visual Studio 录制 Web 测试。
* 参数化令牌，设置参数来指定从验证器返回令牌的时间，并在站点查询中使用该参数。
  （Visual Studio 会尝试参数化测试，但无法正确参数化令牌。）

## <a name="performance-tests"></a>性能测试
> [!NOTE]  
> 不推荐使用基于云的负载测试服务。 可以找到有关不推荐使用、 服务可用性和其他服务的详细信息[此处](https://docs.microsoft.com/azure/devops/test/load-test/overview?view=azure-devops)。

可以在网站上运行负载测试。 与可用性测试一样，可以从全球各地的站点发送简单请求或多步骤请求。 与可用性测试不同的是，发送的许多请求可以模拟多个并发用户。

在“配置”下，转到“性能测试”并单击“新建”以创建测试。

![创建新的性能测试](./media/monitor-web-app-availability/11new-performance-test.png)

测试完成时，会显示响应时间和成功率。

![性能测试结果](./media/monitor-web-app-availability/12performance-test.png)

> [!TIP]
> 若要观察性能测试的效果，请使用[实时流](../../azure-monitor/app/live-stream.md)和[探查器](../../azure-monitor/app/profiler.md)。
>

## <a name="automation"></a>自动化
* [使用 PowerShell 脚本自动设置可用性测试](../../azure-monitor/app/powershell.md#add-an-availability-test)。
* 设置在引发警报时调用的 [webhook](../../azure-monitor/platform/alerts-webhooks.md) 。

## <a name="qna"></a> 常见问题解答

* *站点看上去正常，但测试却失败？为什么 Application Insights 向我发出警报？*

    * 测试是否启用了“分析从属请求”？ 这会导致严格检查脚本、图像等资源。这类故障在浏览器上可能不明显。 检查所有图像、脚本、样式表和页面加载的任何其他文件。 如果其中有任何一个失败，即使 html 主页加载正常，测试也仍会报告为失败。 若要使测试对此类资源故障不再敏感，只需在测试配置中取消选中“分析从属请求”即可。 

    * 若要降低包括网络在内的各方面因素的干扰影响，请确保选中“测试故障时允许重试”配置。 也可从多个位置进行测试并对警报规则阈值进行相应的管理，防止在出现特定于位置的问题时引发不必要的警报。

    * 单击可用性体验中的任意红点或搜索资源管理器中的任意可用性故障，以查看我们报告失败的详细原因。 测试结果以及相关的服务器端遥测数据（如果启用）应该有助于了解测试失败的原因。 暂时性问题的常见原因是网络或连接问题。 

    * 测试是否超时？ 我们在 2 分钟后中止测试。 如果你的 ping 或多步骤测试花费的时间超过 2 分钟，我们会将其报告为失败。 请考虑将测试分成多个可在较短持续时间内完成的测试。

    * 是所有位置都报告失败，还是只有部分位置报告失败？ 如果只有部分位置报告失败，则可能是由网络/CDN 问题引起的。 再次单击红点应该有助于了解该位置报告失败的原因。

* *警报触发和/或解决时，都没有收到电子邮件？*

    检查经典警报配置，确认是否已直接列出你的电子邮件，或者你所在的通讯组列表是否配置为接收通知。 如果是，则检查通讯组列表配置，确认它可以接收外部电子邮件。 另外，检查邮件管理员是否有可能配置了任何可能导致此问题的策略。

* *我没有收到 Webhook 通知？*

    检查以确保接收 Webhook 通知的应用程序可用并成功处理 Webhook 请求。 有关详细信息，请参阅[此文](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-log-webhook)。

* 出现间歇性测试失败和违反协议错误？

    错误（“违反协议: CR 必须后跟 LF”）表明服务器（或依赖项）存在问题。 在响应中设置的标头格式错误时，会发生这种情况。 可能是负载均衡器或 CDN 引发的。 具体说来，某些标头可能没有使用 CRLF 来指示行结束，这违反了 HTTP 规范，因此无法通过 .NET WebRequest 级别的验证。 请检查响应，找出可能违反规范的标头。
    
    请注意:在 HTTP 标头验证比较宽松的浏览器上，URL 可能不会失败。 有关该问题的详细说明，请参阅此博客文章： http://mehdi.me/a-tale-of-debugging-the-linkedin-api-net-and-http-protocol-violations/  
    
* 看不到任何相关的、用于诊断测试失败的服务器端遥测数据？
    
    如果已为服务器端应用程序设置 Application Insights，则可能是因为[采样](../../azure-monitor/app/sampling.md)正在进行。 请选择其他可用性结果。

* *是否可以从 Web 测试调用代码？*

    不。 测试步骤必须在 .webtest 文件中指定。 此外，不能调用其他 Web 测试或使用循环。 但是可以借助一些有用的插件。

* *是否支持 HTTPS？*

    支持 TLS 1.1 和 TLS 1.2。 我们目前不会检查有 HTTPS 证书错误。  

* *“Web 测试”与“可用性测试”之间是否有差异？*

    这两个术语可以互换引用。 可用性测试是更通用的术语，其中除了包含多步骤 Web 测试外，还包含单 URL ping 测试。
    
* *如何在防火墙后面运行的内部服务器上使用可用性测试？*

    有两个可能的解决方案：
    
    * 请将防火墙配置为允许从[我们的 Web 测试代理 IP 地址](../../azure-monitor/app/ip-addresses.md)发出的传入请求。
    * 编写自己的代码，定期测试内部服务器。 在防火墙后的测试服务器上以后台进程的方式运行该代码。 测试进程可以通过核心 SDK 包中的 [TrackAvailability()](https://docs.microsoft.com/dotnet/api/microsoft.applicationinsights.telemetryclient.trackavailability) API 将其结果发送到 Application Insights。 这要求测试服务器能够以传出访问的方式访问 Application Insights 引入终结点，但与允许传入请求相比，这种方式的安全风险要小得多。 结果不会显示在可用性 Web 测试边栏选项卡中，但会作为可用性结果显示在分析、搜索和指标资源管理器中。

* *上传多步骤 Web 测试失败*

    可能导致此问题的一些原因包括：
    * 存在 300 K 大小限制。
    * 不支持循环。
    * 不支持对其他 Web 测试的引用。
    * 不支持数据源。

* *多步骤测试无法完成*

    存在每个测试 100 个请求的限制。 此外，如果运行时间超过两分钟，测试会停止。

* *如何使用客户端证书运行测试？*

    抱歉，不支持这种测试。

## <a name="who-receives-the-classic-alert-notifications"></a>谁会收到（经典）警报通知？

本节仅适用于经典警报，并将帮助优化警报通知以确保只有预期的接收人能收到通知。 若要详细了解[经典警报](../platform/alerts-classic.overview.md)与新的警报体验之间的区别，请参阅[警报概述文章](../platform/alerts-overview.md)。 若要控制新的警报体验中的警报通知，请使用[操作组](../platform/action-groups.md)。

* 建议将经典警报通知用于特定接收人。

* 对于 Y 个位置中 X 个位置的失败相关警报，如已启用“批/组”复选框选项，会向具有管理员/共同管理员角色的用户发送相关通知。  实质上是_订阅_的_所有_管理员均会收到通知。

* 对于有关可用性指标（或任何相关 Application Insights 指标）的警报，如果启用“批/组”复选框选项，则会发送给订阅中具有所有者、参与者或读者角色的用户。 实际上，可以访问订阅 Application Insights 资源的_所有_用户均会接收到通知。 

> [!NOTE]
> 如果当前使用“批/组”复选框选项并禁用它，则无法还原更改。

如果需要根据用户角色通知用户，请使用新的警报体验/近实时警报。 使用[操作组](../platform/action-groups.md)，可以为具有任何参与者/所有者/读者角色的用户配置电子邮件通知（不作为单一选项组合在一起）。



## <a name="next"></a>后续步骤
[搜索诊断日志][diagnostic]

[故障排除][qna]

[Web 测试代理 IP 地址](../../azure-monitor/app/ip-addresses.md)

<!--Link references-->

[azure-availability]: ../../insights-create-web-tests.md
[diagnostic]: ../../azure-monitor/app/diagnostic-search.md
[qna]: ../../azure-monitor/app/troubleshoot-faq.md
[start]: ../../azure-monitor/app/app-insights-overview.md
