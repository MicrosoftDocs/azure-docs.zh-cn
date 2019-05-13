---
title: 使用 Azure 通知中心向 iOS 应用推送通知 | Microsoft Docs
description: 在本教程中，将了解如何使用 Azure 通知中心将推送通知发送到 iOS 应用程序。
services: notification-hubs
documentationcenter: ios
keywords: 推送通知, 推送通知, ios 推送通知
author: jwargo
manager: patniko
editor: spelluru
ms.assetid: b7fcd916-8db8-41a6-ae88-fc02d57cb914
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: tutorial
ms.custom: mvc
ms.date: 01/04/2019
ms.author: jowargo
ms.openlocfilehash: 2793f64528baa2b9a84c671fc73bbe399e2387ea
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65411618"
---
# <a name="tutorial-push-notifications-to-ios-apps-using-azure-notification-hubs"></a>教程：使用 Azure 通知中心将通知推送到 iOS 应用

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

在本教程中，你将使用 Azure 通知中心向 iOS 应用程序推送通知。 你将创建一个空白 iOS 应用，它使用 [Apple Push Notification 服务 (APNs)](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1) 接收推送通知。

在本教程中，你将执行以下步骤：

> [!div class="checklist"]
> * 生成证书签名请求文件
> * 请求你的应用推送通知
> * 为应用程序创建配置文件
> * 针对 iOS 推送通知配置通知中心
> * 将 iOS 应用连接到通知中心
> * 发送测试推送通知
> * 验证应用可以接收通知

