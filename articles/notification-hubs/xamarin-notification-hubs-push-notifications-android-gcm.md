---
title: 使用 Azure 通知中心向 Xamarin.Android 应用推送通知 | Microsoft Docs
description: 在本教程中，将了解如何使用 Azure 通知中心将推送通知发送到 Xamarin.Android 应用程序。
author: jwargo
manager: patniko
editor: spelluru
services: notification-hubs
documentationcenter: xamarin
ms.assetid: 0be600fe-d5f3-43a5-9e5e-3135c9743e54
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 05/01/2019
ms.author: jowargo
ms.openlocfilehash: 00e62226ee7e2b912a909cfa32a25e4562b99e83
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203746"
---
# <a name="tutorial-push-notifications-to-xamarinandroid-apps-using-azure-notification-hubs"></a>教程：使用 Azure 通知中心向 Xamarin.Android 应用推送通知

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>概述

本教程说明如何使用 Azure 通知中心将推送通知发送到 Xamarin.Android 应用程序。 创建一个空白 Xamarin.Android 应用，以便使用 Firebase Cloud Messaging (FCM) 接收推送通知。 使用通知中心将推送通知广播到运行应用的所有设备。 [NotificationHubs 应用](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/Xamarin/GetStartedXamarinAndroid)示例中提供了完成的代码。

在本教程中，我们将执行以下步骤：

> [!div class="checklist"]
> * 创建 Firebase 项目并启用 Firebase Cloud Messaging
> * 创建通知中心
> * 创建 Xamarin.Android 应用并将其连接到通知中心
> * 从 Azure 门户发送测试通知

## <a name="prerequisites"></a>先决条件

* **Azure 订阅**。 如果还没有 Azure 订阅，可以在开始前[创建一个免费 Azure 帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* 基于 Windows 的 [Visual Studio with Xamarin] 或基于 OS X 的 [Visual Studio for Mac]。
* 有效的 Google 帐户

## <a name="create-a-firebase-project-and-enable-firebase-cloud-messaging"></a>创建 Firebase 项目并启用 Firebase Cloud Messaging

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging-xamarin.md)]

## <a name="create-a-notification-hub"></a>创建通知中心

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcm-settings-for-the-notification-hub"></a>配置通知中心的 GCM 设置

