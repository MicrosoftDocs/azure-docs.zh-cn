---
title: 设置和使用 Azure 成本管理中配置 AWS 成本和使用情况报表集成
description: 本文将指导你虽然设置和使用 Azure 成本管理中配置 AWS 成本和使用情况报表集成。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 06/06/2019
ms.topic: conceptual
ms.service: cost-management
manager: ormaoz
ms.custom: ''
ms.openlocfilehash: e87e95ec9e4e20ee4785c2b1f448a7ca5f442b8a
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409196"
---
# <a name="set-up-and-configure-aws-cost-and-usage-report-integration"></a>设置和配置 AWS 成本和使用情况报表集成

通过 Amazon Web Services 成本和使用情况报表集成，可以监视并控制在 Azure 成本管理中 AWS 支出。 集成，可以在其中监视的 Azure 门户和 Azure 和 AWS 支出的控件中单个位置。 此文章介绍了如何设置集成并将其配置，以便使用成本管理功能来分析成本和查看预算。

成本管理流程通过 AWS 访问凭据来获取报表定义和下载的报表 GZIP CSV 文件存储在一个 S3 存储桶的 AWS 成本和使用情况报告。

## <a name="create-a-cost-and-usage-report-in-aws"></a>在 AWS 中创建的成本和使用情况报表

使用成本和使用情况报表是 AWS 推荐的方法来收集和处理 AWS 成本。 有关详细信息，请参阅[AWS 成本和使用情况报告](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-reports-costusage.html)文档文章。

使用**报表**计费和成本管理控制台在 AWS 中使用以下步骤创建的成本和使用情况报表页。

1. 登录到 AWS 管理控制台并打开在的计费和成本管理控制台 https://console.aws.amazon.com/billing。
2. 在导航窗格中，单击**报表**。
3. 单击**创建报表**。
4. 有关**报表名称**，键入报表的名称。
5. 有关**时间单位**，选择**每小时**。
6. 有关**Include**，在报表中添加的每个资源的 Id，然后选择**资源 Id**。
7. 有关**启用对支持**，没有选定内容是必需的。
8. 有关**的数据刷新设置**，选择**自动刷新你的成本&amp;使用情况报告上一个月检测到费用时关闭帐单**。
9. 单击“下一步”。
10. 有关**Amazon S3 存储桶**，键入要传递到的报表的 Amazon S3 存储桶的名称，然后单击**验证**。 存储桶必须具有适当的权限有效。 有关将权限添加到存储桶的详细信息，请参阅[设置存储桶和对象的访问权限](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/set-permissions.html)。
11. 有关**报表路径前缀**，键入你想要添加到报表的名称的报表路径前缀。
12. 有关**压缩**，选择**GZIP**。
13. 单击“下一步”。
14. 已为您的报表，单击查看设置后**评审和已完成**。
    请注意**报表名称**。 将在后续步骤中使用它。

可能需要 24 小时 AWS 着手将报表传递到 Amazon S3 存储桶。 传递开始后，AWS 更新 AWS 成本和使用情况报表文件至少一次一天。 你可以继续配置 AWS 环境而无需等待交付来启动。

## <a name="create-a-role-and-policy-in-aws"></a>在 AWS 中创建角色和策略

Azure 成本管理访问成本和使用情况报表所在的位置每天多次的 S3 存储桶。 成本的管理需要凭据以检查有新的数据访问。 在 AWS 以允许进行访问成本管理中创建角色和策略。

若要启用到 Azure 成本管理中的 AWS 帐户基于角色的访问，请在 AWS 控制台中创建角色。 需要能够_角色 ARN_并_外部 ID_从 AWS 控制台。 更高版本，您使用它们在创建 Azure 成本管理中的 AWS 连接器页面。

使用创建新的角色向导：

1. 登录到 AWS 控制台，然后选择**Services**。
2. 在服务列表中，选择**IAM**。
3. 选择**角色**，然后单击**Create Role**。
4. 在下一页上，选择**另一个 AWS 帐户**。
5. 在中**帐户 ID** ，输入_432263259397_。
6. 在中**选项**，选择**需要外部 ID （第三方将假定此角色时，最佳做法）**。
7. 在中**外部 ID**，输入外部 id。 外部 ID 是 AWS 角色和 Azure 成本管理之间共享的密码。 在成本管理中的新连接器页面也用于相同的外部 ID。 例如，类似于外部 ID _Companyname1234567890123_。
    不要更改为所选内容**要求使用 MFA**。 它应保留已清除。
