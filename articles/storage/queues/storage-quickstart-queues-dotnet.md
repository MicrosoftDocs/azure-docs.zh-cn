---
title: 快速入门：使用 .NET 在 Azure 存储中创建队列
description: 本快速入门介绍如何使用 .NET 的 Azure 存储客户端库创建队列并在其中添加消息。 接下来，介绍如何在队列中读取和处理消息。
services: storage
author: mhopkins-msft
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 02/06/2018
ms.author: mhopkins
ms.reviewer: cbrooks
ms.openlocfilehash: b80ff38e2afb5e9da6227537ea9c483566d03e4d
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65232600"
---
# <a name="quickstart-use-net-to-create-a-queue-in-azure-storage"></a>快速入门：使用 .NET 在 Azure 存储中创建队列

本快速入门介绍如何使用 .NET 的 Azure 存储客户端库创建队列并在其中添加消息。 接下来，介绍如何在队列中读取和处理消息。 

## <a name="prerequisites"></a>先决条件

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

接下来，请下载并安装适用于操作系统的 .NET Core 2.0。 如果运行的是 Windows，可以安装 Visual Studio 并根据偏好使用 .NET Framework。 也可选择安装一个可以在操作系统中使用的编辑器。

### <a name="windows"></a>Windows

- 安装 [.NET Core for Windows](https://www.microsoft.com/net/download/windows) 或 [.NET Framework](https://www.microsoft.com/net/download/windows)（Visual Studio for Windows 已随附）
- 安装 [Visual Studio for Windows](https://www.visualstudio.com/)。 如果使用的是 .NET Core，则可以根据需要安装 Visual Studio。  

有关在 .NET Core 与 .NET Framework 之间做出选择的信息，请参阅[为服务器应用选择 .NET Core 或 .NET Framework](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server)。

### <a name="linux"></a>Linux

- 安装[用于 Linux 的 .NET Core](https://www.microsoft.com/net/download/linux)
- （可选）安装 [Visual Studio Code](https://www.visualstudio.com/) 和 [C# 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp&dotnetid=963890049.1518206068)

### <a name="macos"></a>macOS

- 安装[用于 macOS 的 .NET Core](https://www.microsoft.com/net/download/macos)。
- （可选）安装[用于 Mac 的 Visual Studio](https://www.visualstudio.com/vs/visual-studio-mac/)

## <a name="download-the-sample-application"></a>下载示例应用程序

本快速入门中使用的示例应用程序是基本的控制台应用程序。 可以浏览 [GitHub](https://github.com/Azure-Samples/storage-queues-dotnet-quickstart) 上的示例应用程序。

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。 

```bash
git clone https://github.com/Azure-Samples/storage-queues-dotnet-quickstart.git
```

此命令会将存储库克隆到本地 git 文件夹。 若要打开 Visual Studio 解决方案，请找到 *storage-queues-dotnet-quickstart* 文件夹并将其打开，然后双击“storage-queues-dotnet-quickstart.sln”。 

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串

若要运行应用程序，必须为存储帐户提供连接字符串。 此示例应用程序从环境变量中读取连接字符串，并使用它对 Azure 存储请求进行授权。

复制连接字符串以后，请将其写入运行应用程序的本地计算机的新环境变量中。 若要设置环境变量，请打开控制台窗口，并遵照适用于操作系统的说明。 将 `<yourconnectionstring>` 替换为实际的连接字符串：

### <a name="windows"></a>Windows

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

添加环境变量后，可能需要重启任何正在运行的、需要读取环境变量的程序（包括控制台窗口）。 例如，如果使用 Visual Studio 作为编辑器，请在运行示例之前重启 Visual Studio。 

### <a name="linux"></a>Linux

```bash
export storageconnectionstring=<yourconnectionstring>
```

添加环境变量后，请从控制台窗口运行 `source ~/.bashrc`，使更改生效。

### <a name="macos"></a>macOS

编辑 .bash_profile，然后添加环境变量：

```bash
export STORAGE_CONNECTION_STRING=<yourconnectionstring>
```

添加环境变量后，请从控制台窗口运行 `source .bash_profile`，使更改生效。

## <a name="run-the-sample"></a>运行示例

该示例应用程序创建一个队列并在其中添加消息。 该应用程序首先扫视消息但不会将其从队列中删除，然后检索该消息并将其从队列中删除。

### <a name="windows"></a>Windows

如果使用 Visual Studio 作为编辑器，可以按 **F5** 运行应用程序。 

否则，请导航到应用程序目录，并使用 `dotnet run` 命令运行应用程序。

```
dotnet run
```

### <a name="linux"></a>Linux

导航到应用程序目录，使用 `dotnet run` 命令运行应用程序。

```
dotnet run
```

### <a name="macos"></a>macOS

导航到应用程序目录，使用 `dotnet run` 命令运行应用程序。

```
dotnet run
```

示例应用程序的输出类似于以下示例：

```
Azure Queues - .NET Quickstart sample

Created queue 'quickstartqueues-3136fe9a-fa52-4b19-a447-8999a847da52'

Added message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7' to queue 'quickstartqueues-3136fe9a-fa52-4b19-a447-8999a847da52'
Message insertion time: 2/7/2019 4:30:46 AM +00:00
Message expiration time: 2/14/2019 4:30:46 AM +00:00

Contents of peeked message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7': Hello, World

Message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7' becomes visible again at 2/7/2019 4:31:16 AM +00:00

Processed and deleted message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7'

Press any key to delete the sample queue.
```

## <a name="understand-the-sample-code"></a>了解示例代码

接下来介绍示例代码，探讨其工作方式。

### <a name="try-parsing-the-connection-string"></a>尝试分析连接字符串

该示例首先检查环境变量是否包含一个连接字符串，该字符串在经过分析后可以创建一个指向存储帐户的 [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) 对象。 该示例使用 [TryParse](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount.tryparse) 方法检查连接字符串是否有效。 如果 **TryParse** 成功，它会初始化 *storageAccount* 变量并返回 **true**。

```csharp
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable called storageconnectionstring, on the machine where the application is running.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with calls to Azure Queues here.
    ...    
}
else
{
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add an environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
}
```

### <a name="create-the-queue"></a>创建队列

首先，该示例创建一个队列并在其中添加消息。 

```csharp
// Create a queue called 'quickstartqueues' and append a GUID value so that the queue name 
// is unique in your storage account. 
queue = cloudQueueClient.GetQueueReference("quickstartqueues-" + Guid.NewGuid().ToString());
await queue.CreateAsync();

Console.WriteLine("Created queue '{0}'", queue.Name);
Console.WriteLine();
```

### <a name="add-a-message"></a>添加消息

接下来，该示例将消息添加到队列的后部。 

消息必须采用可包含在 XML 请求中的 UTF-8 编码格式，大小不能超过 64 KB。 如果消息包含二进制数据，则 Microsoft 建议对消息进行 Base64 编码。

消息的最大生存时间默认设置为 7 天。 可为消息生存时间指定任何正数，-1 表示该消息不会过期。

```csharp
// Create a message and add it to the queue. Set expiration time to 14 days.
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
await queue.AddMessageAsync(message, new TimeSpan(14,0,0,0), null, null, null);
Console.WriteLine("Added message '{0}' to queue '{1}'", message.Id, queue.Name);
Console.WriteLine("Message insertion time: {0}", message.InsertionTime.ToString());
Console.WriteLine("Message expiration time: {0}", message.ExpirationTime.ToString());
Console.WriteLine();
```

### <a name="peek-a-message-from-the-queue"></a>扫视队列中的消息

该示例演示如何扫视队列中的消息。 扫视消息时，可以读取该消息的内容。 但是，该消息仍对其他客户端可见，使其他客户端随后可以检索并处理该消息。

```csharp
// Peek at the message at the front of the queue. Peeking does not alter the message's 
// visibility, so that another client can still retrieve and process it. 
CloudQueueMessage peekedMessage = await queue.PeekMessageAsync();

// Display the ID and contents of the peeked message.
Console.WriteLine("Contents of peeked message '{0}': {1}", peekedMessage.Id, peekedMessage.AsString);
Console.WriteLine();
```

### <a name="dequeue-a-message"></a>取消消息排队

该示例还演示了如何取消消息的排队。 取消消息的排队时，会从队列的前部检索该消息，并使该消息暂时对其他客户端不可见。 默认情况下，消息将持续 30 秒不可见。 在此期间，代码可以处理该消息。 若要完成取消消息排队，请在处理后立即删除该消息，使其他客户端不会将同一条消息取消排队。

如果代码由于硬件或软件故障而无法处理消息，则不可见持续时间过后，该消息将再次可见。 其他客户端可以检索同一消息并重试。

```csharp
// Retrieve the message at the front of the queue. The message becomes invisible for 
// a specified interval, during which the client attempts to process it.
CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();

// Display the time at which the message will become visible again if it is not deleted.
Console.WriteLine("Message '{0}' becomes visible again at {1}", retrievedMessage.Id, retrievedMessage.NextVisibleTime);
Console.WriteLine();

//Process and delete the message within the period of invisibility.
await queue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Processed and deleted message '{0}'", retrievedMessage.Id);
Console.WriteLine();
```

### <a name="clean-up-resources"></a>清理资源

该示例通过删除队列来清理它所创建的资源。 删除队列也会删除该队列中包含的所有消息。

```csharp
Console.WriteLine("Press any key to delete the sample queue.");
Console.ReadLine();
Console.WriteLine("Deleting the queue and any messages it contains...");
Console.WriteLine();
if (queue != null)
{
    await queue.DeleteIfExistsAsync();
}
```

## <a name="resources-for-developing-net-applications-with-queues"></a>用于开发包含队列的 .NET 应用程序的资源

请参阅以下附加资源，了解如何开发包含 Azure 队列的 .NET 应用程序：

### <a name="binaries-and-source-code"></a>二进制文件和源代码

- 下载 Azure 存储的最新版 [.NET 客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)的 NuGet 包。 
- 查看 GitHub 上的 [.NET 客户端库源代码](https://github.com/Azure/azure-storage-net)。

### <a name="client-library-reference-and-samples"></a>客户端库参考和示例

- 参阅 [.NET API 参考](https://docs.microsoft.com/dotnet/api/overview/azure/storage)，详细了解 .NET 客户端库。
- 浏览使用 .NET 客户端库编写的[队列存储示例](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=queues)。

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何将消息添加到队列、扫视队列中的消息、取消消息排队，以及使用 .NET 处理消息。 

> [!div class="nextstepaction"]
> [在使用 Azure 队列存储的应用程序之间进行通信](https://docs.microsoft.com/learn/modules/communicate-between-apps-with-azure-queue-storage/index)

- 若要详细了解 .NET Core，请参阅 [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/)（.NET 10 分钟入门）。
