---
title: 在 Linux 上设置开发环境 | Microsoft Docs
description: 在 Linux 上安装运行时和 SDK 并创建本地开发群集。 完成此设置后，便可以开始生成应用程序。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: chackdan
editor: ''
ms.assetid: d552c8cd-67d1-45e8-91dc-871853f44fc6
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: a063461d9da66d57a7bdc3311ae80dec7f2c98f1
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470231"
---
# <a name="prepare-your-development-environment-on-linux"></a>在 Linux 上准备开发环境
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [OSX](service-fabric-get-started-mac.md)
>
>  

若要在 Linux 开发计算机上部署和运行 [Azure Service Fabric 应用程序](service-fabric-application-model.md)，请安装运行时和常用 SDK。 还可以安装用于 Java 和 .NET Core 开发的可选 SDK。 

本文中的步骤假设你在 Linux 上进行本地安装，或者使用 Service Fabric OneBox 容器映像 `microsoft/service-fabric-onebox`。

不支持在适用于 Linux 的 Windows 子系统上安装 Service Fabric 运行时和 SDK。 可以使用受支持的 Azure Service Fabric 命令行接口 (CLI) 来管理托管在云中或本地其他位置的 Service Fabric 实体。 有关如何安装 CLI 的信息，请参阅[设置 Service Fabric CLI](./service-fabric-cli.md)。


## <a name="prerequisites"></a>必备组件

支持使用以下操作系统版本进行开发。

* Ubuntu 16.04 (`Xenial Xerus`)

    确保已安装 `apt-transport-https` 包。
         
    ```bash
    sudo apt-get install apt-transport-https
    ```
* Red Hat Enterprise Linux 7.4（Service Fabric 预览版支持）


## <a name="installation-methods"></a>安装方法

### <a name="script-installation-ubuntu"></a>脚本安装 (Ubuntu)

为方便起见，我们提供了一个脚本用于连同 sfctl CLI 一起安装 Service Fabric 运行时和 Service Fabric 通用 SDK。 在以下部分中执行手动安装步骤。 可以看到正在安装的组件和相关许可证。 运行该脚本即认为你同意所要安装的所有软件的许可条款。

