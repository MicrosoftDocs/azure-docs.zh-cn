---
title: 教程：使用 SSMS 在 Azure SQL 数据库中设计第一个关系数据库 | Microsoft Docs
description: 了解如何使用 SQL Server Management Studio 在 Azure SQL 数据库的单一数据库中设计第一个关系数据库。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.topic: tutorial
author: CarlRabeler
ms.author: carlrab
ms.reviewer: v-masebo
manager: craigg
ms.date: 02/08/2019
ms.openlocfilehash: 3b802573bfc1eba74217d01005a2f232ff575fda
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/28/2019
ms.locfileid: "64730393"
---
# <a name="tutorial-design-a-relational-database-in-a-single-database-within-azure-sql-database-using-ssms"></a>教程：使用 SSMS 在 Azure SQL 数据库的单一数据库中设计关系数据库

Azure SQL 数据库是 Microsoft 云 (Azure) 中的关系型数据库即服务 (DBaaS)。 在本教程中，了解如何使用 Azure 门户 SQL 和 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) (SSMS) 执行以下操作：

> [!div class="checklist"]
> - 使用 Azure 门户创建单一数据库*
> - 通过 Azure 门户设置服务器级 IP 防火墙规则
> - 使用 SSMS 连接到数据库
> - 使用 SSMS 创建表
> - 使用 BCP 大容量加载数据
> - 使用 SSMS 查询数据

