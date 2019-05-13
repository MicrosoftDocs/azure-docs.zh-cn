---
title: Azure 应用程序配置最佳做法 |Microsoft Docs
description: 了解如何充分利用 Azure 应用程序配置
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: d1275a48de5cad9321186ba20860d853b8ce55ad
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413626"
---
# <a name="azure-app-configuration-best-practices"></a>Azure 应用程序配置最佳做法

使用 Azure 应用配置时，本文讨论了常见的模式和实践。

## <a name="key-groupings"></a>键分组

应用配置提供了用于组织键的两个选项： 键前缀或标签。 可以使用一个或两个。

键前缀是密钥的开始部分。 逻辑上可以通过在其名称中使用相同的前缀分组键的一组。 前缀可以包含多个组件连接在一起，通过分隔符如`/`，类似于 URL 路径，以形成一个命名空间。 在许多应用程序、 组件服务和环境的密钥存储在一个应用配置存储区时，此类层次结构是很有用。 要记住的重要一点是，密钥是在应用程序代码引用来检索相应的设置的值。 不应更改密钥，否则您将需要修改代码每次发生的。

标签是对密钥的属性。 它们用于创建密钥的变体。 例如，可以将标签分配给多个版本的密钥。 一个版本可以是迭代、 环境或某些其他上下文信息。 你的应用程序可以请求一组完全不同的密钥值，只需通过指定另一个标签。 所有键的引用可以保持不变。

## <a name="key-value-compositions"></a>键-值组合

应用配置将作为独立的实体与它存储的所有键。 它不会尝试推断任何键之间的关系或继承其层次结构所基于的密钥值。 可以聚合多个集的密钥，但是，使用标签结合堆叠在应用程序代码中的正确配置。

接下来举例说明。 设置**Asset1**其值为"开发"环境可能会有所不同。 可以创建一个空标签和名为"开发"的标签名为"Asset1"的项。 放置的默认值为**Asset1**前者和后者中任何特定值的"开发"中。 在代码中，首先检索而无需任何标签，然后将那些具有"开发"标签以覆盖任何以前的值的相同键的键值。 如果使用.NET Core 之类的现代编程框架，您可以如果本机配置提供程序用于访问应用程序配置免费获得此堆叠的功能。 以下代码片段演示如何实现.NET Core 应用程序中堆叠。

```csharp
// Augment the ConfigurationBuilder with Azure App Configuration
// Pull the connection string from an environment variable
configBuilder.AddAzureAppConfiguration(options => {
    options.Connect(configuration["connection_string"])
           .Use(KeyFilter.Any, LabelFilter.Null)
           .Use(KeyFilter.Any, "Development");
});
```

## <a name="app-configuration-bootstrap"></a>应用配置 bootstrap

若要访问的应用配置存储区，可以使用其连接字符串，这是在 Azure 门户中可用。 连接字符串包含凭据的信息，并被视为机密。 他们需要存储在密钥保管库中。 更好的选择是使用 Azure 管理的标识。 使用此方法，只需启动到配置存储区的访问权限的应用配置终结点 URL。 可以在应用程序代码中嵌入 URL (例如，在*appsettings.json*文件)。 请参阅[与 Azure 集成托管标识](howto-integrate-azure-managed-service-identity.md)的更多详细信息。

## <a name="web-app-or-function-access-to-app-configuration"></a>Web 应用或函数应用配置权限

可以通过 Azure 门户的应用服务应用程序设置到应用配置存储区中输入连接字符串。 此外可以将其存储在密钥保管库和[从应用服务中引用该](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references)。 此外可以使用 Azure 托管的标识来访问配置存储区。 请参阅[与 Azure 集成托管标识](howto-integrate-azure-managed-service-identity.md)的更多详细信息。

或者，可以将配置从应用配置推送到应用服务。 应用配置提供了将数据发送到应用服务直接导出函数 （在 Azure 门户和 CLI）。 使用此方法，您不需要更改应用程序代码。

## <a name="next-steps"></a>后续步骤

* [键和值](./concept-key-value.md)
