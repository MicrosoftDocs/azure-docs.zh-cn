---
title: Azure Data Lake Storage Gen2 中的访问控制概述 | Microsoft Docs
description: 了解 Azure Data Lake Storage Gen2 中访问控制的工作原理
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: 0b8139f11f937ddae30e25f4153e35287289a4d1
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233997"
---
# <a name="access-control-in-azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2 中的访问控制

Azure 数据湖存储第 2 代实现支持 Azure 基于角色的访问控制 (RBAC) 和类似的 POSIX 访问控制列表 (Acl) 的访问控制模型。 本文汇总了 Data Lake Storage Gen2 访问控制模型的基本知识。

<a id="azure-role-based-access-control-rbac" />

## <a name="role-based-access-control"></a>基于角色的访问控制

RBAC 使用角色分配来有效地将应用的权限集*安全主体*。 一个*安全主体*是一个对象，表示用户、 组、 服务主体或定义在 Azure Active Directory (AD) 对 Azure 资源的访问请求的托管的标识。

通常情况下，这些 Azure 资源限制为顶级资源 (例如：Azure 存储帐户）。 就 Azure 存储以及 Azure Data Lake Storage Gen2 而言，此机制已扩展到文件系统资源。

若要了解如何将角色分配给你的存储帐户的作用域中的安全主体，请参阅[进行身份验证访问 Azure blob 和队列使用 Azure Active Directory](https://docs.microsoft.com/azure/storage/common/storage-auth-aad?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

### <a name="the-impact-of-role-assignments-on-file-and-directory-level-access-control-lists"></a>文件和目录级别的访问控制列表的角色分配的影响

使用 RBAC 角色分配是强大的机制来控制访问权限，而它是相对于 Acl 的非常精确进行细化管理的机制。 RBAC 的最小粒度为文件系统级别，我们将它的优先级评为高于 ACL。 因此，如果向文件系统的作用域中的安全主体分配角色，该安全主体具有在该文件系统中，而不考虑 ACL 分配与该角色的所有目录和文件关联的授权级别。

当安全主体授予通过 RBAC 数据权限[内置角色](https://docs.microsoft.com/azure/storage/common/storage-auth-aad?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#built-in-rbac-roles-for-blobs-and-queues)，或通过自定义角色，这些权限都应首先评估在授权请求时。 如果请求的操作获得授权时的安全主体的 RBAC 分配授权是立即解决的和无需额外执行 ACL 检查。 或者，如果安全主体不具有 RBAC 分配，或请求的操作与分配的权限不匹配，然后 ACL 执行检查以确定是否授权的安全主体执行请求的操作。

> [!NOTE]
> 如果安全主体已分配存储 Blob 数据所有者内置角色分配，则安全主体将被视为*超级用户*并授予对所有的转变操作，包括设置完全访问权限目录和文件，它们不是所有者的目录或文件，以及 Acl 所有者。 超级用户访问是唯一获准的更改资源所有者的方式。

## <a name="shared-key-and-shared-access-signature-sas-authentication"></a>共享密钥和共享访问签名 (SAS) 身份验证

Azure 数据湖存储第 2 代支持共享密钥和 SAS 方法进行身份验证。 这些身份验证方法的一个特点是没有标识与调用方关联，因此不能执行安全主体基于权限的授权。

就共享密钥这一种方法而言，调用方有效地获得了“超级用户”访问权限，这意味着对所有资源上所有操作（包括设置所有者和更高 ACL）的完全访问权限。

SAS 令牌本身就包含允许的权限。 它包含的权限有效地应用到所有授权决策，但不执行额外的 ACL 检查。

## <a name="access-control-lists-on-files-and-directories"></a>文件和目录上的访问控制列表

可以将安全主体相关联的文件和目录访问级别。 在中捕获这些关联*访问控制列表 (ACL)*。 每个文件和存储帐户中的目录具有访问控制列表。

如果角色分配给安全主体在存储帐户级别时，可以使用访问控制列表授予该安全主体提升到特定文件和目录的访问权限。

不能使用访问控制列表来提供比由角色分配授予的级别低的访问级别。 例如，如果你将分配[存储 Blob 数据参与者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor-preview)角色的安全主体，则无法使用访问控制列表来防止该安全主体中将写入到一个目录。

### <a name="set-file-and-directory-level-permissions-by-using-access-control-lists"></a>设置文件和目录级别权限使用访问控制列表

若要设置文件和目录级别权限，请参阅以下文章：

|如果你想要使用此工具：    |请参阅以下文章：    |
|--------|-----------|
|Azure 存储资源管理器    |[使用 Azure 存储资源管理器和 Azure Data Lake Storage Gen2 设置文件和目录级别权限](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-how-to-set-permissions-storage-explorer)|
|REST API    |[Path - Update](https://docs.microsoft.com/rest/api/storageservices/datalakestoragegen2/path/update)|

> [!IMPORTANT]
> 如果安全主体是否*服务*主体，务必要使用的服务主体的对象 ID 和不相关的应用注册的对象 ID。 若要获取服务主体的对象 ID 打开 Azure CLI，以及如何将此命令： `az ad sp show --id <Your App ID> --query objectId`。 请务必替换`<Your App ID>`应用注册的应用 id 的占位符。

### <a name="types-of-access-control-lists"></a>类型的访问控制列表

有两种类型的访问控制列表：*访问 Acl*并*默认 Acl*。

访问 ACL 控制对某个对象的访问权限。 文件和目录都具有访问 ACL。

默认 Acl 是与目录相关联的 Acl，以确定该目录下创建任何子项的访问 Acl 的模板。 文件没有默认 ACL。

访问 ACL 和默认 ACL 具有相同的结构。

访问 ACL 和默认 ACL 具有相同的结构。

> [!NOTE]
> 更改父级的默认 ACL 不影响现有子项的访问 ACL 或默认 ACL。

### <a name="levels-of-permission"></a>权限级别

文件系统对象权限为“读取”、“写入”和“执行”，可对下表中所示的文件和目录使用这些权限：

|            |    文件     |   Directory |
|------------|-------------|----------|
| **读取 (R)** | 可以读取文件内容 | 需有“读取”和“执行”权限才能列出目录内容 |
| **写入 (W)** | 可以在文件中写入或追加内容 | 需有“写入”和“执行”权限才能在目录中创建子项 |
| **执行 (X)** | 不表示 Data Lake Storage Gen2 上下文中的任何内容 | 需要遍历目录的子项 |

#### <a name="short-forms-for-permissions"></a>权限的简短形式

**RWX** 用于表示“读取 + 写入 + 执行”。 还有更精简的数字形式，“读取=4”，“写入=2”，“执行=1”，其总和表示各种不同的权限。 下面是一些示例。

| 数字形式 | 简短形式 |      含义     |
|--------------|------------|------------------------|
| 7            | `RWX`        | RWX |
| 5            | `R-X`        | 读取 + 执行         |
| 4            | `R--`        | 读取                   |
| 0            | `---`        | 无权限         |

#### <a name="permissions-inheritance"></a>权限继承

在 Data Lake Storage Gen2 使用的 POSIX 样式的模型中，项的权限存储在项本身中。 换而言之，如果是在已创建子项后设置的权限，则不能从父项继承项的权限。 只有于创建子项前在父项上设置了默认权限时，才能继承权限。

### <a name="common-scenarios-related-to-permissions"></a>与权限相关的常见方案

下表列出了一些常见的方案，以帮助您了解需要哪些权限才能对存储帐户执行某些操作。

|    Operation             |    /    | Oregon/ | Portland/ | Data.txt     |
|--------------------------|---------|----------|-----------|--------------|
| Read Data.txt            |   `--X`   |   `--X`    |  `--X`      | `R--`          |
| Append to Data.txt       |   `--X`   |   `--X`    |  `--X`      | `RW-`          |
| Delete Data.txt          |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| Create Data.txt          |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| List /                   |   `R-X`   |   `---`    |  `---`      | `---`          |
| List /Oregon/           |   `--X`   |   `R-X`    |  `---`      | `---`          |
| List /Oregon/Portland/  |   `--X`   |   `--X`    |  `R-X`      | `---`          |

> [!NOTE]
> 只要以上两个条件成立，删除文件时就不需要文件的写入权限。

### <a name="users-and-identities"></a>用户和标识

每个文件和目录都有这些标识的不同权限：

- 拥有用户
- 拥有组
- 命名用户
- 命名组
- 命名服务主体
- 名为管理的标识
- 所有其他用户

用户和组的标识是 Azure Active Directory (Azure AD) 标识。 因此，除非另行说明，否则*用户*，在数据湖存储第 2 代的上下文，可以向 Azure AD 用户，请参阅中，服务主体、 托管标识或安全组。

#### <a name="the-owning-user"></a>拥有用户

创建项的用户自动成为该项的拥有用户。 拥有用户可以：

* 更改所拥有文件的权限。
* 更改所拥有文件的拥有组，前提是该拥有用户也是目标组的成员。

> [!NOTE]
> 拥有用户无法更改某个文件或目录的拥有用户。 只有超级用户可以更改文件或目录的拥有用户。

#### <a name="the-owning-group"></a>拥有组

在 POSIX ACL 中，每个用户都与“主组”关联。 例如，用户"Alice"可能属于"finance"组。 Alice 还可能属于多个组，但始终有一个组指定为她的主组。 在 POSIX 中，当 Alice 创建文件时，该文件的拥有组设置为她的主组，在本例中为“finance”。 否则，所有者组的行为类似于为其他用户/组分配的权限。

##### <a name="assigning-the-owning-group-for-a-new-file-or-directory"></a>为新的文件或目录分配拥有组

* **情况 1**：根目录“/”。 此目录是在创建 Data Lake Storage Gen2 文件系统时创建的。 在本例中，如果使用的是 OAuth，则拥有组设置为创建文件系统的用户。 如果文件系统创建使用共享密钥、 帐户 SAS 或服务 SAS，则所有者和拥有组设置为 **$superuser**。
* **情况 2**（所有其他情况）：创建新项时，从父目录复制拥有组。

##### <a name="changing-the-owning-group"></a>更改拥有组

拥有组可由以下用户更改：
* 任何超级用户。
* 拥有用户，前提是该拥有用户也是目标组的成员。

> [!NOTE]
> 拥有组无法更改某个文件或目录的 ACL。  虽然拥有组设置为创建在根目录下的情况下的帐户的用户**案例 1**更高版本，单个用户帐户不是有效的权限通过负责人组提供。 可以将此权限分配给有效的用户组（如果适用）。

### <a name="access-check-algorithm"></a>访问检查算法

下面的伪代码表示存储帐户的访问检查算法。

```
def access_check( user, desired_perms, path ) : 
  # access_check returns true if user has the desired permissions on the path, false otherwise
  # user is the identity that wants to perform an operation on path
  # desired_perms is a simple integer with values from 0 to 7 ( R=4, W=2, X=1). User desires these permissions
  # path is the file or directory
  # Note: the "sticky bit" isn't illustrated in this algorithm
  
# Handle super users.
  if (is_superuser(user)) :
    return True

# Handle the owning user. Note that mask isn't used.
entry = get_acl_entry( path, OWNER )
if (user == entry.identity)
    return ( (desired_perms & entry.permissions) == desired_perms )

# Handle the named users. Note that mask IS used.
entries = get_acl_entries( path, NAMED_USER )
for entry in entries:
    if (user == entry.identity ) :
        mask = get_mask( path )
        return ( (desired_perms & entry.permissions & mask) == desired_perms)

# Handle named groups and owning group
member_count = 0
perms = 0
entries = get_acl_entries( path, NAMED_GROUP | OWNING_GROUP )
for entry in entries:
if (user_is_member_of_group(user, entry.identity)) :
    member_count += 1
    perms | =  entry.permissions
if (member_count>0) :
return ((desired_perms & perms & mask ) == desired_perms)

# Handle other
perms = get_perms_for_other(path)
mask = get_mask( path )
return ( (desired_perms & perms & mask ) == desired_perms)
```

#### <a name="the-mask"></a>掩码

如访问检查算法中所示，掩码会限制对命名用户、拥有组和命名组的访问权限。  

> [!NOTE]
> 对于新的 Data Lake Storage Gen2 文件系统，根目录（“/”）的访问 ACL 的掩码默认为：目录 750，文件 640。 文件不接收 X 位，因为它与只存储系统中的文件无关。
>
> 可能会在每次调用时指定掩码。 这就使不同的使用系统（例如群集）能够为文件操作使用不同的有效掩码。 如果根据特定请求指定了掩码，则该掩码完全替代默认掩码。

#### <a name="the-sticky-bit"></a>粘滞位

粘滞位是 POSIX 文件系统更高级的一项功能。 在 Data Lake Storage Gen2 的上下文中，不太可能需要粘滞位。 总之，如果目录上已启用粘滞位，子项只能由子项的拥有用户删除或重命名。

粘滞位不是在 Azure 门户中所示。

### <a name="default-permissions-on-new-files-and-directories"></a>新文件和目录的默认权限

在现有目录下创建新文件或目录时，父目录的默认 ACL 会确定：

- 子目录的默认 ACL 和访问 ACL。
- 子文件的访问 ACL（文件没有默认 ACL）。

#### <a name="umask"></a>umask

创建文件或目录时，umask 用于修改默认 ACL 在子项上的设置方式。 umask 是父目录上一个 9 位的值，它包含“拥有用户”、“拥有组”和“其他”的 RWX 值。

Azure Data Lake Storage Gen2 的 umask 是设置为 007 的常量值。 此值将转换为：

| umask 组件     | 数字形式 | 简短形式 | 含义 |
|---------------------|--------------|------------|---------|
| umask.owning_user   |    0         |   `---`      | 对于拥有用户，将父项的默认 ACL 复制到子项的访问 ACL | 
| umask.owning_group  |    0         |   `---`      | 对于拥有组，将父项的默认 ACL 复制到子项的访问 ACL | 
| umask.other         |    7         |   `RWX`      | 对于其他，删除子项的访问 ACL 的所有权限 |

Azure Data Lake Storage Gen2 的 umask 值实际上表示无论默认 ACL 作何指示，默认情况下永远不会在新子项上传输“其他”的值。 

以下伪代码显示了在为子项创建 ACL 时如何应用 umask。

```
def set_default_acls_for_new_child(parent, child):
    child.acls = []
    for entry in parent.acls :
        new_entry = None
        if (entry.type == OWNING_USER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_user))
        elif (entry.type == OWNING_GROUP) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_group))
        elif (entry.type == OTHER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.other))
        else :
            new_entry = entry.clone(perms = entry.perms )
        child_acls.add( new_entry )
```

## <a name="common-questions-about-acls-in-data-lake-storage-gen2"></a>有关 Data Lake Storage Gen2 中 ACL 的常见问题

### <a name="do-i-have-to-enable-support-for-acls"></a>是否必须启用 ACL 的支持？

不。 通过 Acl 的访问控制可用于存储帐户，只要分层 Namespace (HNS) 功能是打开。

即使关闭了 HNS 功能，Azure RBAC 授权规则仍适用。

### <a name="what-is-the-best-way-to-apply-acls"></a>应用 ACL 的最佳方式是什么？

始终将 Azure AD 安全组用作 ACL 中分配的主体。 拒绝直接分配各个用户或服务主体。 使用此结构，你可以添加和删除用户或服务主体，不需要向整个目录结构重新应用 ACL。 而只需要从相应的 Azure AD 安全组添加或删除它们。 请记住，ACL 不是继承的，重新应用 ACL 需要更新每个文件和子目录上的 ACL。 

### <a name="which-permissions-are-required-to-recursively-delete-a-directory-and-its-contents"></a>以递归方式删除目录及其内容需要哪些权限？

- 调用方需要拥有“超级用户”权限，

或

- 父目录必须拥有“写入 + 执行”权限。
- 要删除的目录及其中的每个目录都需要“读取 + 写入 + 执行”权限。

> [!NOTE]
> 删除目录中的文件不需要具有“写入”权限。 另外，永远无法删除根目录“/”。

### <a name="who-is-the-owner-of-a-file-or-directory"></a>谁是文件或目录的所有者？

文件或目录的创建者就是所有者。 就根目录而言，这就是创建文件系统的用户的标识。

### <a name="which-group-is-set-as-the-owning-group-of-a-file-or-directory-at-creation"></a>创建文件或目录时将哪个组设置为其拥有组？

拥有组是从创建新文件或目录的父目录的拥有组复制而来的。

### <a name="i-am-the-owning-user-of-a-file-but-i-dont-have-the-rwx-permissions-i-need-what-do-i-do"></a>文件的拥有用户没有所需的 RWX 权限， 我该怎么办？

拥有用户只需更改文件的权限，即可自动获得所需的任何 RWX 权限。

### <a name="why-do-i-sometimes-see-guids-in-acls"></a>为什么我时候在 ACL 中看到了 GUID？

如果条目表示一个用户且该用户不再位于 Azure AD 中，则显示 GUID。 当用户离职，或者其帐户已在 Azure AD 中删除时，往往会发生这种情况。 此外，服务主体和安全组没有用于标识它们的用户主体名称 (UPN)，因此用它们的 OID 属性（即一个 GUID）来表示它们。

### <a name="how-do-i-set-acls-correctly-for-a-service-principal"></a>如何为服务主体正确设置 ACL？

为服务主体定义 ACL 时，必须使用所创建应用注册的服务主体的对象 ID (OID)。 请务必注意，注册的应用在特定的 Azure AD 租户中具有独立的服务主体。 注册的应用会在 Azure 门户中显示一个 OID，但服务主体具有另一个（不同的）OID。

若要获取其 OID 对应的应用注册的服务主体，可以使用`az ad sp show`命令。 指定应用程序 ID 作为参数。 下面是一个示例对应于应用程序 ID 的应用注册的服务主体获取 OID = 18218b12-1895年-43e9-ad80-6e8fc1ea88ce。 在 Azure CLI 中运行以下命令：

`az ad sp show --id 18218b12-1895-43e9-ad80-6e8fc1ea88ce --query objectId
<<OID will be displayed>>`

获取服务主体的正确 OID 后，转到存储资源管理器的“管理访问权限”页，以添加该 OID 并为其分配适当的的权限。 请务必选择“保存”。

### <a name="does-data-lake-storage-gen2-support-inheritance-of-acls"></a>Data Lake Storage Gen2 是否支持 ACL 继承？

Azure RBAC 分配支持继承。 从订阅、资源组和存储帐户资源往下分配到文件系统资源。

ACL 不支持继承。 但是，可以使用默认 ACL 来设置父目录下创建的子目录和文件的 ACL。 

### <a name="where-can-i-learn-more-about-posix-access-control-model"></a>在哪里可以了解 POSIX 访问控制模型的详细信息？

* [Linux 上的 POSIX 访问控制列表](https://www.linux.com/news/posix-acls-linux)
* [HDFS 权限指南](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html)
* [POSIX FAQ](https://www.opengroup.org/austin/papers/posix_faq.html)
* [POSIX 1003.1 2008](https://standards.ieee.org/findstds/standard/1003.1-2008.html)
* [POSIX 1003.1 2013](https://pubs.opengroup.org/onlinepubs/9699919799.2013edition/)
* [POSIX 1003.1 2016](https://pubs.opengroup.org/onlinepubs/9699919799.2016edition/)
* [Ubuntu 上的 POSIX ACL](https://help.ubuntu.com/community/FilePermissionsACLs)
* [在 Linux 上使用访问控制列表 (ACL)](https://bencane.com/2012/05/27/acl-using-access-control-lists-on-linux/)

## <a name="see-also"></a>另请参阅

* [Azure Data Lake Storage Gen2 概述](../blobs/data-lake-storage-introduction.md)
