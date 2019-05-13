---
title: 安全地保存 Web 应用程序的机密应用程序设置 - Azure 密钥保管库 | Microsoft Docs
description: 如何使用 ASP.NET Core 密钥保管库提供程序、用户密钥或 .NET 4.7.1 配置生成器来安全地保存密钥应用程序设置（如 Azure 凭据或第三方 API 密钥）
services: visualstudio
author: cawaMS
manager: paulyuk
editor: ''
ms.service: key-vault
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: cawa
ms.openlocfilehash: 9763a14e84d88be1d6f09fb9f16b6b7c9eeffd2d
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65506426"
---
# <a name="securely-save-secret-application-settings-for-a-web-application"></a>安全地保存 Web 应用的密钥应用程序设置

## <a name="overview"></a>概述
本文介绍如何安全地保存 Azure 应用程序的密钥应用程序配置设置。

传统上，所有 Web 应用配置设置都保存在配置文件（如 Web.config）中。这一做法会导致将密钥设置（如云凭据）签入到公共源代码管理系统（如 GitHub）中。 同时，因为更改源代码和重新配置开发设置都需要开销，因此会难以遵循安全最佳实践。

为了确保开发过程的安全性，创建了工具和框架库，以安全地保存应用程序密钥设置，并最大程度地减少对源代码的更改或不对其进行更改。

## <a name="aspnet-and-net-core-applications"></a>ASP.NET 和 .NET Core 应用程序

### <a name="save-secret-settings-in-user-secret-store-that-is-outside-of-source-control-folder"></a>将密钥设置保存在源代码管理文件夹外部的用户密钥存储中。
如果你正在快速建立原型或无法访问 Internet，请先将源代码管理文件夹外部的密钥设置移动到用户密钥存储。 用户密钥存储是保存在用户探查器文件夹下的一个文件，因此密钥不会签入到源代码管理。 下图演示了[用户密钥](https://docs.microsoft.com/aspnet/core/security/app-secrets?tabs=visual-studio)的工作原理。

![用户密钥将密钥设置保持在源代码管理外部](./media/vs-secure-secret-appsettings/aspnetcore-usersecret.PNG)

如果正在运行 .NET Core 控制台应用程序，请使用密钥保管库来安全地保存你的密钥。

### <a name="save-secret-settings-in-azure-key-vault"></a>在 Azure 密钥保管库中保存密钥设置
如果正在开发一个项目，并需要安全地共享源代码，请使用 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)。

1. 在 Azure 订阅中创建密钥保管库。 填写 UI 上的所有必填字段，然后单击边栏选项卡底部的“创建”

    ![创建 Azure Key Vault](./media/vs-secure-secret-appsettings/create-keyvault.PNG)