8. 单击“下一步:权限”。
9. 单击“创建策略”。 新的浏览器选项卡将打开你在其中创建新的策略。
10. 单击**选择服务**。

配置成本和使用情况报表的权限：

1. 类型**成本和使用情况报告**。
2. 选择**访问级别**，**读取** > **DescribeReportDefinitions**。 这样，成本管理，请阅读当前报表定义，并确定它们是否匹配到报表定义系统必备组件。
3. 单击**将其他权限添加**。

配置 S3 存储桶和对象的权限：

1. 单击**选择服务**。
2. 类型_S3_。
3. 选择**访问级别**，**列表** > **ListBucket**。 此操作在 S3 存储桶中获取的对象的列表。
4. 选择**访问级别**，**读取** > **GetObject**。 此操作允许计费文件下载。
5. 选择**资源**。
6. 选择**存储桶 – 添加 ARN**。
7. 在中**存储桶名称**，输入用于存储当前文件的存储桶。
8. 选择**对象 – 添加 ARN**。
9. 在中**存储桶名称**，输入用于存储当前文件的存储桶。
10. 在中**对象名称**，选择**任何**。
11. 单击**将其他权限添加**。

配置成本资源管理器权限：

1. 单击**选择服务**。
2. 类型_成本资源管理器服务_。
3. 选择**所有成本资源管理器服务动作 (ce:\*)**。 此操作将验证集合正确。
4. 单击**将其他权限添加**。

添加组织的权限：

1. 类型**组织**。
2. 选择**访问级别，列表** > **ListAccounts**。 此操作获取的帐户的名称。
3. 在中**查看策略**，输入新策略的名称。 检查以确保输入正确的信息，然后单击**创建策略**。
4. 返回到上一个选项卡并刷新浏览器的 web 页面。 在搜索栏中，搜索新策略。
5. 选择**下一步： 查看**。
6. 输入新角色的名称。 检查以确保输入正确的信息，然后单击**Create Role**。
    请注意**角色 ARN**并**外部 ID**时创建该角色使用在前面的步骤。 设置 Azure 成本管理连接器时，你将从更高版本使用它们。

