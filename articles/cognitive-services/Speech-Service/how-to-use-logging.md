---
title: 语音 SDK 日志记录-语音服务
titleSuffix: Azure Cognitive Services
description: 启用语音 SDK 中的日志记录。
services: cognitive-services
author: amitkumarshukla
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: amishu
ms.openlocfilehash: e8f470647c218188705cfa65cba7077e62d1b2ac
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65148033"
---
# <a name="enable-logging-in-the-speech-sdk"></a>启用语音 SDK 中的日志记录

文件的日志记录是语音 sdk 的可选功能。 在开发期间日志记录提供其他信息和来自语音 SDK 的核心组件的诊断。 可以通过将属性设置启用`Speech_LogFilename`上将语音配置对象传递给的位置和日志文件的名称。 日志记录将识别程序创建从该配置后全局激活，并且之后不能禁用。 日志记录会话运行期间，不能更改日志文件的名称。

> [!NOTE]
> 日志记录是在编程语言，但 JavaScript 除外的所有受支持语音 SDK 中提供。

## <a name="sample"></a>示例

配置对象上指定日志文件名称。 采取`SpeechConfig`为例，假设你已创建了一个实例调用`config`:

```csharp
config.SetProperty(PropertyId.Speech_LogFilename, "LogfilePathAndName");
```

```java
config.setProperty(PropertyId.Speech_LogFilename, "LogfilePathAndName");
```

```C++
config->SetProperty(PropertyId::Speech_LogFilename, "LogfilePathAndName");
```

```Python
config.set_property(speechsdk.PropertyId.Speech_LogFilename, "LogfilePathAndName")
```

```objc
[config setPropertyTo:@"LogfilePathAndName" byId:SPXSpeechLogFilename];
```

可以从 config 对象创建识别程序。 这将使所有的识别器的日志记录。

> [!NOTE]
> 如果您创建`SpeechSynthesizer`从 config 对象，它不会启用日志记录。 如果不过启用日志记录，你将收到来自诊断`SpeechSynthesizer`。

## <a name="create-a-log-file-on-different-platforms"></a>创建不同的平台上的日志文件

适用于 Windows 或 Linux，日志文件可以是用户有权写入任何路径中。 可能会限制或受限制的默认情况下在其他操作系统中的文件系统位置的写入权限。

### <a name="universal-windows-platform-uwp"></a>通用 Windows 平台 (UWP)

UWP 应用程序需要的位置中的其中一个应用程序数据的位置 （本地、 漫游，或临时） 日志文件。 可以在本地应用程序文件夹中创建一个日志文件：

```csharp
StorageFolder storageFolder = ApplicationData.Current.LocalFolder;
StorageFile logFile = await storageFolder.CreateFileAsync("logfile.txt", CreationCollisionOption.ReplaceExisting);
config.SetProperty(PropertyId.Speech_LogFilename, logFile.Path);
```

更多有关文件访问 UWP 应用程序的权限是可用[此处](https://docs.microsoft.com/windows/uwp/files/file-access-permissions)。

### <a name="android"></a>Android

可以将日志文件保存到内部存储、 外部存储或缓存目录。 在内部存储中创建文件或缓存目录专用于应用程序。 最好在外部存储中创建日志文件。

```java
File dir = context.getExternalFilesDir(null);
File logFile = new File(dir, "logfile.txt");
config.setProperty(PropertyId.Speech_LogFilename, logFile.getAbsolutePath());
```

上面的代码会将日志文件保存到特定于应用程序目录的根目录中的外部存储中。 用户可以访问该文件与文件管理器 (通常在`Android/data/ApplicationName/logfile.txt`)。 卸载应用程序时，将删除该文件。

你还需要请求`WRITE_EXTERNAL_STORAGE`清单文件中的权限：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="...">
  ...
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  ...
</manifest>
```

更多有关数据和文件存储为 Android 应用程序可[此处](https://developer.android.com/guide/topics/data/data-storage.html)。

#### <a name="ios"></a>iOS

仅在应用程序沙盒内的目录是可访问。 可以在文档、 库和临时目录中创建文件。 文档目录中的文件可以是供用户。 下面的代码段演示应用程序文档目录中创建一个日志文件：

```objc
NSString *filePath = [
  [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
    stringByAppendingPathComponent:@"logfile.txt"];
[speechConfig setPropertyTo:filePath byId:SPXSpeechLogFilename];
```

若要访问创建的文件，添加以下属性到`Info.plist`的应用程序的属性列表：

```xml
<key>UIFileSharingEnabled</key>
<true/>
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
```

更多有关 iOS 文件系统是可用[此处](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的示例](https://aka.ms/csspeech/samples)