可以 [在 GitHub 上](https://github.com/Azure/azure-notificationhubs-ios/tree/master/Samples)找到本教程的已完成代码。 

## <a name="prerequisites"></a>先决条件

* 有效的 Azure 帐户。 如果没有帐户，只需花费几分钟就能[创建一个免费 Azure 帐户](https://azure.microsoft.com/free)。
* [Windows Azure Messaging Framework]
* 最新版本的 [Xcode]
* 支持 iOS 10（或更高版本）的设备
* [Apple 开发人员计划](https://developer.apple.com/programs/) 成员身份。
  
  > [!NOTE]
  > 由于推送通知的配置要求，必须在物理 iOS 的设备（iPhone 或 iPad）而不是在 iOS 模拟器上部署和测试推送通知。
  
完成本教程是学习有关 iOS 应用的所有其他通知中心教程的先决条件。

[!INCLUDE [Notification Hubs Enable Apple Push Notifications](../../includes/notification-hubs-enable-apple-push-notifications.md)]

## <a name="configure-your-notification-hub-for-ios-push-notifications"></a>针对 iOS 推送通知配置通知中心

在本部分中，你将创建一个通知中心，并使用以前创建的 **.p12** 推送证书配置 APNS 身份验证。 如果想要使用已创建的通知中心，可以跳到步骤 5。

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-your-notification-hub-with-apns-information"></a>使用 APNS 信息配置通知中心

1. 在“通知服务”下选择“Apple (APNS)”。
2. 选择“证书”。
3. 选择**文件图标**。
4. 选择前面导出的 **.p12** 文件。
5. 指定正确的**密码**。
6. 选择“沙盒”模式。 仅当想要将推送通知发送给从应用商店购买应用的用户时，才使用“生产”模式。

    ![在 Azure 门户中配置 APNS 证书][7]

现在已为通知中心配置 APNS，并且有了用于注册应用和发送推送通知的连接字符串。

## <a name="connect-your-ios-app-to-notification-hubs"></a>将 iOS 应用连接到通知中心

1. 在 Xcode 中，创建新的 iOS 项目，并选择“单视图应用程序”模板。

    ![Xcode — 单一视图应用程序][8]

2. 设置新项目的选项时，请务必使用在 Apple 开发人员门户中设置捆绑标识符时使用的同一**产品名称**和**组织标识符**。

    ![Xcode — 项目选项][11]

3. 在“项目导航器”下单击项目名称，然后单击“常规”选项卡，找到“签名”。 确保为 Apple 开发人员帐户选择适当的团队。 XCode 会根据捆绑标识符自动下拉以前创建的预配配置文件。

    如果屏幕未显示在 Xcode 中创建的新预配配置文件，请尝试刷新签名标识的配置文件。 单击菜单栏上的“Xcode”，再依次单击“首选项”、“帐户”选项卡、“查看详细信息”按钮、签名标识，然后单击右下角的刷新按钮。

    ![Xcode — 预配配置文件][9]

4. 选择“功能”选项卡，确保启用“推送通知”

    ![Xcode - 推送功能][12]

5. 添加 Azure 通知中心 SDK 模块。

   可以使用 [Cocoapods](https://cocoapods.org) 或手动将二进制文件添加到项目中，从而将 Azure 通知中心 SDK 集成到应用中。

   - 通过 Cocoapods 集成

     将以下依赖项添加到 `podfile`，以便将 Azure 通知中心 SDK 包含到应用中。

     ```ruby
     pod 'AzureNotificationHubs-iOS'
     ```

     运行 `pod install` 以安装新定义的 Pod 并打开 `.xcworkspace`。

     > [!NOTE]
     > 如果在运行 `pod install` 时看到类似 ```[!] Unable to find a specification for `AzureNotificationHubs-iOS` ``` 的错误，请运行 `pod repo update` 从 Cocoapods 存储库获得最新的 Pod，然后运行 `pod install`。

   - 通过 Carthage 集成

     将以下依赖项添加到 `Cartfile`，以便将 Azure 通知中心 SDK 包含到应用中。

     ```ruby
     github "Azure/azure-notificationhubs-ios"
     ```

     接下来，更新并生成依赖项：

     ```shell
     $ carthage update
     ```

     有关使用 Carthage 的详细信息，请参阅 [Carthage GitHub 存储库](https://github.com/Carthage/Carthage)。

   - 通过将二进制文件复制到项目中进行集成

     1. 下载以 zip 文件形式提供的 [Azure 通知中心 SDK](https://github.com/Azure/azure-notificationhubs-ios/releases) 框架并将其解压缩。

     2. 在 Xcode 中，右键单击项目，然后单击“将文件添加到”选项，将 **WindowsAzureMessaging.framework** 文件夹添加到 Xcode 项目。 选择“选项”，确保选中“根据需要复制项目”，然后单击“添加”。

        ![解压缩 Azure SDK][10]

6. 将新的标头文件添加到名为 `HubInfo.h`的项目。 此文件保存着通知中心的常量。 添加以下定义，然后将字符串文本占位符替换为*中心名称*以及前面记下的 *DefaultListenSharedAccessSignature*。

    ```objc
    #ifndef HubInfo_h
    #define HubInfo_h

        #define HUBNAME @"<Enter the name of your hub>"
        #define HUBLISTENACCESS @"<Enter your DefaultListenSharedAccess connection string"

    #endif /* HubInfo_h */
    ```

7. 打开 `AppDelegate.h` 文件并添加以下导入指令：

    ```objc
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    #import <UserNotifications/UserNotifications.h>
    #import "HubInfo.h"
    ```
8. 根据 iOS 版本，在 `AppDelegate.m` 文件的 `didFinishLaunchingWithOptions` 方法中添加以下代码。 此代码将向 APNs 注册设备句柄：

    ```objc
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeSound |
        UIUserNotificationTypeAlert | UIUserNotificationTypeBadge categories:nil];

    [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
    [[UIApplication sharedApplication] registerForRemoteNotifications];
    ```

9. 在同一文件中添加以下方法：

    ```objc
        - (void) application:(UIApplication *) application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *) deviceToken {
        SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:HUBLISTENACCESS
                                    notificationHubPath:HUBNAME];

        [hub registerNativeWithDeviceToken:deviceToken tags:nil completion:^(NSError* error) {
            if (error != nil) {
                NSLog(@"Error registering for notifications: %@", error);
            }
            else {
                [self MessageBox:@"Registration Status" message:@"Registered"];
            }
        }];
        }

    -(void)MessageBox:(NSString *) title message:(NSString *)messageText
    {
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:title message:messageText preferredStyle:UIAlertControllerStyleAlert];
        UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil];
        [alert addAction:okAction];
        [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alert animated:YES completion:nil];
    }
    ```

    此代码将使用在 HubInfo.h 中指定的连接信息连接到通知中心。 然后，它向通知中心提供设备令牌，使通知中心能够发送通知。

10. 在同一文件中，添加以下方法以便在应用处于活动状态时收到通知的情况下显示 **UIAlert** ：

    ```objc
    - (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
        NSLog(@"%@", userInfo);
        [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
    }
    ```

11. 若要验证是否没有故障，请在设备上生成并运行应用。

## <a name="send-test-push-notifications"></a>发送测试推送通知

可以在 [Azure 门户]中使用“测试性发送”选项，在应用中测试通知的发送。 它会向设备发送测试性的推送通知。

![Azure 门户 - 测试性发送][30]

[!INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

## <a name="verify-that-your-app-receives-push-notifications"></a>验证应用可以接收推送通知

要在 iOS 上测试推送通知，必须将应用部署到物理 iOS 设备。 不能使用 iOS 模拟器发送 Apple 推送通知。

1. 运行应用并验证注册是否成功，并按“确定”。

    ![iOS 应用推送通知注册测试][33]

2. 如上一部分所述，接下来可以从 [Azure 门户]发送测试推送通知。

3. 该推送通知会从特定通知中心发送到所有已注册为接收通知的设备。

    ![iOS 应用推送通知接收测试][35]

## <a name="next-steps"></a>后续步骤

在这个简单的示例中，已将推送通知广播到所有已注册的 iOS 设备。 若要了解如何向特定的 iOS 设备推送通知，请转到以下教程：

> [!div class="nextstepaction"]
>[向特定设备推送通知](notification-hubs-ios-xplat-segmented-apns-push-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-ios-get-started/notification-hubs-apple-config.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-apple-config-cert.png
[8]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app.png
[9]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app2.png
[10]: ./media/notification-hubs-ios-get-started/notification-hubs-create-ios-app3.png
[11]: ./media/notification-hubs-ios-get-started/notification-hubs-xcode-product-name.png
[12]: ./media/notification-hubs-ios-get-started/notification-hubs-enable-push.png
[30]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[31]: ./media/notification-hubs-ios-get-started/notification-hubs-ios-ui.png
[32]: ./media/notification-hubs-ios-get-started/notification-hubs-storyboard-view.png
[33]: ./media/notification-hubs-ios-get-started/notification-hubs-test1.png
[35]: ./media/notification-hubs-ios-get-started/notification-hubs-test3.png

<!-- URLs. -->
[Windows Azure Messaging Framework]: https://go.microsoft.com/fwlink/?LinkID=799698&clcid=0x409
[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/?LinkID=266533
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-ios
[Notification Hubs Guidance]: https://msdn.microsoft.com/library/jj927170.aspx
[Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: https://go.microsoft.com/fwlink/p/?LinkId=272456
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-ios-get-started-push.md
[Azure Notification Hubs Notify Users for iOS with .NET backend]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-ios-xplat-segmented-apns-push-notification.md
[Local and Push Notification Programming Guide]: https://developer.apple.com/library/mac/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW1
[Azure 门户]: https://portal.azure.com