策略 JSON 应类似于下面的示例。 替换_bucketname_ S3 存储桶的名称。

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
"organizations:ListAccounts",
            "ce:*",
            "cur:DescribeReportDefinitions"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::bucketname",
                "arn:aws:s3:::bucketname/*"
            ]
        }
    ]
}
```

## <a name="set-up-a-new-aws-connector-in-azure"></a>设置在 Azure 中的新 AWS 连接器

使用以下信息来创建新的 AWS 连接器并开始监视你的 AWS 成本。

1. 登录到 Azure 门户，网址 https://portal.azure.com。
2. 导航到**成本管理 + 计费** > **成本管理**。
3. 下**设置**，单击**云连接器 （预览版）**。  
    ![显示云连接器 （预览设置） 示例](./media/aws-integration-setup-configure/cloud-connectors-preview01.png)。
4. 单击 **+ 添加**顶部的页后，可以创建新的连接器。
5. 在创建 AWS 连接器页中，键入**显示名称**命名你的连接器。  
    ![创建 AWS 连接器页面上的示例](./media/aws-integration-setup-configure/create-aws-connector01.png)
6. （可选） 选择的默认管理组。 它将存储所有已发现的链接的帐户。 您可以将其设置更高版本。
7. 下**计费**部分中，选择**自动收费公开上市时 1%** 如果你想要在预览过期时确保连续运营。 如果您选择自动选项，则必须选择计费**订阅**。
8. 输入**角色 ARN**。 它是您设置在 AWS 中的角色时使用的值。
9. 输入**外部 ID**。 它是您设置在 AWS 中的角色时使用的值。
10. 输入**报表名称**在 AWS 中创建。
11. 单击**下一步**，然后单击**创建**。

可能需要几个小时内的新 AWS 作用域、 AWS 合并帐户和 AWS 链接的帐户和其成本数据才会显示。

创建连接器之后，我们建议将访问控制分配给连接器。 用户权限分配给新发现作用域：AWS 合并帐户和 AWS 链接帐户。 创建连接器的用户是连接器、 合并的帐户和所有链接的帐户的所有者。

将连接器权限分配给用户，发现发生后不将权限分配给现有的 AWS 作用域。 相反，唯一的新链接的帐户分配权限。

## <a name="additional-steps"></a>其他步骤

- [将管理组设置](../governance/management-groups/index.md#initial-setup-of-management-groups)，如果你尚未准备好。
- 检查新的作用域都添加到作用域选择器。 别忘了单击**刷新**若要查看最新数据。
- 在云连接器页中，选择你的连接器，然后单击**转到计费帐户**将链接的帐户分配到管理组。

## <a name="manage-cloud-connectors"></a>管理云连接器

选择云连接器页上的连接器，你可以：

- 单击**转到计费帐户**来查看 AWS 合并帐户信息。
- 单击**访问控制**管理连接器角色分配。
- 单击**编辑**更新连接器。 在角色 ARN 中出现时，不能更改 AWS 帐户数。 但是，可以创建一个新的连接器。
- 单击**验证**重新运行验证测试，以确保成本管理可以收集数据使用的连接器设置。

![显示创建 AWS 连接器的列表示例](./media/aws-integration-setup-configure/list-aws-connectors.png)

## <a name="role-of-azure-management-groups"></a>Azure 管理组的角色

若要创建在一个位置即可查看跨云提供程序的信息，您需要将你的 Azure 订阅和链接的 AWS 帐户放在相同的管理组中。 如果你尚未使用管理组配置 Azure 环境，请参阅[初始的管理组设置](../governance/management-groups/index.md#initial-setup-of-management-groups)。

如果你想要单独的成本，可以创建包含只是链接的 AWS 帐户的管理组。

## <a name="aws-consolidated-account"></a>合并的 AWS 帐户

合并的 AWS 帐户用于合并计费和付款的多个 AWS 帐户。 合并的 AWS 帐户还可作为 AWS 链接帐户。

![AWS 合并帐户详细信息的示例](./media/aws-integration-setup-configure/aws-consolidated-account01.png)

从页中，你可以：

- 单击**更新**到大容量更新与管理组的链接 AWS 帐户关联。
- 单击**访问控制**设置作用域的角色分配。

### <a name="aws-consolidated-account-permissions"></a>AWS 合并的帐户权限

默认情况下，AWS 合并创建后，基于 AWS 连接器权限设置权限的帐户。 连接器创建者是所有者。

通过使用合并的 AWS 帐户的访问级别页管理的访问级别。 但是，AWS 的权限合并的帐户不会继承通过 AWS 链接的帐户。

## <a name="aws-linked-account"></a>AWS 链接帐户

AWS 链接帐户是其中创建和管理 AWS 资源。 链接的帐户还充当安全边界。

在此页您可以：

- 单击**更新**以更新链接 AWS 帐户与管理组的关联。
- 单击**访问控制**设置作用域的角色分配。

![AWS 链接帐户页的示例](./media/aws-integration-setup-configure/aws-linked-account01.png)

### <a name="aws-linked-account-permissions"></a>AWS 链接帐户权限

默认情况下，创建后，基于 AWS 连接器权限设置 AWS 链接的帐户权限。 连接器创建者是所有者。 通过使用 AWS 链接帐户的访问级别页管理的访问级别。 合并的 AWS 帐户的权限不会继承通过 AWS 链接的帐户。

AWS 链接帐户始终从其所属的管理组中继承权限。

## <a name="next-steps"></a>后续步骤

- 现在，设置并配置 AWS 成本和使用情况报表集成后，继续执行[管理 AWS 成本和用量](aws-integration-manage.md)。
- 如果已熟悉成本分析，请参阅[探索和分析使用成本分析成本](quick-acm-cost-analysis.md)快速入门。
- 如果您熟悉在 Azure 中的预算，请参阅[创建和管理 Azure 预算](tutorial-acm-create-budgets.md)教程。