2. 授予你和团队成员访问密钥保管库的权限。 如果你的团队规模较大，可以创建 [Azure Active Directory 组](https://docs.microsoft.com/azure/active-directory/active-directory-groups-create-azure-portal)并将该安全组访问权限添加到密钥保管库。 在“密钥权限”下拉列表中，检查“密钥管理操作”下的“获取”和“列表”。

    ![添加密钥保管库访问策略](./media/vs-secure-secret-appsettings/add-keyvault-access-policy.png)

3. 将密钥添加到 Azure 门户上的密钥保管库。 对于嵌套的配置设置，请将“:”替换为“--”，以使密钥保管库密钥名称有效。 不能在密钥保管库密钥的名称中使用“:”。

    ![添加密钥保管库密钥](./media/vs-secure-secret-appsettings/add-keyvault-secret.png)

    > [!NOTE] 
    > 在 Visual Studio 2017 V15.6 之前我们用于建议在安装用于 Visual Studio 的 Azure 服务身份验证扩展。 不推荐使用，但现在它集成在 Visual Studio 内。 因此如果要在 visual Studio 2017 的较旧版本，我们建议你至少更新到 VS 2017 15.6年或最高，以便可以以本机方式使用此功能并通过使用 Visual Studio 登录标识本身访问 Key vault。
    >
 
4. 将以下 NuGet 包添加到项目：

    ```
    Microsoft.Azure.Services.AppAuthentication
    ```
5. 将以下代码添加到 Program.cs 文件：

    ```csharp
    public static IWebHost BuildWebHost(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((ctx, builder) =>
            {
                var keyVaultEndpoint = GetKeyVaultEndpoint();
                if (!string.IsNullOrEmpty(keyVaultEndpoint))
                {
                    var azureServiceTokenProvider = new AzureServiceTokenProvider();
                    var keyVaultClient = new KeyVaultClient(
                        new KeyVaultClient.AuthenticationCallback(
                            azureServiceTokenProvider.KeyVaultTokenCallback));
                            builder.AddAzureKeyVault(
                            keyVaultEndpoint, keyVaultClient, new DefaultKeyVaultSecretManager());
                        }
                    })
                    .UseStartup<Startup>()
                    .Build();

        private static string GetKeyVaultEndpoint() => Environment.GetEnvironmentVariable("KEYVAULT_ENDPOINT");
    ```
6. 将密钥保管库 URL 添加到 launchsettings.json 文件。 环境变量名称 KEYVAULT_ENDPOINT 将在步骤 6 添加的代码中进行定义。

    ![将密钥保管库 URL 添加为项目环境变量](./media/vs-secure-secret-appsettings/add-keyvault-url.png)

7. 开始调试项目。 它应已成功运行。

## <a name="aspnet-and-net-applications"></a>ASP.NET 和 .NET 应用程序

.NET 4.7.1 支持密钥保管库和密钥配置生成器，这可以确保在不更改代码的情况下将密钥移动到源代码管理文件夹的外部。
若要继续，请[下载 .NET 4.7.1](https://www.microsoft.com/download/details.aspx?id=56115)，如果应用程序使用的是较旧版本的 .NET Framework，则将其进行迁移。

### <a name="save-secret-settings-in-a-secret-file-that-is-outside-of-source-control-folder"></a>将密钥设置保存在源代码管理文件夹外部的密钥文件中。
如果正在快速编写原型，且不想预配 Azure 资源，请使用此选项。

1. 将以下 NuGet 包安装到你的项目
    ```
    Microsoft.Configuration.ConfigurationBuilders.Basic
    ```

2. 创建类似于以下的文件。 将其保存在你的项目文件夹外部的某个位置下。

    ```xml
    <root>
        <secrets ver="1.0">
            <secret name="secret1" value="foo_one" />
            <secret name="secret2" value="foo_two" />
        </secrets>
    </root>
    ```

3. 将密钥文件定义为 Web.config 文件中的配置生成器。 将该部分置于 appSettings 部分前。

    ```xml
    <configBuilders>
        <builders>
            <add name="Secrets"
                 secretsFile="C:\Users\AppData\MyWebApplication1\secret.xml" type="Microsoft.Configuration.ConfigurationBuilders.UserSecretsConfigBuilder,
                    Microsoft.Configuration.ConfigurationBuilders, Version=1.0.0.0, Culture=neutral" />
        </builders>
    </configBuilders>
    ```

4. 指定 appSettings 部分使用密钥配置生成器。 确保有任何含有虚拟值的密钥设置的条目。

    ```xml
        <appSettings configBuilders="Secrets">
            <add key="webpages:Version" value="3.0.0.0" />
            <add key="webpages:Enabled" value="false" />
            <add key="ClientValidationEnabled" value="true" />
            <add key="UnobtrusiveJavaScriptEnabled" value="true" />
            <add key="secret" value="" />
        </appSettings>
    ```

5. 调试应用。 它应已成功运行。

### <a name="save-secret-settings-in-an-azure-key-vault"></a>在 Azure Key Vault 中保存密钥设置
按照 ASP.NET Core 部分中的说明为你的项目配置密钥保管库。

1. 将以下 NuGet 包安装到你的项目
   ```
   Microsoft.Configuration.ConfigurationBuilders.UserSecrets
   ```

2. 定义 Web.config 中的密钥保管库配置生成器。将该部分置于 appSettings 部分前。 如果密钥保管库位于公共 Azure 中，则将 vaultName 替换为密钥保管库名称，如果正在使用 Sovereign 云，则将其替换为完整的 URI。

    ```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>
    <configBuilders>
        <builders>
            <add name="AzureKeyVault" vaultName="Test911" type="Microsoft.Configuration.ConfigurationBuilders.AzureKeyVaultConfigBuilder, ConfigurationBuilders, Version=1.0.0.0, Culture=neutral" />
        </builders>
    </configBuilders>
    ```
3. 指定 appSettings 部分使用密钥保管库配置生成器。 确保有任何含有虚拟值的密钥设置的条目。

   ```xml
   <appSettings configBuilders="AzureKeyVault">
       <add key="webpages:Version" value="3.0.0.0" />
       <add key="webpages:Enabled" value="false" />
       <add key="ClientValidationEnabled" value="true" />
       <add key="UnobtrusiveJavaScriptEnabled" value="true" />
       <add key="secret" value="" />
   </appSettings>
   ```

4. 开始调试项目。 它应已成功运行。