成功运行脚本后，可以跳转到[设置本地群集](#set-up-a-local-cluster)。

```bash
sudo curl -s https://raw.githubusercontent.com/Azure/service-fabric-scripts-and-templates/master/scripts/SetupServiceFabric/SetupServiceFabric.sh | sudo bash
```

### <a name="manual-installation"></a>手动安装
有关手动安装 Service Fabric 运行时和通用 SDK，请遵照本指南的余下部分操作。

## <a name="update-your-apt-sources-or-yum-repositories"></a>更新 APT 源或 Yum 存储库
若要通过 apt-get 命令行工具安装 SDK 和关联的运行时包，必须首先更新高级打包工具 (APT) 源。

### <a name="ubuntu"></a>Ubuntu

1. 打开终端。
2. 将 Service Fabric 存储库添加到源列表。

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/servicefabric/ xenial main" > /etc/apt/sources.list.d/servicefabric.list'
    ```

3. 将 `dotnet` 存储库添加到源列表。

    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    ```

4. 向 APT Keyring 添加新的 Gnu 隐私防护（GnuPG 或 GPG）密钥。

    ```bash
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 417A0893
    ```

5. 向 APT Keyring 添加官方的 Docker GPG 密钥。

    ```bash
    sudo apt-get install curl
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

6. 设置 Docker 存储库。

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

7. 将 Azul JDK 密钥添加到 APT Keyring 并设置其存储库。

    ```bash
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0x219BD9C9
    sudo apt-add-repository 'deb http://repos.azulsystems.com/ubuntu stable main'
    ```

8. 根据新添加的存储库刷新包列表。

    ```bash
    sudo apt-get update
    ```


### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4（Service Fabric 预览版支持）

1. 打开终端。
2. 下载并安装 Extra Packages for Enterprise Linux(EPEL)。

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum install epel-release-latest-7.noarch.rpm
    ```
3. 将 EfficiOS RHEL7 包存储库添加到系统。

    ```bash
    sudo wget -P /etc/yum.repos.d/ https://packages.efficios.com/repo.files/EfficiOS-RHEL7-x86-64.repo
    ```

4. 将 EfficiOS 包签名密钥导入到本地 GPG keyring。

    ```bash
    sudo rpmkeys --import https://packages.efficios.com/rhel/repo.key
    ```

5. 将 Microsoft RHEL 存储库添加到系统。

    ```bash
    curl https://packages.microsoft.com/config/rhel/7.4/prod.repo > ./microsoft-prod.repo
    sudo cp ./microsoft-prod.repo /etc/yum.repos.d/
    ```

6. 安装 .NET SDK。

    ```bash
    yum install rh-dotnet20 -y
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-a-local-cluster"></a>为本地群集安装并设置 Service Fabric SDK

更新源后，可安装 SDK。 安装 Service Fabric SDK 包，确认安装，并同意许可协议。

### <a name="ubuntu"></a>Ubuntu

```bash
sudo apt-get install servicefabricsdkcommon
```

> [!TIP]
>   以下命令自动接受 Service Fabric 包的许可证：
>   ```bash
>   echo "servicefabric servicefabric/accepted-eula-ga select true" | sudo debconf-set-selections
>   echo "servicefabricsdkcommon servicefabricsdkcommon/accepted-eula-ga select true" | sudo debconf-set-selections
>   ```

### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4（Service Fabric 预览版支持）

```bash
sudo yum install servicefabricsdkcommon
```

SDK 安装随附的 Service Fabric 运行时包含下表中所述的包。 

 | | DotNetCore | Java | Python | NodeJS | 
--- | --- | --- | --- |---
Ubuntu | 2.0.0 | AzulJDK 1.8 | Implicit from npm | 最新 |
RHEL | - | OpenJDK 1.8 | Implicit from npm | 最新 |

## <a name="set-up-a-local-cluster"></a>设置本地群集
安装完成后，启动本地群集。

1. 运行群集安装程序脚本。

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

2. 打开 Web 浏览器，转到 [Service Fabric Explorer](http://localhost:19080/Explorer) (`http://localhost:19080/Explorer`)。 群集启动后，可以看到 Service Fabric Explorer 仪表板。 群集完全设置可能需要几分钟时间。 如果浏览器无法打开该 URL 或者 Service Fabric Explorer 未显示系统已准备就绪，请等待几分钟，然后重试。

    ![Linux 上的 Service Fabric Explorer][sfx-linux]

    此时，能够部署预生成的 Service Fabric 应用程序包或基于来宾容器或来宾可执行文件的新应用程序包。 若要使用 Java 或 .NET Core SDK 生成新服务，请遵循以下部分中的可选设置步骤操作。


> [!NOTE]
> Linux 不支持独立的群集。


> [!TIP]
> 如果有可用的 SSD 磁盘，建议配合使用 `--clusterdataroot` 与 devclustersetup.sh 来传递 SSD 文件夹路径，实现优异的性能。

## <a name="set-up-the-service-fabric-cli"></a>设置 Service Fabric CLI

[Service Fabric CLI](service-fabric-cli.md) 包含用来与 Service Fabric 实体（包括群集和应用程序）交互的命令。 若要安装 CLI，请按照 [Service Fabric CLI](service-fabric-cli.md) 中的说明进行操作。


## <a name="set-up-yeoman-generators-for-containers-and-guest-executables"></a>为容器和来宾可执行文件安装 Yeoman 生成器
Service Fabric 提供基架工具，可以借助此类工具，使用 Yeoman 模板生成器从终端创建 Service Fabric 应用程序。 遵循以下步骤安装 Service Fabric Yeoman 模板生成器：

1. 在计算机上安装 Node.js 和 npm。

    ```bash
    sudo apt-add-repository "deb https://deb.nodesource.com/node_8.x $(lsb_release -s -c) main"
    sudo apt-get update
    sudo apt-get install nodejs
    ```
2. 通过 npm 在计算机上安装 [Yeoman](https://yeoman.io/) 模板生成器。

    ```bash
    sudo npm install -g yo
    ```
3. 通过 npm 安装 Service Fabric Yeo 容器生成器和来宾可执行文件生成器。

    ```bash
    sudo npm install -g generator-azuresfcontainer  # for Service Fabric container application
    sudo npm install -g generator-azuresfguest      # for Service Fabric guest executable application
    ```

安装生成器后，可通过运行 `yo azuresfguest` 或 `yo azuresfcontainer` 分别创建来宾可执行文件或容器服务。

## <a name="set-up-net-core-20-development"></a>设置 .NET Core 2.0 开发

安装[用于 Ubuntu 的 .NET Core 2.0 SDK](https://www.microsoft.com/net/core#linuxubuntu)，开始[创建 C# Service Fabric 应用程序](service-fabric-create-your-first-linux-application-with-csharp.md)。 NuGet.org 可为 .NET Core 2.0 Service Fabric 应用程序托管包，目前提供预览版。

## <a name="set-up-java-development"></a>设置 Java 开发

若要使用 Java 生成 Service Fabric 服务，请安装 Gradle 以运行生成任务。 运行以下命令来安装 Gradle。 Service Fabric Java 库是从 Maven 拉取的。


* Ubuntu

    ```bash
    curl -s https://get.sdkman.io | bash
    sdk install gradle 5.1
    ```

* Red Hat Enterprise Linux 7.4（Service Fabric 预览版支持）

  ```bash
  sudo yum install java-1.8.0-openjdk-devel
  curl -s https://get.sdkman.io | bash
  sdk install gradle
  ```

还需要为 Java 可执行文件安装 Service Fabric Yeo 生成器。 确保已安装 [Yeoman](#set-up-yeoman-generators-for-containers-and-guest-executables)，然后运行以下命令：

  ```bash
  npm install -g generator-azuresfjava
  ```
 
## <a name="install-the-eclipse-plug-in-optional"></a>安装 Eclipse 插件（可选）

可以在面向 Java 开发人员或 Java EE 开发人员的 Eclipse IDE 中安装用于 Service Fabric 的 Eclipse 插件。 可以使用 Eclipse，在 Service Fabric Java 应用程序的基础上创建 Service Fabric 来宾可执行应用程序和容器应用程序。

> [!IMPORTANT]
> Service Fabric 插件需要 Eclipse Neon 或更高版本。 请参阅此注意事项后面的说明，了解如何查看 Eclipse 的版本。 如果安装的是较旧版本的 Eclipse，可以从 [Eclipse 站点](https://www.eclipse.org)下载较新的版本。 建议不要在现有 Eclipse 安装版本的基础上进行安装（覆盖）。 可以先删除现有版本，然后再运行安装程序，也可以在另一目录中安装较新的版本。
> 
> 在 Ubuntu 上，建议直接从 Eclipse 站点进行安装，而不是使用包安装程序（`apt` 或 `apt-get`）。 这样做可确保获取最新版 Eclipse。 可以安装面向 Java 开发人员或 Java EE 开发人员的 Eclipse IDE。

1. 在 Eclipse 中，请确保已安装 Eclipse Neon 或更高版本，以及 Buildship 2.2.1 版或更高版本。 可以通过选择“帮助” > “关于 Eclipse” > “安装详细信息”查看已安装组件的版本。 可以按 [Eclipse Buildship：适用于 Gradle 的 Eclipse 插件][buildship-update]中的说明更新 Buildship。

2. 若要安装 Service Fabric 插件，请选择“帮助” > “安装新软件”。

3. 在“使用”框中，输入 **https://dl.microsoft.com/eclipse**。

4. 选择 **添加** 。

    ![“可用软件”页][sf-eclipse-plugin]

5. 选择 ServiceFabric 插件，然后选择“下一步”。

6. 执行安装步骤。 然后接受最终用户许可协议。

如果已安装 Service Fabric Eclipse 插件，请确保使用最新版本。 通过选择“帮助” > “关于 Eclipse” > “安装详细信息”查看。 然后，在已安装插件的列表中搜索 Service Fabric。如果可以使用更新的版本，请选择“更新”。

有关详细信息，请参阅[使用适用于 Eclipse 的 Service Fabric 插件开发 Java 应用程序](service-fabric-get-started-eclipse.md)。

## <a name="update-the-sdk-and-runtime"></a>更新 SDK 和运行时

若要更新到最新版本的 SDK 和运行时，请运行以下命令。

```bash
sudo apt-get update
sudo apt-get install servicefabric servicefabricsdkcommon
```
若要从 Maven 更新 Java SDK 二进制文件，需在 ``build.gradle`` 文件中更新相应的二进制文件的版本详细信息，使之指向最新 版本。 若要确切地了解需要在何处更新版本，请参阅 [Service Fabric 入门示例](https://github.com/Azure-Samples/service-fabric-java-getting-started)中的任何 ``build.gradle`` 文件。

> [!NOTE]
> 更新这些包可能导致本地开发群集停止运行。 按本文中的说明操作，在升级后重启本地群集。

## <a name="remove-the-sdk"></a>删除 SDK
若要删除 Service Fabric SDK，请运行以下命令。

* Ubuntu

    ```bash
    sudo apt-get remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    sudo apt-get install -f
    ```


* Red Hat Enterprise Linux 7.4（Service Fabric 预览版支持）

    ```bash
    sudo yum remove servicefabric servicefabicsdkcommon
    npm uninstall -g generator-azuresfcontainer
    npm uninstall -g generator-azuresfguest
    ```

## <a name="next-steps"></a>后续步骤

* [使用 Yeoman 在 Linux 上创建和部署第一个 Service Fabric Java 应用程序](service-fabric-create-your-first-linux-application-with-java.md)
* [使用适用于 Eclipse 的 Service Fabric 插件在 Linux 上创建和部署第一个 Service Fabric Java 应用程序](service-fabric-get-started-eclipse.md)
* [在 Linux 上创建第一个 CSharp 应用程序](service-fabric-create-your-first-linux-application-with-csharp.md)
* [在 OSX 上准备开发环境](service-fabric-get-started-mac.md)
* [在 Windows 上准备 Linux 开发环境](service-fabric-local-linux-cluster-windows.md)
* [使用 Service Fabric CLI 管理应用程序](service-fabric-application-lifecycle-sfctl.md)
* [Service Fabric Windows 和 Linux 差异](service-fabric-linux-windows-differences.md)
* [Service Fabric CLI 入门](service-fabric-cli.md)

<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png
