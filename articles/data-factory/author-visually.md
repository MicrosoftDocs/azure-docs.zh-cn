---
title: 在 Azure 数据工厂中创作视觉对象 | Microsoft Docs
description: 了解如何在 Azure 数据工厂中使用视觉对象创作
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 01/09/2019
author: sharonlo101
ms.author: shlo
ms.reviewer: ''
manager: craigg
ms.openlocfilehash: b32e4abebdfb93312c60a25ca8b483f071e5e73c
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507830"
---
# <a name="visual-authoring-in-azure-data-factory"></a>Azure 数据工厂中的视觉对象创作
Azure 数据工厂用户界面体验 (UX) 允许你以可视方式创作和部署资源为你的数据工厂而无需编写任何代码。 通过此无代码的界面，可将活动拖放到管道画布上、执行测试运行、以迭代方式进行调试，以及部署和监视管道运行。 可通过两种方法使用 UX 执行视觉对象创作：

- 直接使用数据工厂服务
- 使用 Azure Repos Git 集成进行创作，以实现协作、源代码管理和版本控制。

## <a name="author-directly-with-the-data-factory-service"></a>直接使用数据工厂服务
使用数据工厂服务可视方式创作不同于使用 Git 集成可视方式创作，具体表现为以下两个方面：

- 数据工厂服务不包括存储所做的更改的 JSON 实体的存储库。
- 数据工厂服务未优化协作或版本控制。

![配置数据工厂服务](media/author-visually/configure-data-factory.png)

当使用 UX 创作画布借助数据工厂服务直接进行创作时，只能使用“全部发布”模式。 所做的任何更改都会直接发布到数据工厂服务。

![发布模式](media/author-visually/data-factory-publish.png)

## <a name="author-with-azure-repos-git-integration"></a>使用 Azure Repos Git 集成进行创作
使用 Azure Repos Git 集成进行创作允许在创作数据工厂管道时进行源代码管理和协作。 用户可以选择将数据工厂与 Azure Repos Git 组织存储库相关联，以进行源代码管理、协作和版本控制等。 单个 Azure Repos Git 组织可以有多个存储库，但 Azure Repos Git 存储库只能与一个数据工厂相关联。 如果没有 Azure Repos 组织或存储库，请按照[这些说明](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization-msa-or-work-student)创建资源。

> [!NOTE]
> 可以在 Azure Repos Git 存储库中存储脚本和数据文件。 但是，必须手动将文件上传到 Azure 存储。 数据工厂管道不会自动将 Azure Repos Git 存储库中存储的脚本或数据文件上传到 Azure 存储。

### <a name="configure-an-azure-repos-git-repository-with-azure-data-factory"></a>使用 Azure 数据工厂配置 Azure Repos Git 存储库
用户可通过两种方法使用数据工厂配置 Azure Repos Git 存储库。

#### <a name="method1"></a> 配置方法 1（Azure Repos Git 存储库）：“开始使用”页

在 Azure 数据工厂，转到**让我们开始吧**页。 选择“配置代码存储库”：

![配置 Azure Repos 代码存储库](media/author-visually/configure-repo.png)

**存储库设置**配置窗格中显示：

![配置存储库设置](media/author-visually/repo-settings.png)

窗格将显示以下 Azure Repos 代码存储库设置：

