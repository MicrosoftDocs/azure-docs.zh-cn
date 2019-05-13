---
title: 使用 Azure 存储资源管理器设置 Data Lake Storage Gen2 的权限
description: 本操作指南介绍如何使用 Azure 存储资源管理器对 Azure Data Lake Storage Gen2 支持的存储帐户中的文件和目录设置权限。
services: storage
author: normesta
ms.custom: mvc
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: quickstart
ms.date: 12/11/2018
ms.author: normesta
ms.openlocfilehash: 5f0211765c96cad668abaad7d42da87ec88298c3
ms.sourcegitcommit: c53a800d6c2e5baad800c1247dce94bdbf2ad324
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64939310"
---
# <a name="set-file-and-directory-level-permissions-using-azure-storage-explorer-with-azure-data-lake-storage-gen2"></a>使用 Azure 存储资源管理器和 Azure Data Lake Storage Gen2 设置文件和目录级别权限

Azure Data Lake Storage Gen2 中存储的文件支持细化权限和访问控制列表 (ACL) 管理。 通过细化权限和 ACL 管理，可以在非常精细的级别上管理对数据的访问权限。

本文介绍如何使用 Azure 存储资源管理器执行以下操作：

> [!div class="checklist"]
> * 设置文件级别权限
> * 设置目录级别权限
> * 将用户或组添加到访问控制列表

## <a name="prerequisites"></a>先决条件

为了以最佳效果描述该过程，我们要求你完成我们的 [Azure 存储资源管理器快速入门](data-lake-storage-Explorer.md)。 这可确保你的存储帐户处于最合适的状态（文件系统已创建并且数据已上传到其中）。

## <a name="managing-access"></a>管理访问权限

你可以在文件系统的根目录中设置权限。 为此，你必须使用有权执行此操作的个人帐户登录到 Azure 存储资源管理器（而不是使用连接字符串）。 请右键单击文件系统，然后选择“管理权限”，打开“管理权限”对话框。

![Microsoft Azure 存储资源管理器 - 管理目录访问权限](media/storage-quickstart-blobs-storage-Explorer/manageperms.png)

“管理权限”对话框可以管理所有者和所有者组的权限。 它还可以将新用户和组添加访问控制列表中，然后你可以管理其权限。

要将新用户或组添加到访问控制列表中，请选择“添加用户或组”字段。

输入要添加到列表中的相应 Azure Active Directory (AAD) 条目，然后选择“添加”。

用户或组随即出现在“用户和组:”字段中，然后便可开始管理其权限。

> [!NOTE]
> 建议的最佳做法是在 AAD 中创建安全组并维护组而不是单个用户的权限。 有关此建议以及其他最佳做法的详细信息，请参阅 [Data Lake Storage Gen2 的最佳做法](data-lake-storage-best-practices.md)。

有两类权限可以分配：访问 ACL 和默认 ACL。

* **访问权限**：访问 ACL 控制对某个对象的访问权限。 文件和目录都具有访问 ACL。

* **默认**：与目录关联的 ACL 模板，用于确定在该目录下创建的任何子项的访问 ACL。 文件没有默认 ACL。

在这两个类别中，你可以对文件或目录分配三种权限：“读取”、“写入”和“执行”。

>[!NOTE]
> 在此处进行选择不会对目录中任何当前存在的项设置权限。 如果文件已存在，则必须转到每个项并手动设置权限。

由于可以管理各个目录以及各个文件的权限，因此可以实现细化访问控制。 管理目录和文件的权限的过程与上述过程相同。 右键单击要管理权限的文件或目录，然后按照相同的过程进行操作。

## <a name="next-steps"></a>后续步骤

本操作指南介绍了如何使用“Azure 存储资源管理器”设置文件和目录的权限。 要了解有关 ACL 的详细信息，包括默认 ACL、访问 ACL、其行为及其相应的权限，请继续阅读有关该主题的概念性文章。

> [!div class="nextstepaction"]
> [Azure Data Lake Storage Gen2 中的访问控制](data-lake-storage-access-control.md)
