---
title: Azure 资源管理器体验版 |Azure Marketplace
description: 使用 Azure 资源管理器生成市场体验版
services: Azure, Marketplace, Cloud Partner Portal,
author: pbutlerm
manager: Patrick .Butler
ms.service: marketplace
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: pabutler
ms.openlocfilehash: 7665050dfc9a561f42fec00c40d0a40dfa5cc183
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64941577"
---
# <a name="azure-resource-manager-test-drive"></a>Azure 资源管理器体验版

本文适用于有产品/服务在 Azure 市场上或使用 AppSource 但只想使用 Azure 资源构建体验版的发布者。

Azure 资源管理器 (Resource Manager) 模板是你的解决方案将其设计最佳表示到的 Azure 资源的编码的容器。 如果您不熟悉资源管理器模板是，研读[了解资源管理器模板](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)并[创作资源管理器模板](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates)以确保您知道如何生成和测试您自己的模板。

体验版的作用是利用所提供的资源管理器模板，将从资源管理器模板中获得的全部资源部署到资源组。

若选择构建 Azure 资源管理器体验版，需满足以下要求：

- 构建、测试然后上传体验版资源管理器模板。
- 配置所需的所有元数据和设置，以启用体验版。
- 启用体验版后重新发布产品/服务。

## <a name="how-to-build-an-azure-resource-manager-test-drive"></a>如何构建 Azure 资源管理器体验版

下面是用于构建 Azure 资源管理器测试驱动器的过程：

1. 设计要你的客户，在数据流关系图中执行操作。
1. 定义希望客户构建哪些体验。
1. 根据上述定义，决定哪些部分和资源所需的客户，若要完成这种体验： 例如，D365 实例或与数据库的网站。
1. 生成本地，设计和测试体验。
1. 包在 ARM 模板部署中，再从那里的体验：
    1. 定义资源的哪些部分是输入的参数;
    1. 变量是什么;
    1. 哪些输出都被赋予客户体验。
1. 发布、 测试和投入。

构建 Azure 资源管理器体验版的最重要环节是定义要让客户体验的方案。 这是一款防火墙产品吗？你是要演示它处理脚本注入攻击的功能多么强大吗？ 这是一款存储产品吗？你是要演示此解决方案压缩文件的功能是多么快速简捷吗？

请务必花费足够的时间评估展示您的产品的最佳方法什么。 特别是围绕所需的所有资源必须，因为它可打包资源管理器模板充分更容易。

要具备以下体系结构才能继续进行防火墙示例：需要一个公共 IP URL 供服务使用，及另外一个公共 IP URL 供防火墙所保护的网站使用。 每个 IP 都部署在虚拟机上，并通过网络安全组和网络接口连接在一起。

在已经设计的资源所需的包，接下来的编写和测试驱动器资源管理器模板的生成。

## <a name="writing-test-drive-resource-manager-templates"></a>编写体验版资源管理器模板

体验版使用完全自动化模式运行部署，因此，体验版模板具有以下限制。

### <a name="parameters"></a>parameters

大多数模板都有一组参数。 参数定义了资源名称、资源大小（例如存储帐户类型或虚拟机大小）、用户名及密码、DNS 名称等。 使用 Azure 门户部署解决方案时，可以手动填充所有的这些参数、选择可用的 DNS 名称或存储帐户名称等。

![Azure 资源管理器中的参数列表](./media/azure-resource-manager-test-drive/param1.png)

但体验版使用完全自动化模式，无需人工干预，因此，体验版仅支持有限的一组参数类别。 若体验版资源管理器模板中的某个参数不属于受支持类别，则必须使用变量或常量值替换此参数。

你可以将任意有效的名称用于参数，体验版使用元数据类型值识别参数类别。 必须为每个模板参数指定元数据类型，否则模板将无法通过验证：

```json
"parameters": {
  ...
  "username": {
    "type": "string",
    "metadata": {
      "type": "username"
    }
  },
  ...
}
```

另请注意，所有参数均为可选，因此，若不想要使用任意一个参数，可不必使用。