1. 在“通知设置”部分选择“Google (GCM)”。
2. 输入先前从 Google Firebase Console 记下的**旧服务器密钥**。
3. 在工具栏上选择“保存”。

    ![](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

通知中心已配置为使用 FCM，并且你有连接字符串，既可用于注册应用以接收通知，又可用于发送推送通知。

## <a name="create-a-xamarinandroid-app-and-connect-it-to-notification-hub"></a>创建 Xamarin.Android 应用并将其连接到通知中心

### <a name="create-visual-studio-project-and-add-nuget-packages"></a>创建 Visual Studio 项目并添加 NuGet 包

1. 在 Visual Studio 中打开“文件”菜单，选择“新建”，然后选择“项目”。 在“新建项目”窗口中执行以下步骤： 
    1. 展开“已安装”、“Visual C#”，然后单击“Android”。
    2. 从列表中选择“Android 应用(Xamarin)”。 
    3. 输入项目的**名称**。 
    4. 选择项目的**位置**。 
    5. 选择“确定” 

        ![“新建项目”对话框](./media/partner-xamarin-notification-hubs-android-get-started/new-project-dialog-new.png)        
2. 在“新建 Android 应用”对话框中，选择“空白应用”，然后选择“确定”。 

    ![“新建项目”对话框](./media/partner-xamarin-notification-hubs-android-get-started/new-android-app-dialog.png)
1. 在“解决方案资源管理器”窗口中展开“属性”，然后单击“AndroidManifest.xml”。 更新包名，使之与你在 Google Firebase Console 中将 Firebase Cloud Messaging 添加到项目时输入的包名匹配。

    ![GCM 中的包名](./media/partner-xamarin-notification-hubs-android-get-started/package-name-gcm.png)
3. 右键单击项目，然后选择“管理 NuGet 包...”。
4. 选择“浏览”按钮。搜索 **Xamarin.GooglePlayServices.Base**。 在结果列表中选择 **Xamarin.GooglePlayServices.Base**。 然后，选择“安装”。

    ![Google Play Services NuGet](./media/partner-xamarin-notification-hubs-android-get-started/google-play-services-nuget.png)
5. 在“NuGet 包管理器”窗口中，搜索“Xamarin.Firebase.Messaging”。 在结果列表中选择 **Xamarin.Firebase.Messaging**。 然后，选择“安装”。
6. 现在，请搜索 **Xamarin.Azure.NotificationHubs.Android**。 在结果列表中选择 **Xamarin.Azure.NotificationHubs.Android**。 然后，选择“安装”。

### <a name="add-the-google-services-json-file"></a>添加 Google Services JSON 文件

1. 将从 Google Firebase Console 下载的 `google-services.json` 文件复制到项目文件夹。
2. 将 `google-services.json` 添加到项目。
3. 在“解决方案资源管理器”窗口中选择 `google-services.json`。
4. 在“属性”窗格中，将“生成操作”设置为“GoogleServicesJson”。 如果看不到 **GoogleServicesJson**，请在关闭 Visual Studio 后将其重新启动，重新打开项目，然后重试。

    ![GoogleServicesJson 生成操作](./media/partner-xamarin-notification-hubs-android-get-started/google-services-json-build-action.png)

### <a name="set-up-notification-hubs-in-your-project"></a>在项目中设置通知中心

#### <a name="registering-with-firebase-cloud-messaging"></a>注册到 Firebase Cloud Messaging

1. 打开 `AndroidManifest.xml` 文件，并将以下 `<receiver>` 元素插入 `<application>` 元素：

    ```xml
    <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" android:exported="false" />
    <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:exported="true" android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
        <category android:name="${applicationId}" />
        </intent-filter>
    </receiver>
    ```
2. 将以下语句添加到 **application** 元素之前。 

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    ```
1. 收集有关 Android 应用和通知中心的以下信息：

   * **侦听连接字符串**：在 [Azure 门户]中的仪表板上，选择“查看连接字符串”。 复制此值的 `DefaultListenSharedAccessSignature` 连接字符串。
   * **中心名称**：中心在 [Azure 门户]中的名称。 例如 *mynotificationhub2*。
3. 在“解决方案资源管理器”窗口中右键单击你的**项目**，选择“添加”，然后选择“类”。
4. 为 Xamarin 项目创建 `Constants.cs` 类，并在该类中定义以下常量值。 将占位符替换成自己的值。

    ```csharp
    public static class Constants
    {
        public const string ListenConnectionString = "<Listen connection string>";
        public const string NotificationHubName = "<hub name>";
    }
    ```
5. 将以下 using 语句添加到 `MainActivity.cs`：

    ```csharp
    using Android.Util;
    using Android.Gms.Common;
    ```
6. 将以下属性添加到 MainActivity 类。 TAG 变量将用于在运行应用时显示警报对话框：

    ```csharp
    public const string TAG = "MainActivity";
    internal static readonly string CHANNEL_ID = "my_notification_channel";
    ```
7. 向 MainActivity 类添加以下方法。 它会检查 **Google Play Services** 在设备上是否可用。 

    ```csharp
    public bool IsPlayServicesAvailable()
    {
        int resultCode = GoogleApiAvailability.Instance.IsGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.Success)
        {
            if (GoogleApiAvailability.Instance.IsUserResolvableError(resultCode))
                Log.Debug(TAG, GoogleApiAvailability.Instance.GetErrorString(resultCode));
            else
            {
                Log.Debug(TAG, "This device is not supported");
                Finish();
            }
            return false;
        }
     
        Log.Debug(TAG, "Google Play Services is available.");
        return true;
    }
    ```
1. 向用于创建通知通道的 MainActivity 类添加以下方法。

    ```csharp
    private void CreateNotificationChannel()
    {
        if (Build.VERSION.SdkInt < BuildVersionCodes.O)
        {
            // Notification channels are new in API 26 (and not a part of the
            // support library). There is no need to create a notification
            // channel on older versions of Android.
            return;
        }
     
        var channelName = CHANNEL_ID;
        var channelDescription = string.Empty;
        var channel = new NotificationChannel(CHANNEL_ID, channelName, NotificationImportance.Default)
        {
            Description = channelDescription
        };
     
        var notificationManager = (NotificationManager)GetSystemService(NotificationService);
        notificationManager.CreateNotificationChannel(channel);
    }
    ```
1. 在 `MainActivity.cs` 中将以下代码添加到 `base.OnCreate(savedInstanceState)` 后面的 `OnCreate`：

    ```csharp
    if (Intent.Extras != null)
    {
        foreach (var key in Intent.Extras.KeySet())
        {
            if(key!=null)
            {
                var value = Intent.Extras.GetString(key);
                Log.Debug(TAG, "Key: {0} Value: {1}", key, value);
            }
        }
    }
    
    IsPlayServicesAvailable();
    CreateNotificationChannel();
    ```
8. 创建新类 `MyFirebaseIIDService`（就像创建 `Constants` 类一样）。
9. 将以下 using 语句添加到 `MyFirebaseIIDService.cs`：

    ```csharp
    using Android.Util;
    using WindowsAzure.Messaging;
    using Firebase.Iid;
    ```

10. 在 `MyFirebaseIIDService.cs` 中添加以下 `class` 声明，让类从 `FirebaseInstanceIdService` 继承：

    ```csharp
    [Service]
    [IntentFilter(new[] { "com.google.firebase.INSTANCE_ID_EVENT" })]
    public class MyFirebaseIIDService : FirebaseInstanceIdService
    ```
11. 在 `MyFirebaseIIDService.cs` 中添加以下代码：

    ```csharp
    const string TAG = "MyFirebaseIIDService";
    NotificationHub hub;

    public override void OnTokenRefresh()
    {
        var refreshedToken = FirebaseInstanceId.Instance.Token;
        Log.Debug(TAG, "FCM token: " + refreshedToken);
        SendRegistrationToServer(refreshedToken);
    }

    void SendRegistrationToServer(string token)
    {
        // Register with Notification Hubs
        hub = new NotificationHub(Constants.NotificationHubName,
                                    Constants.ListenConnectionString, this);

        var tags = new List<string>() { };
        var regID = hub.Register(token, tags.ToArray()).RegistrationId;

        Log.Debug(TAG, $"Successful registration of ID {regID}");
    }
    ```
12. 为项目创建另一新类，将其命名为 `MyFirebaseMessagingService`。
13. 将以下 using 语句添加到 `MyFirebaseMessagingService.cs`。

    ```csharp
    using Android.Util;
    using Firebase.Messaging;
    using Android.Support.V4.App;
    using Build = Android.OS.Build;
    ```
14. 将以下内容添加到类声明上方，让类从 `FirebaseMessagingService` 继承：

    ```csharp
    [Service]
    [IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
    public class MyFirebaseMessagingService : FirebaseMessagingService
    ```
15. 将以下代码添加到 `MyFirebaseMessagingService.cs`：

    ```csharp
    const string TAG = "MyFirebaseMsgService";
    public override void OnMessageReceived(RemoteMessage message)
    {
        Log.Debug(TAG, "From: " + message.From);
        if(message.GetNotification()!= null)
        {
            //These is how most messages will be received
            Log.Debug(TAG, "Notification Message Body: " + message.GetNotification().Body);
            SendNotification(message.GetNotification().Body);
        }
        else
        {
            //Only used for debugging payloads sent from the Azure portal
            SendNotification(message.Data.Values.First());

        }
    }

    void SendNotification(string messageBody)
    {
        var intent = new Intent(this, typeof(MainActivity));
        intent.AddFlags(ActivityFlags.ClearTop);
        var pendingIntent = PendingIntent.GetActivity(this, 0, intent, PendingIntentFlags.OneShot);

        var notificationBuilder = new NotificationCompat.Builder(this)
                    .SetContentTitle("FCM Message")
                    .SetSmallIcon(Resource.Drawable.ic_launcher)
                    .SetContentText(messageBody)
                    .SetAutoCancel(true)
                    .SetShowWhen(false)
                    .SetContentIntent(pendingIntent);

        if (Build.VERSION.SdkInt >= BuildVersionCodes.O)
        {
            notificationBuilder.SetChannelId(MainActivity.CHANNEL_ID);
        }

        var notificationManager = NotificationManager.FromContext(this);

        notificationManager.Notify(0, notificationBuilder.Build());
    }
    ```
16. **生成**项目。
17. 在设备或加载的模拟器上**运行**应用

## <a name="send-test-notification-from-the-azure-portal"></a>从 Azure 门户发送测试通知

可以在 [Azure 门户]中使用“测试性发送”选项，在应用中测试通知的发送。 它会向设备发送测试性的推送通知。

![Azure 门户 - 测试性发送](media/partner-xamarin-notification-hubs-android-get-started/send-test-notification.png)

通常，推送通知是在移动服务或 ASP.NET 等后端服务中通过兼容库发送的。 如果后端没有可用库，也可使用 REST API 直接发送通知消息。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何将广播通知发送到所有注册到后端的 Android 设备。 若要了解如何向特定的 Android 设备推送通知，请转到以下教程：

> [!div class="nextstepaction"]
>[向特定设备推送通知](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md)

<!-- Anchors. -->
[Enable Google Cloud Messaging]: #register
[Configure your Notification Hub]: #configure-hub
[Connecting your app to the Notification Hub]: #connecting-app
[Run your app with the emulator]: #run-app
[Send notifications from your back-end]: #send
[Next steps]:#next-steps

<!-- Images. -->
[11]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-configure-android.png
[13]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app1.png
[15]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app3.png
[18]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app7.png
[19]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app8.png
[20]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-android-toast.png
[22]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-project1.png
[23]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-project2.png
[24]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-xamarin-android-app-options.png
[25]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-google-services-json.png
[30]: ./media/notification-hubs-android-get-started/notification-hubs-test-send.png

<!-- URLs. -->
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-xamarin-android/#create-new-service
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js
[Visual Studio with Xamarin]: https://docs.microsoft.com/visualstudio/install/install-visual-studio
[Visual Studio for Mac]: https://www.visualstudio.com/vs/visual-studio-mac/
[Azure 门户]: https://portal.azure.com/
[wns object]: https://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: https://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Android]: https://msdn.microsoft.com/library/dn282661.aspx
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[GitHub]: https://github.com/Azure/azure-notificationhubs-android
