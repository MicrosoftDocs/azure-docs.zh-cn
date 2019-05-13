---
title: 测试 Azure Functions
description: 为 Visual Studio 中的 C# 函数和 VS Code 中的 JavaScript 函数创建自动测试
services: functions
documentationcenter: na
author: craigshoemaker
manager: jeconnoc
keywords: Azure Functions，函数，事件处理，webhook，动态计算，无服务体系结构，测试
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: cshoe
ms.openlocfilehash: 9a078579fff355d7ddb996316af2a2136fb62335
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473333"
---
# <a name="strategies-for-testing-your-code-in-azure-functions"></a>在 Azure Functions 中测试代码的策略

本文演示如何为 Azure Functions 创建自动测试。 

建议测试所有代码，但是，包装函数的逻辑并在函数外部创建测试可以获得最佳效果。 将逻辑抽象出来可以限制函数的代码行，并让该函数专门负责调用其他类或模块。 但是，本文演示如何针对 HTTP 和计时器触发的函数创建自动测试。

下面的内容根据不同的语言和环境划分为两个不同的部分。 可以了解：

- [使用 xUnit 在 Visual Studio 的 C# 中生成测试](#c-in-visual-studio)
- [使用 Jest 在 VS Code 的 JavaScript 中生成测试](#javascript-in-vs-code)

[GitHub](https://github.com/Azure-Samples/azure-functions-tests) 上提供了示例存储库。

## <a name="c-in-visual-studio"></a>Visual Studio 中的 C#
以下示例描述如何在 Visual Studio 中创建 C# 函数应用，并使用 [xUnit](https://xunit.github.io) 运行和测试该应用。

![使用 Visual Studio 中的 C# 测试 Azure Functions](./media/functions-test-a-function/azure-functions-test-visual-studio-xunit.png)

### <a name="setup"></a>设置

若要设置环境，请创建一个函数和测试应用。 以下步骤可帮助你创建用于支持测试的应用和函数：

1. [创建新函数应用](./functions-create-first-azure-function.md)并将其命名为 *Functions*
2. [从模板创建 HTTP 函数](./functions-create-first-azure-function.md)并将其命名为 *HttpTrigger*。
3. [从模板创建计时器函数](./functions-create-scheduled-function.md)并将其命名为 *TimerTrigger*。
4. 单击“文件”>“新建”>“项目”>“Visual C#”>“.NET Core”>“xUnit 测试项目”，在 Visual Studio 中[创建 xUnit 测试应用](https://xunit.github.io/docs/getting-started-dotnet-core)，并将其命名为“Functions.Test”。 
5. 使用 Nuget 添加引用从测试应用[Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc/)
6. [从 *Functions.Test* 应用引用 *Functions* 应用](https://docs.microsoft.com/visualstudio/ide/managing-references-in-a-project?view=vs-2017)。

### <a name="create-test-classes"></a>创建测试类

创建应用程序后，可以创建用于运行自动测试的类。

每个函数采用 [ILogger](https://docs.microsoft.com/dotnet/api/microsoft.extensions.logging.ilogger) 的实例来处理消息日志记录。 有些测试不记录消息，或者与日志记录的实现方式无关。 还有一些测试需要评估记录的消息，以确定是否通过了测试。

`ListLogger` 类用于实现 `ILogger` 接口并保存在消息的内部列表中，以便在测试期间用于评估。

**右键单击**上*Functions.Test*应用程序并选择**添加 > 类**，其命名为**NullScope.cs**并输入以下代码：

```csharp
using System;

namespace Functions.Tests
{
    public class NullScope : IDisposable
    {
        public static NullScope Instance { get; } = new NullScope();

        private NullScope() { }

        public void Dispose() { }
    }
}
```

下一步，**右击**上*Functions.Test*应用程序并选择**添加 > 类**，其命名为**ListLogger.cs**和输入以下代码：

```csharp
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Text;

namespace Functions.Tests
{
    public class ListLogger : ILogger
    {
        public IList<string> Logs;

        public IDisposable BeginScope<TState>(TState state) => NullScope.Instance;

        public bool IsEnabled(LogLevel logLevel) => false;

        public ListLogger()
        {
            this.Logs = new List<string>();
        }

        public void Log<TState>(LogLevel logLevel, 
                                EventId eventId,
                                TState state,
                                Exception exception,
                                Func<TState, Exception, string> formatter)
        {
            string message = formatter(state, exception);
            this.Logs.Add(message);
        }
    }
}
```

`ListLogger` 类实现 `ILogger` 接口收缩的以下成员：

- **BeginScope**：范围将上下文添加到日志记录。 在这种情况下，测试只是指向静态实例上`NullScope`类，以允许对函数的测试。

- **IsEnabled**：提供 `false` 的默认值。

- **Log**：此方法使用提供的 `formatter` 函数来设置消息格式，然后将生成的文本添加到 `Logs` 集合。

`Logs` 集合是 `List<string>` 的实例，在构造函数中初始化。

接下来，**右键单击**“Functions.Test”应用程序并选择“添加”>“类”，将类命名为 **LoggerTypes.cs**，然后输入以下代码：

```csharp
namespace Functions.Tests
{
    public enum LoggerTypes
    {
        Null,
        List
    }
}
```
此枚举指定测试使用的记录器类型。 

接下来，**右键单击**“Functions.Test”应用程序并选择“添加”>“类”，将类命名为 **TestFactory.cs**，然后输入以下代码：

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Http.Internal;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Logging.Abstractions;
using Microsoft.Extensions.Primitives;
using System.Collections.Generic;

namespace Functions.Tests
{
    public class TestFactory
    {
        public static IEnumerable<object[]> Data()
        {
            return new List<object[]>
            {
                new object[] { "name", "Bill" },
                new object[] { "name", "Paul" },
                new object[] { "name", "Steve" }

            };
        }

        private static Dictionary<string, StringValues> CreateDictionary(string key, string value)
        {
            var qs = new Dictionary<string, StringValues>
            {
                { key, value }
            };
            return qs;
        }

        public static DefaultHttpRequest CreateHttpRequest(string queryStringKey, string queryStringValue)
        {
            var request = new DefaultHttpRequest(new DefaultHttpContext())
            {
                Query = new QueryCollection(CreateDictionary(queryStringKey, queryStringValue))
            };
            return request;
        }

        public static ILogger CreateLogger(LoggerTypes type = LoggerTypes.Null)
        {
            ILogger logger;

            if (type == LoggerTypes.List)
            {
                logger = new ListLogger();
            }
            else
            {
                logger = NullLoggerFactory.Instance.CreateLogger("Null Logger");
            }

            return logger;
        }
    }
}
```
`TestFactory` 类实现以下成员：

- **Data**：此属性返回示例数据的 [IEnumerable](https://docs.microsoft.com/dotnet/api/system.collections.ienumerable) 集合。 键/值对表示传入查询字符串中的值。

- **CreateDictionary**：此方法接受键/值对作为参数，并返回新的 `Dictionary` 用于创建 `QueryCollection` 来表示查询字符串值。

- **CreateHttpRequest**：此方法创建使用给定查询字符串参数初始化的 HTTP 请求。

- **CreateLogger**：此方法基于记录器类型返回用于测试的记录器类。 `ListLogger` 跟踪可在测试中评估的记录消息。

接下来，**右键单击**“Functions.Test”应用程序并选择“添加”>“类”，将类命名为 **FunctionsTests.cs**，然后输入以下代码：

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using Xunit;

namespace Functions.Tests
{
    public class FunctionsTests
    {
        private readonly ILogger logger = TestFactory.CreateLogger();

        [Fact]
        public async void Http_trigger_should_return_known_string()
        {
            var request = TestFactory.CreateHttpRequest("name", "Bill");
            var response = (OkObjectResult)await HttpFunction.Run(request, logger);
            Assert.Equal("Hello, Bill", response.Value);
        }

        [Theory]
        [MemberData(nameof(TestFactory.Data), MemberType = typeof(TestFactory))]
        public async void Http_trigger_should_return_known_string_from_member_data(string queryStringKey, string queryStringValue)
        {
            var request = TestFactory.CreateHttpRequest(queryStringKey, queryStringValue);
            var response = (OkObjectResult)await HttpFunction.Run(request, logger);
            Assert.Equal($"Hello, {queryStringValue}", response.Value);
        }

        [Fact]
        public void Timer_should_log_message()
        {
            var logger = (ListLogger)TestFactory.CreateLogger(LoggerTypes.List);
            TimerTrigger.Run(null, logger);
            var msg = logger.Logs[0];
            Assert.Contains("C# Timer trigger function executed at", msg);
        }
    }
}
```
在此类中实现的成员包括：

- **Http_trigger_should_return_known_string**：此测试创建对 HTTP 函数发出的、其查询字符串值为 `name=Bill` 的请求，并检查是否返回了预期的响应。

- **Http_trigger_should_return_string_from_member_data**：此测试使用 xUnit 属性为 HTTP 函数提供示例数据。

- **Timer_should_log_message**：此测试创建 `ListLogger` 的实例并将其传递给计时器函数。 运行该函数后，将检查日志以确保存在预期的消息。

如果你想要访问在测试中的应用程序设置，则可以使用[System.Environment.GetEnvironmentVariable](./functions-dotnet-class-library.md#environment-variables)。

### <a name="run-tests"></a>运行测试

若要运行测试，请导航到“测试资源管理器”并单击“全部运行”。

![使用 Visual Studio 中的 C# 测试 Azure Functions](./media/functions-test-a-function/azure-functions-test-visual-studio-xunit.png)

### <a name="debug-tests"></a>调试测试

若要调试测试，请在测试中设置一个断点，导航到“测试资源管理器”，然后单击“运行”>“调试上次运行”。

## <a name="javascript-in-vs-code"></a>VS Code 中的 JavaScript

以下示例描述如何在 VS Code 中创建 JavaScript 函数应用，并使用 [Jest](https://jestjs.io) 运行和测试该应用。 此过程使用 [VS Code Functions 扩展](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)来创建 Azure Functions。

![使用 VS Code 中的 JavaScript 测试 Azure Functions](./media/functions-test-a-function/azure-functions-test-vs-code-jest.png)

### <a name="setup"></a>设置

若要设置环境，请运行 `npm init`，在空文件夹中初始化新的 Node.js 应用。

```bash
npm init -y
```
接下来，运行以下命令安装 Jest：

```bash
npm i jest
```
现在，请更新 _package.json_ 以将现有的 test 命令替换为以下命令：

```bash
"scripts": {
    "test": "jest"
}
```

### <a name="create-test-modules"></a>创建测试模块
初始化项目后，可以创建用于运行自动测试的模块。 首先，创建名为 *testing* 的新文件夹用于保存支持模块。

在 *testing* 文件夹中添加一个新文件，将其命名为 **defaultContext.js**，然后添加以下代码：

```javascript
module.exports = {
    log: jest.fn()
};
```
此模块模拟 *log* 函数来表示默认的执行上下文。

接下来，添加一个新文件，将其命名为 **defaultTimer.js**，然后添加以下代码：

```javascript
module.exports = {
    IsPastDue: false
};
```
此模块实现 `IsPastDue` 属性，表示该实例是一个虚构的计时器实例。

接下来，使用 VS Code Functions 扩展[创建新的 JavaScript HTTP 函数](https://code.visualstudio.com/tutorials/functions-extension/getting-started)，并将其命名为 *HttpTrigger*。 创建函数后，在名为 **index.test.js** 的同一文件夹中添加一个新文件，然后添加以下代码：

```javascript
const httpFunction = require('./index');
const context = require('../testing/defaultContext')

test('Http trigger should return known text', async () => {

    const request = {
        query: { name: 'Bill' }
    };

    await httpFunction(context, request);

    expect(context.log.mock.calls.length).toBe(1);
    expect(context.res.body).toEqual('Hello Bill');
});
```
模板中的 HTTP 函数返回与查询字符串中提供的名称连接在一起的“Hello”字符串。 此测试创建虚构的请求实例，并将其传递给 HTTP 函数。 此测试检查 *log* 方法是否调用了一次，并且返回的文本是否等于“Hello Bill”。

接下来，使用 VS Code Functions 扩展创建新的 JavaScript 计时器函数，并将其命名为 *TimerTrigger*。 创建函数后，在名为 **index.test.js** 的同一文件夹中添加一个新文件，然后添加以下代码：

```javascript
const timerFunction = require('./index');
const context = require('../testing/defaultContext');
const timer = require('../testing/defaultTimer');

test('Timer trigger should log message', () => {
    timerFunction(context, timer);
    expect(context.log.mock.calls.length).toBe(1);
});
```
模板中的计时器函数在函数正文的末尾记录一条消息。 此测试确保调用 *log* 函数一次。

### <a name="run-tests"></a>运行测试
若要运行测试，请按 **CTRL + ~** 打开命令窗口，然后运行 `npm test`：

```bash
npm test
```

![使用 VS Code 中的 JavaScript 测试 Azure Functions](./media/functions-test-a-function/azure-functions-test-vs-code-jest.png)

### <a name="debug-tests"></a>调试测试

若要调试测试，请将以下配置添加到 *launch.json* 文件中：

```json
{
  "type": "node",
  "request": "launch",
  "name": "Jest Tests",
  "program": "${workspaceRoot}\\node_modules\\jest\\bin\\jest.js",
  "args": [
      "-i"
  ],
  "internalConsoleOptions": "openOnSessionStart"
}
```

接下来，在测试中设置一个断点并按 **F5**。

## <a name="next-steps"></a>后续步骤

了解如何为函数编写自动测试后，请继续参阅以下资源：
- [手动运行非 HTTP 触发的函数](./functions-manually-run-non-http.md)
- [Azure Functions 错误处理](./functions-bindings-error-pages.md)
- [Azure 函数事件网格触发器本地调试](./functions-debug-event-grid-trigger-local.md)
