---
title: include 文件
description: include 文件
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 03/30/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: f0ff729084d194ff2e05e89eadc45782f775b1c5
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64929347"
---
本部分说明如何从 .NET 控制台应用以标记模板通知的形式发送突发新闻。 

1. 在 Visual Studio 中创建新的 Visual C# 控制台应用程序：a. 在菜单中，选择“文件” > “新建” > “项目”。
    b. 展开“Visual C#”，然后选择“Windows 桌面”。 
    c. 在模板列表中选择“控制台应用(.NET Framework)”。 
    d. 输入应用的**名称**。 
    e. 为应用选择**文件夹**。
    f. 选择“确定”创建该项目。 
2. 在 Visual Studio 主菜单中，选择“工具” > “NuGet 包管理器” > “包管理器控制台”，并在控制台窗口中输入以下字符串：
   
    ```
    Install-Package Microsoft.Azure.NotificationHubs
    ```
   
3. 按 **Enter**。  
    此操作会使用 [Microsoft.Azure.Notification Hubs NuGet 包]添加对 Azure 通知中心 SDK 的引用。

4. 打开文件 Program.cs 并添加以下 `using` 语句：
   
    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

5. 在 `Program` 类中，添加以下方法，或替换此方法（如果已存在）：
   
    ```csharp
    private static async void SendTemplateNotificationAsync()
    {
        // Define the notification hub.
        NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");

        // Create an array of breaking news categories.
        var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};

        // Send the notification as a template notification. All template registrations that contain
        // "messageParam" and the proper tags will receive the notifications.
        // This includes APNS, GCM, WNS, and MPNS template registrations.

        Dictionary<string, string> templateParams = new Dictionary<string, string>();

        foreach (var category in categories)
        {
            templateParams["messageParam"] = "Breaking " + category + " News!";
            await hub.SendTemplateNotificationAsync(templateParams, category);
        }
    }
    ```   
   
    此代码将针对字符串数组中的所有 6 个标记发送模板通知。 使用标记是为了确保设备仅接收已注册类别的通知。

5. 在前面的代码中，将 `<hub name>` 和 `<connection string with full access>` 占位符替换为通知中心名称和从通知中心仪表板获取的 *DefaultFullSharedAccessSignature* 的连接字符串。

6. 在 **Main** 方法中添加以下行：
   
    ```csharp
    SendTemplateNotificationAsync();
    Console.ReadLine();
    ```

7. 生成控制台应用。

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

<!-- URLs. -->
[Get started with Notification Hubs]: ../articles/notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Notification Hubs REST interface]: https://msdn.microsoft.com/library/windowsazure/dn223264.aspx
[Add push notifications for Mobile Apps]: ../articles/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-push.md
[How to use Notification Hubs from Java or PHP]: ../articles/notification-hubs/notification-hubs-java-push-notification-tutorial.md
[Microsoft.Azure.Notification Hubs NuGet 包]: http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/
