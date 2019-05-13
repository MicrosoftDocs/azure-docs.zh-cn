---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 2/20/2019
ms.author: erhopf
ms.openlocfilehash: 9469fd6a1ffc61e90948178b105abd9f83e55fde
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65020678"
---
1. 启动 Eclipse。

1. 在 Eclipse Launcher 中，在“工作区”字段中输入某个新工作区目录的名称。 然后选择“启动”。

   ![Eclipse Launcher 的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-01-create-new-eclipse-workspace.png)

1. 片刻之后，Eclipse IDE 的主窗口将会显示。 如果出现了欢迎屏幕，请将其关闭。

1. 从 Eclipse 菜单栏上，通过选择“文件” > “新建” > “项目”创建一个新项目。

1. 将显示“新建项目”对话框。 选择“Java 项目”，然后选择“下一步”。

   ![“新建项目”对话框的屏幕截图，其中突出显示了 Java 项目](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-02-select-wizard.png)

1. “新建 Java 项目”向导随即启动。 在“项目名称”字段中，输入 **quickstart**，然后选择 **JavaSE 1.8** 作为执行环境。 选择“完成”。

   ![“新建 Java 项目”向导的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-03-create-java-project.png)

1. 如果出现了“打开关联的透视图?”窗口，请选择“打开透视图”。

1. 在**包资源管理器**中，右键单击 **quickstart** 项目。 从上下文菜单中选择“配置” > “转换为 Maven 项目”。

   ![包资源管理器的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-04-convert-to-maven-project.png)

1. 此时将显示“新建 POM”窗口。 在“组 ID”字段中输入 **com.microsoft.cognitiveservices.speech.samples**，然后在“项目 ID”字段中输入 **quickstart**。 然后选择“完成”。

   ![“新建 POM”窗口的屏幕截图](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-05-configure-maven-pom.png)

1. 打开 **pom.xml** 文件并对其进行编辑。

   * 在文件末尾，在右标记 `</project>` 前面，创建一个 `repositories` 元素，使其中包含对语音 SDK 的 Maven 存储库的引用，如下所示：

     [!code-xml[POM Repositories](~/samples-cognitive-services-speech-sdk/quickstart/java-jre/pom.xml#repositories)]

   * 还添加 `dependencies` 元素（使用语音 SDK 版本 1.5.0 作为依赖项）：

     [!code-xml[POM Dependencies](~/samples-cognitive-services-speech-sdk/quickstart/java-jre/pom.xml#dependencies)]

   * 保存更改。