### <a name="accepted-parameter-metadata-types"></a>接受的参数元数据类型

| 元数据类型   | 参数类型  | 描述     | 示例值    |
|---|---|---|---|
| baseuri     | string          | 部署包的基 URI| https:\//\<\..\>.blob.core.windows.net/\<\..\> |
| **username**    | string          | 随机的新用户名。| admin68876      |
| **password**    | 安全字符串    | 随机的新密码 | Lp!ACS\^2kh     |
| 会话 ID   | string          | 唯一的体验版会话 ID (GUID)    | b8c8693e-5673-449c-badd-257a405a6dee |

#### <a name="username"></a>username

体验版使用部署包的基 URI 初始化此参数，因此，可以使用此参数构造包含到包内的任意文件的 URI。

```json
"parameters": {
  ...
  "baseuri": {
    "type": "string",
    "metadata": {
      "type": "baseuri",
      "description": "Base Uri of the deployment package."
    }
  },
  ...
}
```

在模板内部，可以使用此参数构造体验版部署包中的任意文件的 URI。 以下示例演示如何构造链接的模板的 URI：

```json
"templateLink": {
  "uri": "[concat(parameters('baseuri'),'templates/solution.json')]",
  "contentVersion": "1.0.0.0"
}
```

#### <a name="username"></a>username

体验版使用随机的新用户名初始化此参数：

```json
"parameters": {
  ...
  "username": {
    "type": "string",
    "metadata": {
      "type": "username",
      "description": "Solution admin name."
    }
  },
  ...
}
```

示例值：

    admin68876

可以将随机的或恒定的用户名用于解决方案。

#### <a name="password"></a>password

体验版使用随机的新密码初始化此参数：

```json
"parameters": {
  ...
  "password": {
    "type": "securestring",
    "metadata": {
      "type": "password",
      "description": "Solution admin password."
    }
  },
  ...
}
```

示例值：

    Lp!ACS^2kh

可以将随机的或恒定的密码用于解决方案。

#### <a name="session-id"></a>会话 ID

体验版使用表示体验版会话 ID 的唯一 GUID 初始化此参数：

```json
"parameters": {
  ...
  "sessionid": {
    "type": "string",
    "metadata": {
      "type": "sessionid",
      "description": "Unique Test Drive session id."
    }
  },
  ...
}
```

示例值：

    b8c8693e-5673-449c-badd-257a405a6dee

如有必要，可以使用此参数对体验版会话进行唯一地标识。

### <a name="unique-names"></a>唯一的名称

一些 Azure 资源（如存储帐户或 DNS 名称）需要具有全局唯一的名称。

这意味着，每当体验版部署资源管理器模板时，它都会为其所有的资源创建一个**具有唯一名称的新资源组**\'。 因此，需要将 [uniquestring](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-functions#uniquestring) 函数与资源组 ID 上的变量名称连接在一起使用，以生成随机的唯一值：

```json
"variables": {
  ...
  "domainNameLabel": "[concat('contosovm',uniquestring(resourceGroup().id))]",
  "storageAccountName": "[concat('contosodisk',uniquestring(resourceGroup().id))]",
  ...
}
```