| 设置 | 描述 | 值 |
|:--- |:--- |:--- |
| **存储库类型** | Azure Repos 代码存储库的类型。<br/> | Azure Repos Git |
| **Azure Active Directory** | Azure AD 租户的名称。 | `<your tenant name>` |
| **Azure Repos 组织** | Azure Repos 组织名称。 可以在 `https://{organization name}.visualstudio.com` 中找到 Azure Repos 组织名称。 可以[登录到 Azure Repos 组织](https://www.visualstudio.com/team-services/git/)来访问 Visual Studio 配置文件并查看存储库和项目。 | `<your organization name>` |
| **ProjectName** | Azure Repos 项目名称。 可以在 `https://{organization name}.visualstudio.com/{project name}` 中找到 Azure Repos 项目名称。 | `<your Azure Repos project name>` |
| **RepositoryName** | Azure Repos 代码存储库名称。 Azure Repos 项目包含 Git 存储库，随着项目的发展来管理源代码。 要么创建一个新的存储库，要么使用项目中已有的存储库。 | `<your Azure Repos code repository name>` |
| **协作分支** | 将用于发布的 Azure Repos 协作分支。 默认为 `master`。 如果希望从其他分支发布资源，可更改此设置。 | `<your collaboration branch name>` |
| **根文件夹** | Azure Repos 协作分支中的根文件夹。 | `<your root folder name>` |
| 确认选中“将现有的数据工厂资源导入存储库”选项。 | 指定是否从 UX 创作画布将现有数据工厂资源导入 Azure Repos Git 存储库。 选择相应的框以将你的数据工厂资源导入 JSON 格式关联的 Git 存储库。 此操作单独导出每个资源（即，链接的服务和数据集导出到单独的 JSON）。 如果未选中此框，不能导入现有的资源。 | 已选择（默认） |

#### <a name="configuration-method-2-azure-repos-git-repo-ux-authoring-canvas"></a>配置方法 2（Azure Repos Git 存储库）：UX 创作画布
在 Azure 数据工厂 UX**创作画布**，找到你的数据工厂。 选择**数据工厂**下拉列表菜单，然后选择**配置代码存储库**。

此时将显示配置窗格。 有关配置设置的详细信息，请参阅中的说明<a href="#method1">配置方法 1</a>。

![配置 UX 创作的代码存储库设置](media/author-visually/configure-repo-2.png)

### <a name="use-a-different-azure-active-directory-tenant"></a>使用不同的 Azure Active Directory 租户

可在不同的 Azure Active Directory 租户中创建 Azure Repos Git 存储库。 若要指定不同的 Azure AD 租户，必须对所用 Azure 订阅拥有管理员权限。

### <a name="use-your-personal-microsoft-account"></a>使用 Microsoft 个人帐户

若要将 Microsoft 个人帐户用于 Git 集成，可以将你的 Azure 个人存储库链接到贵组织的 Active Directory。

1. 将你的 Microsoft 个人帐户添加到贵组织的 Active Directory 以作为来宾。 有关详细信息，请参阅[在 Azure 门户中添加 Azure Active Directory B2B 协作用户](../active-directory/b2b/add-users-administrator.md)。

2. 使用 Microsoft 个人帐户登录 Azure 门户。 然后切换到贵组织的 Active Directory。

3. 请转到 Azure DevOps 部分，现在你可以在其中看到个人存储库。 选择存储库并连接到 Active Directory。

执行这些配置步骤之后，在数据工厂 UI 中设置 Git 集成时个人存储库可用。

有关将 Azure 存储库连接到贵组织的 Active Directory 的详细信息，请参阅[将 Azure DevOps 组织连接到 Azure Active Directory](/azure/devops/organizations/accounts/connect-organization-to-azure-ad)。

### <a name="switch-to-a-different-git-repo"></a>切换到不同 Git 存储库

若要切换到不同 Git 存储库，请在“数据工厂概述”页的右上角找到该图标，如以下屏幕截图所示。 如果看不到该图标，请清除本地浏览器缓存。 选择该图标以删除与当前存储库的关联。

删除与当前存储库的关联后，可以将 Git 设置配置为使用不同的存储库。 然后，可以将现有的数据工厂资源导入到新存储库。

![删除与当前 Git 存储库的关联](media/author-visually/remove-repo.png)

### <a name="use-version-control"></a>使用版本控制
版本控制（也称为_源代码管理_）系统允许开发人员协作处理代码并跟踪对基本代码所做的更改。 源代码管理是用于多开发人员项目的重要工具。

与数据工厂关联的每个 Azure Repos Git 存储库都有一个协作分支。 （默认协作分支为 `master`）。 用户还可以通过单击“+ 新建分支”创建功能分支，并在功能分支中进行开发工作。

![通过同步或发布更改代码](media/author-visually/sync-publish.png)

准备好功能分支的功能开发后，可以单击“创建拉取请求”。 此操作会将你转到 Azure Repos Git，可以在这里发起拉取请求、完成代码评审并将更改合并至协作分支。 （默认分支为 `master`）。 只能从协作分支发布至数据工厂服务。 

![创建新的拉取请求](media/author-visually/create-pull-request.png)

### <a name="configure-publishing-settings"></a>配置发布设置

若要配置发布分支（即在其中保存资源管理器模板的分支），将 `publish_config.json` 文件添加到协作分支的根文件夹中。 数据工厂读取此文件，查找字段 `publishBranch`，并使用所提供的值创建新的分支（如果尚不存在）。 然后它将所有资源管理器模板都保存到指定位置。 例如:

```json
{
    "publishBranch": "factory/adf_publish"
}
```

从 Git 模式发布时，可以确认数据工厂使用的是所需的发布分支，如以下屏幕截图中所示：

![确认正确的发布分支](media/author-visually/configure-publish-branch.png)

当指定新的发布分支，数据工厂不会删除以前的发布分支。 如果想要远程删除以前的发布分支，请手动将其删除。

加载工厂时数据工厂仅读取 `publish_config.json` 文件。 如果已在门户中加载工厂，请刷新浏览器，以便所做的更改生效。

### <a name="publish-code-changes"></a>发布代码更改
将更改合并至协作分支（默认为 `master`）后，选择“发布”以将主分支中的代码更改手动发布至数据工厂服务。

![将更改发布至数据更改服务](media/author-visually/publish-changes.png)

> [!IMPORTANT]
> 主分支不表示数据工厂服务中部署的内容。 必须将主分支手动发布到数据工厂服务。

### <a name="advantages-of-git-integration"></a>Git 集成的优点

-   **源代码管理**。 随着数据工厂工作负载变得至关重要，你可能需要将工厂与 Git 集成，以利用以下几种源代码管理优势：
    -   跟踪/审核更改的功能。
    -   还原导致 bug 的更改的功能。
-   **部分保存**。 在工厂中进行大量更改后，便将发现，在常规实时模式下，无法将更改保存为草稿，这是因为你尚未准备就绪，或者是因为你不希望因电脑发生故障而丢失更改。 通过 Git 集成，可以继续以增量方式保存更改，并且仅在准备就绪后才发布到工厂。 Git 会充当工作的暂存位置，直到对更改进行测试并感到满意为止。
-   **协作和控制**。 如果有多个团队成员参与同一工厂，则可能需要通过代码评审流程让团队成员相互协作。 此外，还可以设置工厂，以免工厂的每一位参与者都有权部署到工厂。 可以仅允许团队成员通过 Git 进行更改，但只允许团队中的某些人将更改“发布”到工厂。
-   **显示差异**。 在 Git 模式下，可以看到即将发布到工厂的有效负载的精细差异。 此差异显示了自上次发布到工厂后已修改/添加/删除的所有资源/实体。 基于此差异，可以继续进行发布，也可以返回并检查更改，稍后再回来。
-   **更好的 CI/CD**。 如果使用的是 Git 模式，则可以将发布管道配置为开发工厂中发生任何更改后立即自动触发。 此外，还可以自定义工厂中可用作资源管理器模板的参数的属性。 仅将所需属性集保留为参数并将其他所有内容进行硬编码可能很有用。
-   **性能更好**。 在 Git 模式下，平均工厂的加载速度比常规实时模式下的快 10 倍，因为资源是通过 Git 下载的。

### <a name="best-practices-for-git-integration"></a>Git 集成的最佳做法

-   **权限**。 通常情况下，你不希望所有团队成员都有权更新工厂。
    -   所有团队成员都应具有对数据工厂的读取权限。
    -   应仅允许选定的一组人员发布到工厂，为此，他们需要成为工厂“数据工厂参与者”角色的成员。
    -   对于源代码管理，其中一种良好做法是不允许直接签入到协作分支。 此要求可防止 bug，因为每次签入都会通过拉取请求过程。
-   **切换模式**。
    -    进入 Git 模式后，我们不建议来回切换到实时模式，这主要是因为在切换回 Git 时，将无法看到在实时模式下进行的任何更改。 请尝试在 Git 模式下进行更改，然后通过 UI 将其发布。
    -   同样，也不要使用任何数据工厂 powershell cmdlet，因为它们通过直接将提供的更改应用于实时工厂，会实现同样的效果。
-   **使用 Azure Key Vault 中的密码**。
    -   我们强烈建议使用 AzureKeyVault 将任何连接字符串或密码存储到数据工厂链接服务。
    -   我们不会在 Git 中存储任何此类机密信息（出于安全原因），因此对链接服务的任何更改都会立即发布到实时工厂。 有时候并不需要进行此类立即发布，因为这些更改可能尚未经过测试，这违背了 Git 的目的。
    -   因此，必须从使用基于 Azure Key Vault 的链接服务中获取所有此类机密。
    -   使用 Key Vault 的其他一些优势在于，这可使 CICD 变得更轻松，因为它不要求在资源管理器模板部署期间提供这些机密。

## <a name="author-with-github-integration"></a>使用 GitHub 集成进行创作

使用 GitHub 集成进行创作允许在创作数据工厂管道时进行源代码管理和协作。 用户可以选择将数据工厂与 GitHub 帐户存储库相关联，以进行源代码管理、协作和版本控制。 单个的 GitHub 帐户可以有多个存储库，但 GitHub 存储库可以只有一个数据工厂与相关联。 如果没有 GitHub 帐户或存储库，请按照 [这些说明](https://github.com/join) 创建资源。

GitHub 与数据工厂的集成支持公共 GitHub（即 [https://github.com](https://github.com)）和 GitHub Enterprise。 只要你对 GitHub 中的存储库具有读写权限，就可以将公共和专用 GitHub 存储库与数据工厂一起使用。

若要配置 GitHub 存储库，必须对所用 Azure 订阅拥有管理员权限。

有关此功能的 9 分钟介绍和演示，请观看以下视频：

> [!VIDEO https://channel9.msdn.com/shows/azure-friday/Azure-Data-Factory-visual-tools-now-integrated-with-GitHub/player]

### <a name="limitations"></a>限制

- 可在 GitHub 存储库中存储脚本和数据文件。 但是，必须手动将文件上传到 Azure 存储。 数据工厂管道不会自动将 GitHub 存储库中存储的脚本或数据文件上传到 Azure 存储。

- 2.14.0 以下版本的 GitHub Enterprise 在 Microsoft Edge 浏览器中无法正常运行。

- 数据工厂视觉创作工具的 GitHub 集成仅适用于正式版数据工厂。

### <a name="configure-a-public-github-repository-with-azure-data-factory"></a>使用 Azure 数据工厂配置公共 GitHub 存储库

用户可通过两种方法使用数据工厂配置 GitHub 存储库。

**配置方法 1（公共存储库）：“开始使用”页**

在 Azure 数据工厂中，转到“开始使用” **** 页。 选择“配置代码存储库” ****：

![数据工厂“入门”页](media/author-visually/github-integration-image1.png)

此时将显示“存储库设置” **** 配置窗格：

![GitHub 存储库设置](media/author-visually/github-integration-image2.png)

窗格将显示以下 Azure Repos 代码存储库设置：

| **设置**                                              | **说明**                                                                                                                                                                                                                                                                                                                                                                                                                   | **值**          |
|----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| **存储库类型**                                      | Azure Repos 代码存储库的类型。                                                                                                                                                                                                                                                                                                                                                                                             | GitHub             |
| **GitHub 帐户**                                       | GitHub 帐户名。 此名称可从 https:\//github.com/{account 名称} / {存储库名称}。 导航到此页时，系统会提示输入 GitHub 帐户的 GitHub OAuth 凭据。                                                                                                                                                                                                                                               |                    |
| **RepositoryName**                                       | GitHub 代码存储库名称。 GitHub 帐户包含用于管理源代码的 Git 存储库。 可以创建新存储库，或使用帐户中现有的存储库。                                                                                                                                                                                                                              |                    |
| **协作分支**                                 | 将用于发布的 GitHub 协作分支。 默认为主分支。 如果希望从其他分支发布资源，可更改此设置。                                                                                                                                                                                                                                                               |                    |
| **根文件夹**                                          | GitHub 协作分支中的根文件夹。                                                                                                                                                                                                                                                                                                                                                                             |                    |
| 确认选中“将现有的数据工厂资源导入存储库”选项。 | 指定是否将现有数据工厂资源从 UX  **创作画布** 导入到 GitHub 存储库中。 选择相应的框以将你的数据工厂资源导入 JSON 格式关联的 Git 存储库。 此操作单独导出每个资源（即，链接的服务和数据集导出到单独的 JSON）。 如果未选中此框，不能导入现有的资源。 | 已选择（默认） |
| **要将资源导入到的分支**                       | 指定要将数据工厂资源（管道、数据集、链接服务等）导入哪个分支。 可将资源导入以下分支之一：a. 协作；b. 新建；c. 使用现有                                                                                                                                                                                                     |                    |

#### <a name="configuration-method-2-public-repo-ux-authoring-canvas"></a>配置方法 2（公共存储库）：UX 创作画布

在 Azure 数据工厂 UX  **创作画布**中，找到数据工厂。 选择“数据工厂” **** 下拉菜单，然后选择“配置代码存储库” ****。

此时将显示配置窗格。 有关配置设置的详细信息，请参阅前面 *配置方法 1* 中的说明。

### <a name="configure-a-github-enterprise-repository-with-azure-data-factory"></a>使用 Azure 数据工厂配置 GitHub Enterprise 存储库

可通过两种方法使用数据工厂配置 GitHub Enterprise 存储库。

#### <a name="configuration-method-1-enterprise-repo-lets-get-started-page"></a>配置方法 1（Enterprise 存储库）：“开始使用”页

在 Azure 数据工厂中，转到“开始使用” **** 页。 选择“配置代码存储库” ****：

![数据工厂“入门”页](media/author-visually/github-integration-image1.png)

此时将显示“存储库设置” **** 配置窗格：

![GitHub 存储库设置](media/author-visually/github-integration-image3.png)

窗格将显示以下 Azure Repos 代码存储库设置：

| **设置**                                              | **说明**                                                                                                                                                                                                                                                                                                                                                                                                                   | **值**          |
|----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| **存储库类型**                                      | Azure Repos 代码存储库的类型。                                                                                                                                                                                                                                                                                                                                                                                             | GitHub             |
| **使用 GitHub Enterprise**                                | 用于选择 GitHub Enterprise 的复选框                                                                                                                                                                                                                                                                                                                                                                                              |                    |
| **GitHub Enterprise URL**                                | GitHub Enterprise 根 URL。 例如： https://github.mydomain.com                                                                                                                                                                                                                                                                                                                                                          |                    |
| **GitHub 帐户**                                       | GitHub 帐户名。 此名称可从 https:\//github.com/{account 名称} / {存储库名称}。 导航到此页时，系统会提示输入 GitHub 帐户的 GitHub OAuth 凭据。                                                                                                                                                                                                                                               |                    |
| **RepositoryName**                                       | GitHub 代码存储库名称。 GitHub 帐户包含用于管理源代码的 Git 存储库。 可以创建新存储库，或使用帐户中现有的存储库。                                                                                                                                                                                                                              |                    |
| **协作分支**                                 | 将用于发布的 GitHub 协作分支。 默认为主分支。 如果希望从其他分支发布资源，可更改此设置。                                                                                                                                                                                                                                                               |                    |
| **根文件夹**                                          | GitHub 协作分支中的根文件夹。                                                                                                                                                                                                                                                                                                                                                                             |                    |
| 确认选中“将现有的数据工厂资源导入存储库”选项。 | 指定是否将现有数据工厂资源从 UX  **创作画布** 导入到 GitHub 存储库中。 选择相应的框以将你的数据工厂资源导入 JSON 格式关联的 Git 存储库。 此操作单独导出每个资源（即，链接的服务和数据集导出到单独的 JSON）。 如果未选中此框，不能导入现有的资源。 | 已选择（默认） |
| **要将资源导入到的分支**                       | 指定要将数据工厂资源（管道、数据集、链接服务等）导入哪个分支。 可将资源导入以下分支之一：a. 协作；b. 新建；c. 使用现有                                                                                                                                                                                                     |                    |

#### <a name="configuration-method-2-enterprise-repo-ux-authoring-canvas"></a>配置方法 2（Enterprise 存储库）：UX 创作画布

在 Azure 数据工厂 UX  **创作画布**中，找到数据工厂。 选择“数据工厂” **** 下拉菜单，然后选择“配置代码存储库” ****。

此时将显示配置窗格。 有关配置设置的详细信息，请参阅前面 *配置方法 1* 中的说明。

## <a name="use-the-expression-language"></a>使用表达式语言
可使用 Azure 数据工厂支持的表达式语言，在定义属性值过程中指定表达式。

通过选择“添加动态内容”，指定属性值的表达式：

![使用表达式语言](media/author-visually/dynamic-content-1.png)

## <a name="use-functions-and-parameters"></a>使用函数和参数

可以在数据工厂“表达式生成器”中使用函数或指定管道和数据集的参数：

有关受支持的表达式的更多信息，请参阅 [Azure 数据工厂中的表达式和函数](control-flow-expression-language-functions.md)。

![添加动态内容](media/author-visually/dynamic-content-2.png)

## <a name="provide-feedback"></a>提供反馈
选择“反馈”可发表有关功能的看法，或者告知 Microsoft 出现了工具问题：

![反馈](media/author-visually/provide-feedback.png)

## <a name="next-steps"></a>后续步骤
若要了解有关监视和管理管道的信息，请参阅[以编程方式监视和管理管道](monitor-programmatically.md)。
