---
title: 使用 Azure 通知中心向特定的 Android 应用程序用户推送通知 | Microsoft Docs
description: 了解如何使用 Azure 通知中心向特定用户推送通知。
documentationcenter: android
services: notification-hubs
author: jwargo
manager: patniko
editor: spelluru
ms.assetid: ae0e17a8-9d2b-496e-afd2-baa151370c25
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc
ms.date: 05/01/2019
ms.author: jowargo
ms.openlocfilehash: 86a2cd824d1896211efd40bb8aa1d007149ef2db
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65203581"
---
# <a name="tutorial-push-notification-to-specific-android-application-users-by-using-azure-notification-hubs"></a>教程：使用 Azure 通知中心向特定的 Android 应用程序用户推送通知

[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

本教程说明如何使用 Azure 通知中心将推送通知发送到特定设备上的特定应用程序用户。 ASP.NET WebAPI 后端用于对客户端进行身份验证并生成通知，如指南文章[从应用后端注册](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend)中所述。 本教程基于在[教程：使用 Azure 通知中心和 Firebase Cloud Messaging 将通知推送到 Android 设备](notification-hubs-android-push-notification-google-fcm-get-started.md)。

在本教程中，我们将执行以下步骤：

> [!div class="checklist"]
> * 创建用于用户身份验证的后端 Web API 项目。  
> * 更新 Android 应用程序。
> * 测试应用程序

## <a name="prerequisites"></a>先决条件

在开始本教程之前完成[教程：使用 Azure 通知中心和 Firebase Cloud Messaging 将通知推送到 Android 设备](notification-hubs-android-push-notification-google-fcm-get-started.md)。

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="create-the-android-project"></a>创建 Android 项目

下一步是更新在[教程：使用 Azure 通知中心和 Firebase Cloud Messaging 将通知推送到 Android 设备](notification-hubs-android-push-notification-google-fcm-get-started.md)。

1. 打开 `res/layout/activity_main.xml` 文件，替换以下内容定义：

    它会添加新的 EditText 控件，方便你以用户身份登录。 此外还会添加一个字段用作要发送的通知的一部分的用户名标记：

    ```xml
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/usernameText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/usernameHint"
        android:layout_above="@+id/passwordText"
        android:layout_alignParentEnd="true" />
    <EditText
        android:id="@+id/passwordText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/passwordHint"
        android:inputType="textPassword"
        android:layout_above="@+id/buttonLogin"
        android:layout_alignParentEnd="true" />
    <Button
        android:id="@+id/buttonLogin"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/loginButton"
        android:onClick="login"
        android:layout_above="@+id/toggleButtonFCM"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="24dp" />
    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textOn="WNS on"
        android:textOff="WNS off"
        android:id="@+id/toggleButtonWNS"
        android:layout_toLeftOf="@id/toggleButtonFCM"
        android:layout_centerVertical="true" />
    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textOn="FCM on"
        android:textOff="FCM off"
        android:id="@+id/toggleButtonFCM"
        android:checked="true"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true" />
    <ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textOn="APNS on"
        android:textOff="APNS off"
        android:id="@+id/toggleButtonAPNS"
        android:layout_toRightOf="@id/toggleButtonFCM"
        android:layout_centerVertical="true" />
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessageTag"
        android:layout_below="@id/toggleButtonFCM"
        android:layout_centerHorizontal="true"
        android:hint="@string/notification_message_tag_hint" />
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessage"
        android:layout_below="@+id/editTextNotificationMessageTag"
        android:layout_centerHorizontal="true"
        android:hint="@string/notification_message_hint" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/send_button"
        android:id="@+id/sendbutton"
        android:onClick="sendNotificationButtonOnClick"
        android:layout_below="@+id/editTextNotificationMessage"
        android:layout_centerHorizontal="true" />
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:id="@+id/text_hello"
        />
    </RelativeLayout>
    ```
2. 打开 `res/values/strings.xml` 文件并将 `send_button` 定义替换为以下行，以重新定义 `send_button` 的字符串并为其他控件添加字符串：

    ```xml
    <string name="usernameHint">Username</string>
    <string name="passwordHint">Password</string>
    <string name="loginButton">1. Sign in</string>
    <string name="send_button">2. Send Notification</string>
    <string name="notification_message_hint">Notification message</string>
    <string name="notification_message_tag_hint">Recipient username</string>
    ```

    `main_activity.xml` 的图形布局现在应如下图所示：

    ![][A1]
3. 在 `MainActivity` 类所在的包中创建一个名为 `RegisterClient` 的新类。 将以下代码用于新的类文件。

    ```java
  
    import java.io.IOException;
    import java.io.UnsupportedEncodingException;
    import java.util.Set;
    
    import org.apache.http.HttpResponse;
    import org.apache.http.HttpStatus;
    import org.apache.http.client.ClientProtocolException;
    import org.apache.http.client.HttpClient;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.client.methods.HttpPut;
    import org.apache.http.client.methods.HttpUriRequest;
    import org.apache.http.entity.StringEntity;
    import org.apache.http.impl.client.DefaultHttpClient;
    import org.apache.http.util.EntityUtils;
    import org.json.JSONArray;
    import org.json.JSONException;
    import org.json.JSONObject;
    
    import android.content.Context;
    import android.content.SharedPreferences;
    import android.util.Log;
    
    public class RegisterClient {
        private static final String PREFS_NAME = "ANHSettings";
        private static final String REGID_SETTING_NAME = "ANHRegistrationId";
        private String Backend_Endpoint;
        SharedPreferences settings;
        protected HttpClient httpClient;
        private String authorizationHeader;
    
        public RegisterClient(Context context, String backendEndpoint) {
            super();
            this.settings = context.getSharedPreferences(PREFS_NAME, 0);
            httpClient =  new DefaultHttpClient();
            Backend_Endpoint = backendEndpoint + "/api/register";
        }
    
        public String getAuthorizationHeader() {
            return authorizationHeader;
        }
    
        public void setAuthorizationHeader(String authorizationHeader) {
            this.authorizationHeader = authorizationHeader;
        }
    
        public void register(String handle, Set<String> tags) throws ClientProtocolException, IOException, JSONException {
            String registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
    
            JSONObject deviceInfo = new JSONObject();
            deviceInfo.put("Platform", "fcm");
            deviceInfo.put("Handle", handle);
            deviceInfo.put("Tags", new JSONArray(tags));
    
            int statusCode = upsertRegistration(registrationId, deviceInfo);
    
            if (statusCode == HttpStatus.SC_OK) {
                return;
            } else if (statusCode == HttpStatus.SC_GONE){
                settings.edit().remove(REGID_SETTING_NAME).commit();
                registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
                statusCode = upsertRegistration(registrationId, deviceInfo);
                if (statusCode != HttpStatus.SC_OK) {
                    Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                    throw new RuntimeException("Error upserting registration");
                }
            } else {
                Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                throw new RuntimeException("Error upserting registration");
            }
        }
    
        private int upsertRegistration(String registrationId, JSONObject deviceInfo)
                throws UnsupportedEncodingException, IOException,
                ClientProtocolException {
            HttpPut request = new HttpPut(Backend_Endpoint+"/"+registrationId);
            request.setEntity(new StringEntity(deviceInfo.toString()));
            request.addHeader("Authorization", "Basic "+authorizationHeader);
            request.addHeader("Content-Type", "application/json");
            HttpResponse response = httpClient.execute(request);
            int statusCode = response.getStatusLine().getStatusCode();
            return statusCode;
        }
    
        private String retrieveRegistrationIdOrRequestNewOne(String handle) throws ClientProtocolException, IOException {
            if (settings.contains(REGID_SETTING_NAME))
                return settings.getString(REGID_SETTING_NAME, null);
    
            HttpUriRequest request = new HttpPost(Backend_Endpoint+"?handle="+handle);
            request.addHeader("Authorization", "Basic "+authorizationHeader);
            HttpResponse response = httpClient.execute(request);
            if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                Log.e("RegisterClient", "Error creating registrationId: " + response.getStatusLine().getStatusCode());
                throw new RuntimeException("Error creating Notification Hubs registrationId");
            }
            String registrationId = EntityUtils.toString(response.getEntity());
            registrationId = registrationId.substring(1, registrationId.length()-1);
    
            settings.edit().putString(REGID_SETTING_NAME, registrationId).commit();
    
            return registrationId;
        }
    }
    ```

    此组件将实现所需的 REST 调用，以便能够联系应用后端来注册推送通知。 它还会在本地存储通知中心创建的 *registrationIds*，如[从应用后端注册](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend)中所述。 它使用你在单击“登录”按钮时存储在本地存储中的授权令牌。
4. 在 `MainActivity` 类中，添加一个用于 `RegisterClient` 类的字段和一个用于 ASP.NET 后端终结点的字符串。 确保使用前面获取的实际后端终结点来替换 `<Enter Your Backend Endpoint>`。 例如，`http://mybackend.azurewebsites.net`。

    ```java
    private RegisterClient registerClient;
    private static final String BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";
    FirebaseInstanceId fcm;
    String FCM_token = null;
    ```

5. 在 `MainActivity` 类的 `onCreate` 方法中，删除或注释掉 `hub` 字段的初始化以及对 `registerWithNotificationHubs` 方法的调用。 然后，添加代码来初始化 `RegisterClient` 类的实例。 该方法应包含以下行：

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mainActivity = this;
        MyHandler.createChannelAndHandleNotifications(getApplicationContext());
        fcm = FirebaseInstanceId.getInstance();
        registerClient = new RegisterClient(this, BACKEND_ENDPOINT);
        setContentView(R.layout.activity_main);
    }
    ```
7. 向 `MainActivity.java` 文件添加以下 `import` 语句。

    ```java
    import android.util.Base64;
    import android.view.View;
    import android.widget.EditText;
    
    import android.widget.Button;
    import android.widget.ToggleButton;
    import java.io.UnsupportedEncodingException;
    import android.content.Context;
    import java.util.HashSet;
    import android.widget.Toast;
    import org.apache.http.client.ClientProtocolException;
    import java.io.IOException;
    import org.apache.http.HttpStatus;
    
    import android.os.AsyncTask;
    import org.apache.http.HttpResponse;
    import org.apache.http.client.methods.HttpPost;
    import org.apache.http.entity.StringEntity;
    import org.apache.http.impl.client.DefaultHttpClient;
    
    import android.app.AlertDialog;
    import android.content.DialogInterface;
    
    import com.google.firebase.iid.FirebaseInstanceId;
    import com.google.firebase.iid.InstanceIdResult;
    import com.google.android.gms.tasks.OnSuccessListener;
    import java.util.concurrent.TimeUnit;
    ```
8. 将 onStart 方法的代码替换为以下代码：

    ```java
    super.onStart();
    Button sendPush = (Button) findViewById(R.id.sendbutton);
    sendPush.setEnabled(false);
    ```
9. 然后，添加以下方法，处理“登录”按钮的单击事件，并发送推送通知。

    ```java
    public void login(View view) throws UnsupportedEncodingException {
        this.registerClient.setAuthorizationHeader(getAuthorizationHeader());

        final Context context = this;
        new AsyncTask<Object, Object, Object>() {
            @Override
            protected Object doInBackground(Object... params) {
                try {

                    FirebaseInstanceId.getInstance().getInstanceId().addOnSuccessListener(new OnSuccessListener<InstanceIdResult>() {
                        @Override
                        public void onSuccess(InstanceIdResult instanceIdResult) {
                            FCM_token = instanceIdResult.getToken();
                            Log.d(TAG, "FCM Registration Token: " + FCM_token);
                        }
                    });
                    TimeUnit.SECONDS.sleep(1);
                    registerClient.register(FCM_token, new HashSet<String>());
                } catch (Exception e) {
                    DialogNotify("MainActivity - Failed to register", e.getMessage());
                    return e;
                }
                return null;
            }

            protected void onPostExecute(Object result) {
                Button sendPush = (Button) findViewById(R.id.sendbutton);
                sendPush.setEnabled(true);
                Toast.makeText(context, "Signed in and registered.",
                        Toast.LENGTH_LONG).show();
            }
        }.execute(null, null, null);
    }

    private String getAuthorizationHeader() throws UnsupportedEncodingException {
        EditText username = (EditText) findViewById(R.id.usernameText);
        EditText password = (EditText) findViewById(R.id.passwordText);
        String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
        basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);
        return basicAuthHeader;
    }

    /**
        * This method calls the ASP.NET WebAPI backend to send the notification message
        * to the platform notification service based on the pns parameter.
        *
        * @param pns     The platform notification service to send the notification message to. Must
        *                be one of the following ("wns", "fcm", "apns").
        * @param userTag The tag for the user who will receive the notification message. This string
        *                must not contain spaces or special characters.
        * @param message The notification message string. This string must include the double quotes
        *                to be used as JSON content.
        */
    public void sendPush(final String pns, final String userTag, final String message)
            throws ClientProtocolException, IOException {
        new AsyncTask<Object, Object, Object>() {
            @Override
            protected Object doInBackground(Object... params) {
                try {

                    String uri = BACKEND_ENDPOINT + "/api/notifications";
                    uri += "?pns=" + pns;
                    uri += "&to_tag=" + userTag;

                    HttpPost request = new HttpPost(uri);
                    request.addHeader("Authorization", "Basic "+ getAuthorizationHeader());
                    request.setEntity(new StringEntity(message));
                    request.addHeader("Content-Type", "application/json");

                    HttpResponse response = new DefaultHttpClient().execute(request);

                    if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                        DialogNotify("MainActivity - Error sending " + pns + " notification",
                                response.getStatusLine().toString());
                        throw new RuntimeException("Error sending notification");
                    }
                } catch (Exception e) {
                    DialogNotify("MainActivity - Failed to send " + pns + " notification ", e.getMessage());
                    return e;
                }

                return null;
            }
        }.execute(null, null, null);
    }
    ```

    “登录”按钮的 `login` 处理程序生成在输入的用户名和密码上使用的基本身份验证令牌（代表身份验证方案使用的任何令牌），然后使用 `RegisterClient` 调用后端来注册。

    `sendPush` 方法调用后端来触发根据用户标记向用户发送安全通知。 `sendPush` 针对的平台通知服务取决于传入的 `pns` 字符串。

10. 将下面的 `DialogNotify` 方法添加到 `MainActivity` 类。

    ```java
    protected void DialogNotify(String title, String message)
    {
        AlertDialog alertDialog = new AlertDialog.Builder(MainActivity.this).create();
        alertDialog.setTitle(title);
        alertDialog.setMessage(message);
        alertDialog.setButton(AlertDialog.BUTTON_NEUTRAL, "OK",
                new DialogInterface.OnClickListener() {
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });
        alertDialog.show();
    }
    ```
11. 在 `MainActivity` 类中，更新 `sendNotificationButtonOnClick` 方法以使用用户选择的平台通知服务调用 `sendPush` 方法，如下所示。

    ```java
    /**
    * Send Notification button click handler. This method sends the push notification
    * message to each platform selected.
    *
    * @param v The view
    */
    public void sendNotificationButtonOnClick(View v)
            throws ClientProtocolException, IOException {

        String nhMessageTag = ((EditText) findViewById(R.id.editTextNotificationMessageTag))
                .getText().toString();
        String nhMessage = ((EditText) findViewById(R.id.editTextNotificationMessage))
                .getText().toString();

        // JSON String
        nhMessage = "\"" + nhMessage + "\"";

        if (((ToggleButton)findViewById(R.id.toggleButtonWNS)).isChecked())
        {
            sendPush("wns", nhMessageTag, nhMessage);
        }
        if (((ToggleButton)findViewById(R.id.toggleButtonFCM)).isChecked())
        {
            sendPush("fcm", nhMessageTag, nhMessage);
        }
        if (((ToggleButton)findViewById(R.id.toggleButtonAPNS)).isChecked())
        {
            sendPush("apns", nhMessageTag, nhMessage);
        }
    }
    ```
12. 在 `build.gradle` 文件中，将以下行添加到 `buildTypes` 节后的 `android` 节。

    ```java
    useLibrary 'org.apache.http.legacy'
    ```
13. 生成项目。

## <a name="test-the-app"></a>测试应用程序

1. 在设备或模拟器上使用 Android Studio 运行该应用程序。
2. 在 Android 应用中，输入用户名和密码。 它们必须都是相同的字符串值，并且不能包含空格或特殊字符。
3. 在 Android 应用中，单击“登录”。 等待指示“已登录并已注册”的 toast 消息。 它启用“发送通知”按钮。

    ![][A2]
4. 单击切换按钮，以启用已在其中运行该应用并已注册用户的所有平台。
5. 输入接收通知消息的用户的名称。 必须已在目标设备上为通知注册该用户。
6. 为用户输入要接收为推送通知的消息。
7. 单击“发送通知”。  已使用匹配的用户名标记注册的每个设备都会收到该推送通知。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何向其标记与注册相关联的特定用户推送通知。 若要了解如何推送基于位置的通知，请转到以下教程：

> [!div class="nextstepaction"]
>[推送基于位置的通知](notification-hubs-push-bing-spatial-data-geofencing-notification.md)

[A1]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users.png
[A2]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users-enter-password.png