请务必将参数/变量字符串 (\'contosovm\') 与唯一的字符串输出 (\'resourceGroup().id\') 连接在一起，因为这样可确保每个变量的唯一性和可靠性。

例如，大多数的资源名称不能以数字开头，而唯一字符串函数可以返回一个以数字开头的字符串。 因此，若使用原始的唯一字符串输出，部署将失败。 

有关详细信息，可查看[本文](https://docs.microsoft.com/azure/guidance/guidance-naming-conventions)中的资源命名规则和限制。

### <a name="deployment-location"></a>部署位置

你可以在不同的 Azure 区域中部署体验版。 此想法是希望让用户选择最近的区域，进而提供最佳的用户体验。

当体验版创建实验室实例时，通常会在用户所选的区域中创建一个资源组，然后在此组上下文中执行部署模板。 因此，模板应从资源组选择部署位置：

```json
"variables": {
  ...
  "location": "[resourceGroup().location]",
  ...
}
```

然后将此位置用于特定实验室实例的各个资源：

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/publicIPAddresses",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/virtualNetworks",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Network/networkInterfaces",
    "location": "[variables('location')]",
    ...
  },
  {
    "type": "Microsoft.Compute/virtualMachines",
    "location": "[variables('location')]",
    ...
  }
]
```

需确保允许订阅在所选的各个区域中部署想要部署的所有资源。 同时，还需确保虚拟机映像在将要启用的所有区域中可用，否则，部署模板将不适用于某些区域。

### <a name="outputs"></a>Outputs

通常情况下，在使用资源管理器模板时，无需生成任何输出即可进行部署。 因为你知道用于填充模板参数的所有值，并且始终可以手动检查任意资源的属性。

但就体验版资源管理器模板而言，将所有信息（网站 URI、虚拟机主机名、用户名及密码）返回到体验版至关重要，因为需要使用这些信息来访问实验室。 请确保所有输出名称是可读的，因为这些变量要呈现给客户。

对于模板输出，没有任何限制。 只需记住，体验版会将所有输出值转换为字符串，因此，如果将对象发送到输出，用户会看到 JSON 字符串。

示例：

```json
"outputs": {
  "Host Name": {
    "type": "string",
    "value": "[reference(variables('pubIpId')).dnsSettings.fqdn]"
  },
  "User Name": {
    "type": "string",
    "value": "[parameters('adminName')]"
  },
  "Password": {
    "type": "string",
    "value": "[parameters('adminPassword')]"
  }
}
```

### <a name="subscription-limits"></a>订阅限制

此外，还需考虑订阅和服务限制。 如若要部署多达 10 个 4 核虚拟机，需确保用于实验室的订阅允许使用 40 核。

有关详细信息，可查看[本文中](https://docs.microsoft.com/azure/azure-subscription-service-limits)的 Azure 订阅和服务限制。 由于可同时使用多个体验版，因此，请验证订阅是否能够处理可采用的并发体验版总数乘以核数后的总核数。

### <a name="what-to-upload"></a>要上传的内容

体验版资源管理器模板以 zip 文件格式上传，其中可包含各种部署项目，但需有一个名称为“main-template.json”的文件。 此文件是 Azure 资源管理器部署模板，体验版使用它来实例化实验室。

如果有除此文件之外的其他资源，可将其引用为此模板内的外部资源，或者可以将该资源包含在 zip 文件中。

在发布认证期间，体验版会将部署包解压缩，并将其中的内容放入内部体验版 blob 容器。 此容器结构反映部署包的结构：

| package.zip                       | 体验版 blob 容器         |
|---|---|
| main-template.json                | https:\//\<\...\>.blob.core.windows.net/\<\...\>/main-template.json  |
| templates/solution.json           | https:\//\<\...\>.blob.core.windows.net/\<\...\>/templates/solution.json |
| scripts/warmup.ps1                | https:\//\<\...\>.blob.core.windows.net/\<\...\>/scripts/warmup.ps1  |


我们将此 blob 容器的 URI 称为基 URI。 实验室的每个修订版有其自己的 blob 容器，因此，实验室的每个修订版有其自己的基 URI。 体验版可以通过模板参数将未解压的部署包的基 URI 传递到模板。

## <a name="transforming-template-examples-for-test-drive"></a>体验版模板转换示例

将资源体系结构转换为体验版资源管理器模板的过程可能会让人望而却步。 为此，我们提供了以最佳方式[转换当前的部署模板](./transforming-examples-for-test-drive.md)示例，以便简化此过程。

## <a name="how-to-publish-a-test-drive"></a>如何发布体验版

至此，已经构建体验版，下面将逐步介绍用于成功发布体验版的各个必填字段。

![在用户界面中启用体验版](./media/azure-resource-manager-test-drive/howtopub1.png)

第一个字段（也是最重要的字段）是切换是否要为产品/服务启用体验版。 选择“是”时，将显示包含所有必填字段的表单的其余部分，以供填写。如果选择“否”，则会禁用该表单；在禁用体验版的情况下重新发布时，会从生产环境中删除体验版。

请注意:如果用户正在活跃地使用任何体验版，则这些体验版将继续运行，直到用户的会话过期。

### <a name="details"></a>详细信息

要填写的下一个部分是体验版产品/服务的详细信息。

![体验版详细信息](./media/azure-resource-manager-test-drive/howtopub2.png)

说明 - [必填] 在此处编写体验版功能的主要说明。 客户将在此处阅读产品体验版涵盖的方案。 

用户手册 - 必填。体验版体验的深入演练。 客户将打开此文档，并完全根据其中的说明演练整个体验版的操作。 此内容必须容易理解和遵循！ （必须是 .pdf 文件）

体验版演示视频 - 建议填写。类似于用户手册，最好是在其中包含体验版体验的视频教程。 客户在使用体验版之前和使用期间将会观看此视频，并完全根据其中的说明演练整个体验版的操作。 此内容必须容易理解和遵循！

- 名称 - 视频的标题
- 链接 - 必须是视频网站或视频的嵌入式 URL。 下面是有关如何获取嵌入式 URL 的示例：
- 缩略图 - 必须是优质图像（533x324 像素）。 建议在此处插入体验版体验某些部分的屏幕截图。

下面显示了在客户使用体验版体验期间，如何向他们显示这些字段。

![体验版字段在市场产品/服务中的位置](./media/azure-resource-manager-test-drive/howtopub4.png)

### <a name="technical-configuration"></a>技术配置

要填写的下一个部分是指定在何处上传体验版资源管理器模板，并定义体验版实例的具体工作方式。

![](./media/azure-resource-manager-test-drive/howtopub5.png)

实例- 必填。在此处配置需要多少个实例、在哪些区域及客户多久可以获得体验版。

- 实例 - “选择区域”处用于选择在何处部署体验版资源管理器模板。 建议仅选择一个最希望客户位于的区域。
- 热 - 每个选定区域中已部署的和正在等待访问的体验版实例数。 客户无需等待部署，就能立即访问此体验版。 弊端在于这些实例始终在 Azure 订阅上运行，因此，它们会产生较高的运行时间成本。 强烈建议拥有至少一个“热门”实例，因为多数客户不想等待完整部署完成，这样客户使用量就会有所下降。
- **暖** - 每个区域已部署的体验版实例数，且在部署这些实例后停止了 VM 并将其存储在 Azure 存储中。 “暖”实例的等待时间慢于“热”实例，但存储的运行时间成本也较低。
- **冷** - 每个区域可能部署的体验版实例数。 “冷”实例要求在客户请求体验版时整个体验版资源管理器模板完成部署，因此，它慢于“热”或“暖”实例。 但其优势在于只需为体验版的持续时间付费。

现在计算要提供的潜在并发体验版总数，并验证订阅的配额限制是否可以处理此并发数目：

（选定区域数 x “热”实例数）+（选定区域数 x “暖”实例数）+（选定区域数 x “冷”实例数）

体验版持续时间(小时) – 必填。体验版保持活动状态的持续时间，以小时为单位。 此时间段结束后，体验版会自动终止。

体验版资源管理器模板 - 必填。在此处上传资源管理器模板。 这是指在上一个部分中生成的文件。 将主模板文件命名为“main-template.json”，并确保资源管理器模板包含所需的关键变量的输出参数。 （必须是 .zip 文件）

访问信息 - 必填。客户获取体验版后，会向他们显示访问信息。 这些说明旨在共享体验版资源管理器模板中的有用输出参数。 若要包含输出参数，请使用双大括号（例如 {{outputname}}），这样，它们就会正确插入到相应的位置。 （建议在此处使用 HTML 字符串格式，以便在前端正确显示）。

### <a name="test-drive-deployment-subscription-details"></a>体验版部署订阅详细信息

要填写的最后一个部分是确保能够通过连接 Azure 订阅和 Azure Active Directory (AD) 来自动部署体验版。

![体验版部署订阅详细信息](./media/azure-resource-manager-test-drive/subdetails1.png)

Azure 订阅 ID - 必填。授予对 Azure 服务和 Azure 门户的访问权限。 将在该订阅中报告资源用量和计收服务费用。 如果没有一个专门用于体验版的独立 Azure 订阅，请创建一个订阅。 登录到 Azure 门户并导航到左侧菜单中的“订阅”，即可找到 Azure 订阅 ID。 （示例：“a83645ac-1234-5ab6-6789-1h234g764ghty”）

![Azure 订阅](./media/azure-resource-manager-test-drive/subdetails2.png)

Azure AD 租户 ID - 必填。如果已有一个租户 ID，可在“属性”-\>“目录 ID”下面找到它。

![Azure Active Directory 属性](./media/azure-resource-manager-test-drive/subdetails3.png)

否则，请在 Azure Active Directory 中创建一个新租户。

![Azure Active Directory 租户列表](./media/azure-resource-manager-test-drive/subdetails4.png)

![定义组织、域和 Azure AD 租户的国家/地区](./media/azure-resource-manager-test-drive/subdetails5.png)

![确认所选内容](./media/azure-resource-manager-test-drive/subdetails6.png)

Azure AD 应用 ID - 必填。下一步是创建并注册新应用程序。 我们将使用此应用程序对体验版实例执行操作。

1. 导航到新建的目录或现有目录，然后在筛选器窗格中选择“Azure Active Directory”。
2. 搜索“应用注册”，然后单击“添加”
3. 提供应用程序名称。
4. 选择“Web 应用/API”作为“类型”
5. 在“登录 URL”中提供任意值。我们不会使用该字段。
6. 单击“创建”。
7. 创建应用程序后，转到“属性”-\>“将应用程序设为多租户”，然后点击“保存”。

单击“保存”。 最后一步是获取已注册应用的应用程序 ID，并将其粘贴到此处的体验版字段中。

![Azure AD 应用程序 ID 详细信息](./media/azure-resource-manager-test-drive/subdetails7.png)

由于我们要使用该应用程序部署到订阅，因此，需将该应用程序添加为订阅中的参与者。 下面是操作说明：

1. 导航到“订阅”边栏选项卡，并选择专门用于体验版的相应订阅。
1. 单击“访问控制(IAM)”。
1. 单击“角色分配”选项卡。![添加新的访问控制主体](./media/azure-resource-manager-test-drive/SetupSub7_1.jpg)
1. 单击“添加角色分配”。
1. 将角色设置为“参与者”。
1. 键入 Azure AD 应用程序的名称，并选择要向其分配该角色的应用程序。
    ![添加权限](./media/azure-resource-manager-test-drive/SetupSub7_2.jpg)
1. 单击“ **保存**”。

Azure AD 应用密钥 - 必填。最终的字段将会生成身份验证密钥。 在密钥下添加密钥说明，将持续时间设置为永不过期，然后选择“保存”。 必须避免使用已过期的密钥，否则，在生产环境中体验版将会中断。 复制此值，并将其粘贴到所需的体验版字段中。

![显示 Azure AD 应用程序密钥](./media/azure-resource-manager-test-drive/subdetails8.png)

## <a name="next-steps"></a>后续步骤

填写所有体验版字段后，请继续重新发布产品/服务。 在体验版通过认证后，应该对产品/服务预览版中的客户体验进行广泛的测试。 在 UI 中启动体验版，然后在 Azure 门户内打开 Azure 订阅并验证是否已正确地完整部署了体验版。

![Azure 门户](./media/azure-resource-manager-test-drive/subdetails9.png)

请务必注意，不要删除为客户预配的任何体验版实例，在客户用完体验版之后，体验版服务将自动清理这些资源组。

如果对预览版产品/服务感到满意，现在就可以将它上线！ 发布产品/服务后，Microsoft 会完成最终评审过程，以仔细检查完整的端到端体验。 如果出于某种原因拒绝了该产品/服务，我们会向产品/服务的工程联系人发送通知，解释需要解决哪些问题。

如果有其他问题、寻求故障排除建议，或想要让体验版获得更大的成功，请访问[常见问题解答、故障排除和最佳做法](./marketing-and-best-practices.md)。