*如果还没有 Azure 订阅，请在开始前[创建免费帐户](https://azure.microsoft.com/free/)。

> [!NOTE]
> 本教程使用单一数据库。 你也可以使用弹性池中的共用数据库，或托管实例中的实例数据库。 若要连接到托管实例，请参阅以下托管实例快速入门：[快速入门：配置 Azure VM 以连接到 Azure SQL 数据库托管实例](sql-database-managed-instance-configure-vm.md)和[快速入门：配置从本地到 Azure SQL 数据库托管实例的点到站点连接](sql-database-managed-instance-configure-p2s.md)。

## <a name="prerequisites"></a>先决条件

要完成本教程，请确保已安装：

- [SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx)（最新版本）
- [BCP 和 SQLCMD](https://www.microsoft.com/download/details.aspx?id=36433)（最新版本）

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com/)。

## <a name="create-a-blank-single-database"></a>创建空的单一数据库

创建 Azure SQL 数据库中的单一数据库时，会使用定义好的一组计算和存储资源。 数据库在 [Azure 资源组](../azure-resource-manager/resource-group-overview.md)中创建，使用[数据库服务器](sql-database-servers.md)进行托管。

遵循以下步骤创建空白的单一数据库。

1. 在 Azure 门户的左上角单击“创建资源”。
2. 在“新建”页上的“Azure 市场”部分中选择“数据库”，然后在“特别推荐”部分中单击“SQL 数据库”。

   ![创建空数据库](./media/sql-database-design-first-database/create-empty-database.png)

3. 如上图所示，在“SQL 数据库”表单中填写以下信息：

    | 设置       | 建议的值 | 说明 |
    | ------------ | ------------------ | ------------------------------------------------- |
    | **数据库名称** | yourDatabase | 如需有效的数据库名称，请参阅[数据库标识符](/sql/relational-databases/databases/database-identifiers)。 |
    | **订阅** | yourSubscription  | 有关订阅的详细信息，请参阅[订阅](https://account.windowsazure.com/Subscriptions)。 |
    | **资源组** | yourResourceGroup | 如需有效的资源组名称，请参阅 [Naming rules and restrictions](/azure/architecture/best-practices/naming-conventions)（命名规则和限制）。 |
    | **选择源** | 空白数据库 | 指定创建空白数据库。 |

4. 单击“服务器”以使用现有的数据库服务器，或者创建并配置新的数据库服务器。 选择现有服务器或单击“创建新服务器”，然后在“新建服务器”窗体中填写以下信息：

    | 设置       | 建议的值 | 说明 |
    | ------------ | ------------------ | ------------------------------------------------- |
    | **服务器名称** | 任何全局唯一名称 | 如需有效的服务器名称，请参阅 [Naming rules and restrictions](/azure/architecture/best-practices/naming-conventions)（命名规则和限制）。 |
    | 服务器管理员登录名 | 任何有效的名称 | 如需有效的登录名，请参阅[Database Identifiers](/sql/relational-databases/databases/database-identifiers)（数据库标识符）。 |
    | **密码** | 任何有效的密码 | 密码必须至少有八个字符，且必须使用以下类别中的三个类别的字符：大写字符、小写字符、数字以及非字母数字字符。 |
    | **位置** | 任何有效的位置 | 有关区域的信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/)。 |

    ![创建数据库 - 服务器](./media/sql-database-design-first-database/create-database-server.png)

5. 单击“选择”。
6. 单击“定价层”，指定服务层级、DTU 或 vCore 数，以及存储量。 可以浏览相关选项，了解每个服务层级可提供的 DTU/vCore 数和存储。

    选择服务层、DTU 数或 vCore 数以及存储量后，然后单击“应用”。

7. 输入空白数据库的“排序规则”（就本教程来说，请使用默认值）。 有关排序规则的详细信息，请参阅 [Collations](/sql/t-sql/statements/collations)（排序规则）

8. 填写“SQL 数据库”窗体后，单击“创建”以预配单一数据库。 这个步骤可能需要几分钟的时间。

9. 在工具栏上，单击“通知”可监视部署过程。

   ![通知](./media/sql-database-design-first-database/notification.png)

## <a name="create-a-server-level-ip-firewall-rule"></a>创建服务器级 IP 防火墙规则

SQL 数据库服务在服务器级别创建 IP 防火墙。 此防火墙阻止外部应用程序和工具连接到服务器和服务器上的任何数据库，除非防火墙规则允许其 IP 通过防火墙。 若要启用与单一数据库的外部连接，必须首先为 IP 地址（或 IP 地址范围）添加 IP 防火墙规则。 遵循这些步骤创建 [SQL 数据库服务器级 IP 防火墙规则](sql-database-firewall-configure.md)。

> [!IMPORTANT]
> SQL 数据库服务通过端口 1433 进行通信。 如果尝试从企业网络内部连接到此服务，则该网络的防火墙可能不允许经端口 1433 的出站流量。 如果是这样，则无法连接到单一数据库，除非管理员打开端口 1433。

1. 部署完成后，在左侧菜单中单击“SQL 数据库”，然后在“SQL 数据库”页上单击“yourDatabase”。 此时会打开数据库的概览页，其中显示了完全限定的“服务器名称”（例如 yourserver.database.windows.net），并提供了其他配置的选项。

2. 复制此完全限定的服务器名称，将其用于从 SQL Server Management Studio 连接到服务器和数据库。

   ![服务器名称](./media/sql-database-design-first-database/server-name.png)

3. 单击工具栏上的“设置服务器防火墙”。 此时会打开 SQL 数据库服务器的“防火墙设置”页。

   ![服务器级别 IP 防火墙规则](./media/sql-database-design-first-database/server-firewall-rule.png)

4. 在工具栏上单击“添加客户端 IP”，将当前的 IP 地址添加到新的 IP 防火墙规则。 IP 防火墙规则可以针对单个 IP 地址或一系列 IP 地址打开端口 1433。

5. 单击“ **保存**”。 此时会针对当前的 IP 地址创建服务器级 IP 防火墙规则，在 SQL 数据库服务器上打开端口 1433。

6. 单击“确定”，然后关闭“防火墙设置”页。

你的 IP 地址现在可以通过 IP 防火墙。 现在可以使用 SQL Server Management Studio 或其他所选工具连接到单一数据库。 确保使用之前创建的服务器管理员帐户。

> [!IMPORTANT]
> 默认情况下，所有 Azure 服务都允许通过 SQL 数据库 IP 防火墙进行访问。 在此页上单击“关”即可对所有 Azure 服务执行禁用操作。

## <a name="connect-to-the-database"></a>连接到数据库

使用 [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) 来与单一数据库建立连接。

1. 打开 SQL Server Management Studio。
2. 在“连接到服务器”对话框中，输入以下信息：

   | 设置       | 建议的值 | 说明 |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **服务器类型** | 数据库引擎 | 此值是必需的。 |
   | **服务器名称** | 完全限定的服务器名称 | 例如，yourserver.database.windows.net。 |
   | **身份验证** | SQL Server 身份验证 | SQL 身份验证是本教程中配置的唯一身份验证类型。 |
   | **登录名** | 服务器管理员帐户 | 在创建服务器时指定的帐户。 |
   | **密码** | 服务器管理员帐户的密码 | 创建服务器时指定的密码。 |

   ![连接到服务器](./media/sql-database-design-first-database/connect.png)

3. 单击“连接到服务器”对话框中的“选项”。 在“连接到数据库”部分输入 yourDatabase，以连接到此数据库。

    ![连接到服务器上的 DB](./media/sql-database-design-first-database/options-connect-to-db.png)  

4. 单击“连接”。 此时会在 SSMS 中打开“对象资源管理器”窗口。

5. 在对象资源管理器中展开“数据库”，然后展开 yourDatabase，查看示例数据库中的对象。

   ![数据库对象](./media/sql-database-design-first-database/connected.png)  

## <a name="create-tables-in-your-database"></a>在数据库中创建表

使用 [Transact-SQL](/sql/t-sql/language-reference) 创建具有 4 个表格的数据库架构，这些表格是大专院校的学生管理系统的模型：

- 人员
- 课程
- 学生
- 额度

以下关系图显示了这些表的相互关系。 其中一些表引用其他表中的列。 例如，“学生”表引用“人员”表的 PersonId 列。 请研究此关系图，了解本教程中各种表的相互关系。 若要深入了解如何创建有效的数据库表，请参阅[创建有效的数据库表](https://msdn.microsoft.com/library/cc505842.aspx)。 有关选择数据类型的信息，请参阅[数据类型](/sql/t-sql/data-types/data-types-transact-sql)。

> [!NOTE]
> 还可以使用 [SQL Server Management Studio 中的表设计器](/sql/ssms/visual-db-tools/design-database-diagrams-visual-database-tools)来创建和设计表。

![表关系](./media/sql-database-design-first-database/tutorial-database-tables.png)

1. 在“对象资源管理器”中，右键单击 yourDatabase，并选择“新建查询”。 此时会打开一个空白查询窗口，该窗口连接到数据库。

2. 在查询窗口中，执行以下查询以在数据库中创建 4 个表：

   ```sql
   -- Create Person table
   CREATE TABLE Person
   (
       PersonId INT IDENTITY PRIMARY KEY,
       FirstName NVARCHAR(128) NOT NULL,
       MiddelInitial NVARCHAR(10),
       LastName NVARCHAR(128) NOT NULL,
       DateOfBirth DATE NOT NULL
   )

   -- Create Student table
   CREATE TABLE Student
   (
       StudentId INT IDENTITY PRIMARY KEY,
       PersonId INT REFERENCES Person (PersonId),
       Email NVARCHAR(256)
   )

   -- Create Course table
   CREATE TABLE Course
   (
       CourseId INT IDENTITY PRIMARY KEY,
       Name NVARCHAR(50) NOT NULL,
       Teacher NVARCHAR(256) NOT NULL
   )

   -- Create Credit table
   CREATE TABLE Credit
   (
       StudentId INT REFERENCES Student (StudentId),
       CourseId INT REFERENCES Course (CourseId),
       Grade DECIMAL(5,2) CHECK (Grade <= 100.00),
       Attempt TINYINT,
       CONSTRAINT [UQ_studentgrades] UNIQUE CLUSTERED
       (
           StudentId, CourseId, Grade, Attempt
       )
   )
   ```

   ![创建表](./media/sql-database-design-first-database/create-tables.png)

3. 展开“对象资源管理器”中 yourDatabase 下的“表”节点以查看创建的表。

   ![创建的 ssms 表](./media/sql-database-design-first-database/ssms-tables-created.png)

## <a name="load-data-into-the-tables"></a>将数据加载到表

1. 在“下载”文件夹中创建名为 sampleData 的文件夹，为数据库存储示例数据。

2. 右键单击以下链接并将它们保存到 sampleData 文件夹。

   - [SampleCourseData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCourseData)
   - [SamplePersonData](https://sqldbtutorial.blob.core.windows.net/tutorials/SamplePersonData)
   - [SampleStudentData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleStudentData)
   - [SampleCreditData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCreditData)

3. 打开命令提示符窗口并导航到 sampleData 文件夹。

4. 执行以下命令，将示例数据插入表，使用环境值替换“服务器”、“数据库”、“用户”和“密码”的值。

   ```cmd
   bcp Course in SampleCourseData.txt -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Person in SamplePersonData.txt -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Student in SampleStudentData.txt -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Credit in SampleCreditData.txt -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   ```

现已将示例数据加载到了之前创建的表中。

## <a name="query-data"></a>查询数据

执行以下查询，从数据库表中检索信息。 有关写入 SQL 查询的详细信息，请参阅[写入 SQL 查询](https://technet.microsoft.com/library/bb264565.aspx)。 第一个查询将联接所有 4 个表，以查找由“Dominick Pope”授课的分数高于 75% 的学生。 第二个查询将联接所有 4 个表，以查找“Noe Coleman”注册过的课程。

1. 在 SQL Server Management Studio 查询窗口中，执行以下查询：

   ```sql
   -- Find the students taught by Dominick Pope who have a grade higher than 75%
   SELECT  person.FirstName, person.LastName, course.Name, credit.Grade
   FROM  Person AS person
       INNER JOIN Student AS student ON person.PersonId = student.PersonId
       INNER JOIN Credit AS credit ON student.StudentId = credit.StudentId
       INNER JOIN Course AS course ON credit.CourseId = course.courseId
   WHERE course.Teacher = 'Dominick Pope'
       AND Grade > 75
   ```

2. 在查询窗口中执行以下查询：

   ```sql
   -- Find all the courses in which Noe Coleman has ever enrolled
   SELECT  course.Name, course.Teacher, credit.Grade
   FROM  Course AS course
       INNER JOIN Credit AS credit ON credit.CourseId = course.CourseId
       INNER JOIN Student AS student ON student.StudentId = credit.StudentId
       INNER JOIN Person AS person ON person.PersonId = student.PersonId
   WHERE person.FirstName = 'Noe'
       AND person.LastName = 'Coleman'
   ```

## <a name="next-steps"></a>后续步骤

本教程介绍了许多基本数据库任务。 你已了解如何：

> [!div class="checklist"]
> - 创建单一数据库
> - 设置服务器级 IP 防火墙规则
> - 使用 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) (SSMS) 连接到该数据库
> - 创建表
> - 批量加载数据
> - 查询该数据

转向下一教程，了解如何使用 Visual Studio 和 C# 设计数据库。

> [!div class="nextstepaction"]
> [在 Azure SQL 数据库 C# 和 ADO.NET 的单一数据库中设计关系数据库](sql-database-design-first-database-csharp.md)
