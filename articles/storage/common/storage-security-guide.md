---
title: Azure 存储安全指南 | Microsoft Docs
description: 详细介绍保护 Azure 存储的多种方法，包括但不限于 RBAC、存储服务加密、客户端加密、SMB 3.0 和 Azure 磁盘加密。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 03/21/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 48ebbabca8d38db3a7c1344981f79991de29df80
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65154400"
---
# <a name="azure-storage-security-guide"></a>Azure 存储安全指南

Azure 存储提供一整套安全性功能，这些功能相辅相成，帮助开发人员构建安全的应用程序：

- 所有写入 Azure 存储的数据，使用[存储服务加密 (SSE)](storage-service-encryption.md) 进行自动加密。 有关详细信息，请参阅[宣布推出针对 Azure Blob、文件、表和队列存储的默认加密](https://azure.microsoft.com/blog/announcing-default-encryption-for-azure-blobs-files-table-and-queue-storage/)。
- Azure 存储支持使用 Azure Active Directory (Azure AD) 和基于角色的访问控制 (RBAC) 进行资源管理操作和数据操作，如下所示：   
    - 可以将作用域为存储帐户的 RBAC 角色分配给安全主体，并使用 Azure AD 为密钥管理之类的资源管理操作授权。
    - Azure AD 的集成支持的 blob 和队列数据操作。 可以将范围为订阅、资源组、存储帐户或单个容器或队列的 RBAC 角色分配给 Azure 资源的某个安全主体或托管标识。 有关详细信息，请参阅[使用 Azure Active Directory 对 Azure 存储访问进行身份验证](storage-auth-aad.md)。   
- 在应用程序和 Azure 之间传输数据时，可使用[客户端加密](../storage-client-side-encryption.md)、HTTPS 或 SMB 3.0 保护数据。  
- Azure 虚拟机使用的 OS 和数据磁盘可使用 [Azure 磁盘加密](../../security/azure-security-disk-encryption.md)进行加密。 
- 在 Azure 存储中，可以使用[共享访问签名](../storage-dotnet-shared-access-signature-part-1.md)授予数据对象的委派访问权限。

本文概述其中每项可配合 Azure 存储使用的安全功能。 提供的文章链接提供每项功能的详细信息，让你轻松地进一步探讨每个主题。

下面是本文涵盖的主题：

* [管理平面安全性](#management-plane-security) - 保护存储帐户

  管理平面包含用于管理存储帐户的资源。 此部分包括 Azure 资源管理器部署模型，以及如何使用基于角色的访问控制 (RBAC) 来控制对存储帐户的访问。 还解决了如何管理存储帐户密钥以及重新生成此类密钥。
* [数据平面安全性](#data-plane-security) - 保护对数据的访问

  此部分探讨如何在存储帐户（例如 Blob、文件、队列和表）中，允许使用共享访问签名和存储访问策略来访问实际的数据对象。 我们介绍服务级别 SAS 和帐户级别 SAS。 此外，介绍如何限制访问特定的 IP 地址（或 IP 地址范围）、如何限制用于 HTTPS 的协议，以及如何吊销共享访问签名而无需等到它过期。
* [传输中加密](#encryption-in-transit)

  此部分讨论如何在将数据传输到 Azure 存储或从中传出时提供保护。 我们将讨论 HTTPS 的建议用法，以及 SMB 3.0 针对 Azure 文件共享使用的加密。 同时还将探讨客户端加密，它可让你在将数据传输到客户端应用程序中的存储之前加密数据，以及从存储传出后解密数据。
* [静态加密](#encryption-at-rest)

  我们还将讨论存储服务加密 (SSE)，目前此项服务已针对新的和现有的存储帐户自动启用。 此外，将了解如何使用 Azure 磁盘加密，并探究磁盘加密、SSE 与客户端加密之间的基本差异和用例。 我们将简要探讨美国政府针对计算机实施的FIPS 合规性。
* 使用[存储分析](#storage-analytics)审核 Azure 存储的访问

  此部分介绍如何在存储分析日志中查找某个请求的相关信息。 我们将查看实际的分析记录数据，并了解如何分辨请求是否是利用存储帐户密钥、共享访问签名还是匿名方式发出的，以及该请求是成功还是失败。
* [使用 CORS 启用基于浏览器的客户端](#cross-origin-resource-sharing-cors)

  此部分介绍如何允许跨域资源共享 (CORS)。 我们将讨论跨域访问，以及如何使用 Azure 存储内置的 CORS 功能来处理这种访问。

## <a name="management-plane-security"></a>管理平面安全性
管理平面包含影响存储帐户本身的操作。 例如，可以创建或删除存储帐户、获取订阅中的存储帐户列表、检索存储帐户密钥，或重新生成存储帐户密钥。

创建新的存储帐户时，可以选择经典或 Resource Manager 部署模型。 在 Azure 中创建资源的经典模型只允许以孤注一掷的方式访问订阅，并访问存储帐户。

本指南着重于资源管理器模型，即创建存储帐户的建议方法。 使用 Resource Manager 存储帐户而不是提供整个订阅的访问权限，可以使用基于角色的访问控制 (RBAC) 以更高的限制级别来控制对管理平面的访问。

### <a name="how-to-secure-your-storage-account-with-role-based-access-control-rbac"></a>如何使用基于角色的访问控制 (RBAC) 来保护存储帐户
让我们讨论 RBAC 是什么及其用法。 每个 Azure 订阅都有一个 Azure Active Directory。 可以为来自该目录的用户、组和应用程序授予访问权限，以便在使用 Resource Manager 部署模型的 Azure 订阅中管理资源。 此类型的安全性称为基于角色的访问控制 (RBAC)。 若要管理此访问权限，可以使用 [Azure 门户](https://portal.azure.com/)、[Azure CLI 工具](../../cli-install-nodejs.md)、[PowerShell](/powershell/azureps-cmdlets-docs) 或 [Azure 存储资源提供程序 REST API](https://msdn.microsoft.com/library/azure/mt163683.aspx)。

通过 Resource Manager 模型，可将存储帐户置于资源组中，并使用 Azure Active Directory 控制对该特定存储帐户的管理平面的访问。 例如，可以授权特定用户访问存储帐户密钥，而其他用户可以查看有关存储帐户的信息，但无法访问存储帐户密钥。

#### <a name="granting-access"></a>授予访问权限
访问权限是通过将相应的 RBAC 角色分配给适当范围的用户、组和应用程序来授予的。 若要授予对整个订阅的访问权限，请在订阅级别分配角色。 可以将权限授予资源组本身，以此授予资源组中所有资源的访问权限。 也可以将特定角色分配给特定资源，例如存储帐户。

关于使用 RBAC 访问 Azure 存储帐户的管理操作，需了解以下要点：

* 简单而言，分配访问权限时，会将一个角色分配给希望拥有访问权限的帐户。 可控制对用于管理该存储帐户的操作的访问权限，但不能控制对帐户中数据对象的访问权限。 例如，你可以授予检索存储帐户属性（例如冗余）的权限，但不能授予检索 Blob 存储中的容器或容器中的数据的权限。
* 针对有权访问存储帐户中数据对象的用户，可为他们提供权限来读取存储帐户密钥，而该用户可以使用这些密钥来访问 Blob、队列、表和文件。
* 可将角色分配给特定的用户帐户、用户组或应用程序。
* 每个角色都有“操作”和“非操作”列表。 例如，“虚拟机参与者”角色具有“listKeys”操作，允许读取存储帐户密钥。 “参与者”具有“非操作”，例如在 Active Directory 中更新用户的访问权限。
* 存储的角色包括（但不限于）以下角色：

  * 所有者 – 他们可以管理一切，包括访问权限。
  * 参与者 – 他们可以执行所有者可执行的所有操作，但分配访问权限除外。 拥有此角色的用户可以查看和重新生成存储帐户密钥。 他们可以使用存储帐户密钥来访问数据对象。
  * 读者 – 他们可以查看有关存储帐户的信息（机密除外）。 例如，如果将存储帐户中拥有读取者权限的角色分配给某个用户，该用户就可以查看存储帐户的属性，但无法对属性进行任何更改或查看存储帐户密钥。
  * 存储帐户参与者 - 他们可以管理存储帐户 - 他们可以读取订阅的资源组和资源，以及创建和管理订阅资源组部署。 他们也可以访问存储帐户密钥，这又意味着他们可以访问数据平面。
  * 用户访问管理员 – 他们可以管理对存储帐户的用户访问权限。 例如，他们可将“读取者”权限授予特定的用户。
  * 虚拟机参与者 – 他们可以管理虚拟机，但无法管理他们连接的存储帐户。 此角色可以列出存储帐户密钥，意味着分配此角色的用户可以更新数据平面。

    为了让用户能够创建虚拟机，他们必须能够在存储帐户中创建相应的 VHD 文件。 为此，他们需要能够检索存储帐户密钥，并将它传递给创建 VM 的 API。 因此，他们必须拥有此权限才能列出存储帐户密钥。
* 定义自定义角色的功能可让你从 Azure 资源上执行的可用操作列表中编写一系列操作。
* 必须先在 Azure Active Directory 中设置用户，才能为他们分配角色。
* 可以创建一份报告，其中描述了哪个用户已使用 PowerShell 或 Azure CLI 在哪个范围为哪些对象授予/吊销哪种类型的访问权限。

#### <a name="resources"></a>资源
* [Azure Active Directory 基于角色的访问控制](../../role-based-access-control/role-assignments-portal.md)

  此文解释了 Azure Active Directory 基于角色的访问控制及其工作原理。
* [RBAC：内置角色](../../role-based-access-control/built-in-roles.md)

  此文详细说明了 RBAC 中所有可用的内置角色。
* [了解 Resource Manager 部署和经典部署](../../azure-resource-manager/resource-manager-deployment-model.md)

  此文介绍了 Resource Manager 部署和经典部署模型，并阐释了使用 Resource Manager 和资源组的优点。 本文介绍了 Azure 计算、网络和存储提供程序在 Resource Manager 模式下的工作方式。
* [使用 REST API 管理基于角色的访问控制](../../role-based-access-control/role-assignments-rest.md)

  此文说明如何使用 REST API 来管理 RBAC。
* [Azure Storage Resource Provider REST API Reference](https://msdn.microsoft.com/library/azure/mt163683.aspx)（Azure 存储资源提供程序 REST API 参考）

  该 API 参考信息介绍了可用于按编程方式管理存储帐户的 API。
* [使用资源管理器身份验证 API 访问订阅](../../azure-resource-manager/resource-manager-api-authentication.md)

  此文说明如何使用 Resource Manager API 进行身份验证。
* [Role-Based Access Control for Microsoft Azure from Ignite](https://channel9.msdn.com/events/Ignite/2015/BRK2707)

  这是第 9 频道中提供的 2015 MS Ignite 会议视频链接。 此次研讨会讨论了 Azure 中的访问管理和报告功能，并探索使用 Azure Active Directory 安全访问 Azure 订阅的最佳实践。

### <a name="managing-your-storage-account-keys"></a>管理存储帐户密钥
存储帐户密钥是由 Azure 创建的 512 位字符串，配合存储帐户名称用于访问存储于存储帐户中的数据对象，例如，Blob、表中的实体、队列消息，以及 Azure 文件共享中的文件。 控制对存储帐户密钥的访问就能控制对该存储帐户的数据平面的访问。

每个存储帐户在 [Azure 门户](https://portal.azure.com/)和 PowerShell cmdlet 中有两个密钥，分别称为“密钥 1”和“密钥 2”。 可采用多种多种方式手动重新生成这些密钥，包括（但不限于）使用 [Azure 门户](https://portal.azure.com/)、PowerShell、Azure CLI，或以编程方式使用 .NET 存储客户端库或 Azure 存储服务 REST API。

有许多原因会导致重新生成存储帐户密钥。

* 可出于安全原因而定期重新生成密钥。
* 如果某人设法侵入应用程序并检索硬编码或存储在配置文件中的密钥来为他们提供存储帐户的完整访问权限，则必须重新生成存储帐户密钥。
* 如果团队使用存储资源管理器应用程序来保留存储帐户密钥，而其中一位团队成员离职，则也需要重新生成密钥。 在某人离职后，应用程序仍将继续运行，使其他成员可以访问存储帐户。 这实际上是他们创建帐户级别共享访问签名的主要原因 – 可以改用帐户级别的 SAS，而不是将访问密钥存储在配置文件中。

#### <a name="key-regeneration-plan"></a>密钥重新生成计划
你不希望在不进行某些规划的情况下单纯重新生成你使用的密钥。 如果这样做，可能将切断对该存储帐户的所有访问权限，而这会造成严重中断。 这就是为什么要有两个密钥。 一次应该重新生成一个密钥。

重新生成密钥之前，先确保拥有依赖于存储帐户的所有应用程序列表，以及在 Azure 中使用的所有其他服务。 例如，如果 Azure 媒体服务依赖于存储帐户，则必须在重新生成密钥后将访问密钥与媒体服务重新同步。 如果使用存储资源管理器之类的任何应用程序，也必须为这些应用程序提供新的密钥。 如果 VM 的 VHD 文件存储在存储帐户中，它们将不会受到重新生成存储帐户密钥的影响。

可以在 Azure 门户中重新生成密钥。 重新生成密钥之后，最多 10 分钟后即可跨存储服务进行同步。

准备就绪后，请遵循以下常规过程，详细了解应如何更改密钥。 在本例中，假设你当前使用的是密钥 1，并想要更改所有项目以改用密钥 2。

1. 重新生成密钥 2 以确保密钥的安全。 可在 Azure 门户中执行此操作。
2. 在存储密钥所存储到的所有应用程序中，更改存储密钥以使用密钥 2 的新值。 测试并发布应用程序。
3. 在所有应用程序和服务成功启动并运行之后，重新生成密钥 1。 这可确保未明确提供新密钥的任何人都不再拥有存储帐户的访问权限。

如果当前使用密钥 2，可以使用相同的过程，但反转密钥名称。

可以过几天后迁移，更改每个应用程序来使用新的密钥并进行发布。 全部完成之后，应该返回并重新生成旧密钥，使其不再可用。

还可将存储帐户密钥作为机密放在 [Azure 密钥保管库](https://azure.microsoft.com/services/key-vault/)中，并让应用程序从此处检索该密钥。 然后，重新生成密钥并更新 Azure 密钥保管库时，就不需要重新部署应用程序，因为它们自动从 Azure 密钥保管库中选择新密钥。 请注意，可以让应用程序每次在需要密钥时读取它，或者，可以将它缓存在内存中，如果使用密钥时失败，将再次从 Azure Key Vault 中检索该密钥。

使用 Azure 密钥保管库还可以提高存储密钥的安全级别。 如果使用此方法，永远都不需要将存储密钥硬编码于配置文件中，这样将删除某人不需特定权限即可访问密钥的途径。

使用 Azure 密钥保管库的另一个优点是，还可使用 Azure Active Directory 来控制对密钥的访问。 这意味着，可以将访问权限授予少数必须从 Azure 密钥保管库检索密钥的应用程序，并了解其他应用程序在未特别授予它们权限的情况下无法访问密钥。

> [!NOTE]
> Microsoft 建议同一时间在所有应用程序中只使用一个密钥。 如果在某些地方使用密钥 1 并在其他地方使用密钥 2，将无法在没有部分应用程序失去访问的情况下轮转密钥。

#### <a name="resources"></a>资源

* [在 Azure 门户中管理存储帐户设置](storage-account-manage.md)
* [Azure Storage Resource Provider REST API Reference](https://msdn.microsoft.com/library/mt163683.aspx)（Azure 存储资源提供程序 REST API 参考）

## <a name="data-plane-security"></a>数据平面安全性
数据平面安全是指用于保护存储在 Azure 存储的数据对象（Blob、队列、表和文件）的方法。 我们已了解在传输数据期间加密数据和安全的方法，但该从何处着手来控制访问对象？

Azure 存储中数据对象的访问授权有三个选项，包括：

- 使用 Azure AD 进行容器和队列的访问授权。 进行身份验证时，Azure AD 相对于其他方法具有很多优势，包括不需要将机密存储在代码中。 有关详细信息，请参阅[使用 Azure Active Directory 对 Azure 存储访问进行身份验证](storage-auth-aad.md)。 
- 使用存储帐户密钥通过共享密钥进行访问授权。 通过共享密钥进行授权需要将存储帐户密钥存储在应用程序中，因此 Microsoft 建议尽可能改用 Azure AD。
- 使用共享访问签名授予特定时间段对特定数据对象的受控权限。

此外，对于 Blob 存储，可以通过对保存 Blob 的容器的访问级别进行相应设置，来允许对 Blob 进行公共访问。 如果将容器的访问权限设置为“Blob”或“容器”，则允许该容器中 Blob 的公共读取访问权限。 这意味着 URL 指向该容器中 Blob 的任何人都可以在浏览器中打开它，而不需要使用共享访问签名或拥有存储帐户密钥。

除通过授权限制访问外，还可使用[防火墙和虚拟网络](storage-network-security.md)来根据网络规则限制对存储帐户的访问。  通过此方法，可拒绝对公共 Internet 流量的访问，并仅向特定 Azure 虚拟网络或公共 Internet IP 地址范围授予访问权限。

### <a name="storage-account-keys"></a>存储帐户密钥
存储帐户密钥是由 Azure 创建的 512 位字符串，配合存储帐户名称用于访问存储于存储帐户中的数据对象。

例如，可读取 Blob、写入队列、创建表，还可修改文件。 可通过 Azure 门户或利用诸多 存储资源管理器应用程序中的某一种执行其中的很多操作。 还可编写代码，利用 REST API 或某个存储客户端库执行这些操作。

如[管理平面安全性](#management-plane-security)部分中所述，可通过授予对 Azure 订阅的完全访问权限，使用户可访问经典存储帐户的存储密钥。 可通过基于角色的访问控制 (RBAC)，控制借助 Azure 资源管理器模型访问存储帐户存储密钥的情况。

### <a name="how-to-delegate-access-to-objects-in-your-account-using-shared-access-signatures-and-stored-access-policies"></a>如何使用共享访问签名和存储访问策略来委派对帐户中对象的访问权限
共享访问签名是一个字符串，包含可附加到 URI 的安全令牌，可用于委派存储对象的访问权限，以及指定访问的权限和日期/时间范围等限制。

可授予用户访问 Blob、容器、队列、文件和表的权限。 使用表时，可以实际授予权限来访问表中某个范围的实体，方法是指定想要让用户有权访问的分区和行键范围。 例如，如果已使用具有地理状态的分区键存储数据，则可为某人提供仅限加州数据的访问权限。

另举一例，可以为 Web 应用程序提供 SAS 令牌，使它能够将条目写入队列，并为辅助角色应用程序提供 SAS 令牌，以便从队列中获取和处理消息。 或者，你可以为某位客户提供 SAS 令牌，使他们能够将图片上传到 Blob 存储中的容器，并为 Web 应用程序提供权限来读取这些图片。 在这两种情况下，都可实现关注点分离 – 每个应用程序只能获取执行其任务所需的访问权限。 这是通过使用共享访问签名来实现的。

#### <a name="why-you-want-to-use-shared-access-signatures"></a>使用共享访问签名的原因
为什么要使用 SAS 而不只是分发存储帐户密钥，哪一种方法更方便？ 分发存储帐户密钥就像是在存储王国中共享密钥。 它会授予完全访问权限。 其他人可以使用密钥，并将其整个音乐库上传到你的存储帐户。 他们也许还能将文件替换为受病毒感染的版本，或窃取数据。 不应草率地无限制分发对存储帐户的访问权限。

如果使用共享访问签名，可以只为客户端提供有限时间内所需的权限。 例如，如果有人将 Blob 上传到帐户，则你可以授予他们刚好足够时间的写入权限来上传 Blob（当然，这取决于 Blob 的大小）。 如果改变想法，可以撤销该访问权限。

此外，可以指定将使用 SAS 所发出的请求限制为特定的 IP 地址或 Azure 外部的 IP 地址范围。 你还可以要求使用特定协议（HTTPS 或 HTTP/HTTPS）来发出请求。 这意味着，如果只想要允许 HTTPS 流量，可以将所需的协议设置为仅限 HTTPS，并阻止 HTTP 流量。

#### <a name="definition-of-a-shared-access-signature"></a>共享访问签名的定义
共享访问签名是一组附加到指向资源的 URL 的查询参数

其中提供有关允许的访问权限的信息，以及准许该访问权限的时间长度。 下面提供了一个示例：此 URI 将提供对 Blob 的读取权限，期限为五分钟。 请注意，SAS 查询参数必须以 URL 编码，例如 %3A 表示冒号 (:)，%20 表示空格。

```
http://mystorage.blob.core.windows.net/mycontainer/myblob.txt (URL to the blob)
?sv=2015-04-05 (storage service version)
&st=2015-12-10T22%3A18%3A26Z (start time, in UTC time and URL encoded)
&se=2015-12-10T22%3A23%3A26Z (end time, in UTC time and URL encoded)
&sr=b (resource is a blob)
&sp=r (read access)
&sip=168.1.5.60-168.1.5.70 (requests can only come from this range of IP addresses)
&spr=https (only allow HTTPS requests)
&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D (signature used for the authentication of the SAS)
```

#### <a name="how-the-shared-access-signature-is-authorized-by-the-azure-storage-service"></a>Azure 存储服务对共享访问签名进行授权的方式
当存储服务收到请求时，它将获取输入查询参数，并使用与调用程序相同的方法来创建签名。 然后比较这两个签名。 如果它们相符，则存储服务可以检查存储服务版本以确保它有效、检查当前日期和时间是在指定时段内、确保请求的访问权限对应于发出的请求，等等。

以上述 URL 为例，如果 URL 指向文件而不是 Blob，此请求会失败，因为它指定共享访问签名适用于 Blob。 如果调用的 REST 命令是更新 Blob，则该命令会失败，因为共享访问签名指定只准许读访问权限。

#### <a name="types-of-shared-access-signatures"></a>共享访问签名的类型
* 服务级别 SAS 可用于访问存储帐户中的特定资源。 其中的一些示例是检索容器中的 Blob 列表、下载 Blob、更新表中的实体、将消息添加到队列，或将文件上传到文件共享。
* 帐户级别 SAS 可用于访问服务级别 SAS 可用的任何功能。 此外，它可以为服务级别 SAS 不准许的资源提供选项，例如，能够创建容器、表、队列及文件共享。 也可一次性指定对多个服务的访问权限。 例如，可以为某人提供对存储帐户中 Blob 和文件的访问权限。

#### <a name="creating-a-sas-uri"></a>创建 SAS URI
1. 可以根据需要创建 URI，每次都定义所有查询参数。

   该方法十分灵活，但如果每次都有一组类似的逻辑参数，使用存储访问策略是个不错的想法。
2. 可以针对整个容器、文件共享、表或队列创建存储访问策略。 然后使用此策略作为所创建的 SAS URI 的基础。 可以轻松吊销基于存储访问策略的权限。 可以对每个容器、队列、表或文件共享最多定义五个策略。

   例如，如果要让许多人读取特定容器中的 Blob，则可以创建存储访问策略，表示“提供读访问权限”以及每次都一样的任何其他设置。 然后，可以使用存储访问策略的设置，并指定过期日期/时间，来创建 SAS URI。 这样做的优点是不需要每次指定所有查询参数。

#### <a name="revocation"></a>吊销
假设 SAS 已泄露，或者要基于公司安全或法规遵循要求更改 SAS。 如何使用该 SAS 吊销对资源的访问权限？ 这取决于 SAS URI 的创建方式。

如果使用即席 URI，将有三个选项。 可以颁发具有短期过期策略的 SAS 令牌，然后等待 SAS 过期。 可以重命名或删除资源（假设令牌范围只限于单个对象）。 可以更改存储帐户密钥。 根据使用该存储帐户的服务数目而定，最后一个选项可能造成很大的影响，而且在没有任何规划的情况下可能达不到想要的效果。

如果使用派生自存储访问策略的 SAS，就可以通过吊销存储访问策略来删除访问权限 – 只能在它已经完全过期时进行更改，或完全删除它。 这会立即生效，并使每个使用该存储访问策略创建的 SAS 失效。 更新或删除存储访问策略可能将影响通过 SAS 访问该特定容器、文件共享、表或队列的用户，但如果要写入客户端，使得他们可在旧的 SAS 变成无效时请求一个新的 SAS，则这将可正常运行。

由于使用派生自存储访问策略的 SAS 可以立即撤销该 SAS，因此建议的最佳做法是尽可能使用存储访问策略。

#### <a name="resources"></a>资源
有关使用共享访问签名和存储访问策略的详细信息和示例，请参阅以下文章：

* 下面是参考文章。

  * [服务 SAS](https://msdn.microsoft.com/library/dn140256.aspx)

    此文提供有关使用服务级别 SAS 配合 Blob、队列、表范围和文件的示例。
  * [Constructing a Service SAS](https://msdn.microsoft.com/library/dn140255.aspx)（构造服务 SAS）
  * [Constructing an account SAS](https://msdn.microsoft.com/library/mt584140.aspx)（构造帐户 SAS）
* 这些教程介绍了如何使用 .NET 客户端库创建共享访问签名和存储访问策略。

  * [使用共享访问签名 (SAS)](../storage-dotnet-shared-access-signature-part-1.md)
  * [共享访问签名，第 2 部分：创建 SAS 并将其用于 Blob 服务](../blobs/storage-dotnet-shared-access-signature-part-2.md)

    此文包含 SAS 模型的说明、共享访问签名的示例，以及 SAS 用法最佳实践的建议。 此外介绍了如何吊销授予的权限。

* Authentication

  * [Authentication for the Azure Storage Services](https://msdn.microsoft.com/library/azure/dd179428.aspx)（Azure 存储服务的身份验证）
* 共享访问签名入门教程

  * [SAS 入门教程](https://github.com/Azure-Samples/storage-dotnet-sas-getting-started)

## <a name="encryption-in-transit"></a>传输中加密
### <a name="transport-level-encryption--using-https"></a>传输级加密 – 使用 HTTPS
为确保 Azure 存储数据安全而要采取的另一个措施是在客户端与 Azure 存储之间加密数据。 第一条建议是始终使用 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 协议，这可确保通过公共 Internet 进行安全通信。

若要获得安全的信道，在调用 REST API 或访问存储中的对象时，应该始终使用 HTTPS。 此外，除了可用于委派对 Azure 存储对象的访问权限，**共享访问签名**还可用于指定在使用共享访问签名时只能使用 HTTPS 协议，确保任何使用 SAS 令牌发出链接的人都使用正确的协议。

在调用 REST API 来访问存储帐户中的对象时，可以通过为存储帐户启用 [需要安全传输](../storage-require-secure-transfer.md) 来强制使用 HTTPS。 启用此设置后，使用 HTTP 的连接将被拒绝。

### <a name="using-encryption-during-transit-with-azure-file-shares"></a>传输期间对 Azure 文件共享使用加密
[Azure 文件](../files/storage-files-introduction.md)支持通过 SMB 3.0 进行加密，以及在使用文件 REST API 时通过 HTTPS 进行加密。 在 Azure 文件共享所在的 Azure 区域（例如本地或另一 Azure 区域）之外进行装载时，始终需要使用带加密功能的 SMB 3.0。 SMB 2.1 不支持加密，因此默认情况下只允许在 Azure 中的相同区域内连接，但可以强制使用带加密功能的 SMB 3.0，只需对存储帐户[要求安全传输](../storage-require-secure-transfer.md)即可。

带加密功能的 SMB 3.0 可以在[所有受支持的 Windows 和 Windows Server 操作系统](../files/storage-how-to-use-files-windows.md)中使用，但 Windows 7 和 Windows Server 2008 R2 除外，这二者只支持 SMB 2.1。 在 [macOS](../files/storage-how-to-use-files-mac.md) 和使用 Linux 内核 4.11 及更高版本的 [Linux](../files/storage-how-to-use-files-linux.md) 发行版上，SMB 3.0 也受支持。 针对 SMB 3.0 的加密支持也通过多个 Linux 发行版向后移植到旧版 Linux 内核。详情请参阅[了解 SMB 客户端要求](../files/storage-how-to-use-files-linux.md#smb-client-reqs)。

### <a name="using-client-side-encryption-to-secure-data-that-you-send-to-storage"></a>使用客户端加密来保护发送到存储的数据
还可通过客户端加密，帮助确保在客户端应用程序与存储之间传输时数据安全。 数据先经过加密，再传输到 Azure 存储。 从 Azure 存储检索数据时，在客户端上收到数据之后会将其解密。 即使数据在通过连接时已加密，但还是建议使用 HTTPS，因为它内置了数据完整性检查，有助于降低影响数据完整性的网络错误。

客户端加密也可用于加密静态数据，因为数据以加密形式存储。 有关更详细的信息，请参阅[静态加密](#encryption-at-rest)部分。

## <a name="encryption-at-rest"></a>静态加密
3 大 Azure 功能可提供静态加密。 Azure 磁盘加密可用于加密 IaaS 虚拟机中的 OS 和数据磁盘。 客户端加密和 SSE 都用于加密 Azure 存储中的数据。 

尽管可以使用客户端加密来加密传输中的数据（也以其加密形式存储于存储中），可能习惯在传输期间只使用 HTTPS，而且有一些方式可让数据在存储时自动加密。 有两种做法可以执行此操作 - Azure 磁盘加密和 SSE。 其中一种是用于直接加密 VM 使用的 OS 和数据磁盘上的数据，另一种用于加密写入 Azure Blob 存储的数据。

### <a name="storage-service-encryption-sse"></a>存储服务加密 (SSE)

针对所有存储帐户启用 SSE，并且不能禁用。 将数据写入 Azure 存储时，SSE 自动加密数据。 从 Azure 存储读取数据时，Azure 存储会在返回数据之前将其解密。 SSE 帮助保护数据，而无需修改代码或将代码添加到任何应用程序。

你可以使用 Microsoft 托管的密钥或你自己的自定义密钥。 Microsoft 生成托管密钥，并根据 Microsoft 内部策略的定义管理其安全存储和定期轮转。 有关使用自定义密钥的详细信息，请参阅[在 Azure 密钥保管库中使用客户托管密钥进行存储服务加密](storage-service-encryption-customer-managed-keys.md)。

SSE 自动加密所有性能层（标准和高级）、所有部署模型（Azure 资源管理器和经典）、所有 Azure 存储服务（Blob、队列、表和文件）中的数据。 

### <a name="client-side-encryption"></a>客户端加密
在介绍传输中数据加密时，我们曾提到客户端加密。 此功能可用于以编程方式加密客户端应用程序中的数据，然后通过连接发送数据以写入 Azure 存储，并在从 Azure 存储检索数据之后以编程方式解密数据。

这确实可提供传输中加密，但也会提供静态加密的功能。 尽管会在传输过程中加密数据，仍建议使用 HTTPS 来充分利用内置的数据完整性检查，帮助降低影响数据完整性的网络错误。

例如，如果 Web 应用程序将存储 Blob 和检索 Blob，而你想要让应用程序和数据尽可能保持安全，则可使用此功能。 在此情况下，请使用客户端加密。 客户端与 Azure Blob 服务之间的流量包含加密的资源，并且没有人能够解释传输中的数据并将它重组到专用 Blob。

客户端加密内置于 Java 和 .NET 存储客户端库，这些库使用 Azure 密钥保管库 API 让实现变得很简单。 加密和解密数据的程序将使用信封技术，并在每个存储对象中存储加密使用的元数据。 例如，对于 Blob，会会其存储在 Blob 元数据中；对于队列，会它添加到每个队列消息。

对于加密本身，可以生成和管理自己的加密密钥。 也可以使用 Azure 存储客户端库所生成的密钥，或者可以让 Azure 密钥保管库生成密钥。 可以将加密密钥存储在本地密钥存储中，或将它们存储在 Azure Key Vault 中。 Azure 密钥保管库可让你使用 Azure Active Directory 为特定的用户授予 Azure 密钥保管库中密码的访问权限。 这意味着，并非每个人都能读取 Azure Key Vault，以及检索用于进行客户端加密的密钥。

#### <a name="resources"></a>资源
* [在 Microsoft Azure 存储中使用 Azure 密钥保管库加密和解密 Blob](../blobs/storage-encrypt-decrypt-blobs-key-vault.md)

  此文说明如何配合 Azure 密钥保管库使用客户端加密，包括如何使用 PowerShell 来创建 KEK 并将它存储在保管库中。
* [Microsoft Azure 存储的客户端加密和 Azure 密钥保管库](../storage-client-side-encryption.md)

  本文介绍了客户端加密，并通过示例解释了如何使用存储客户端库加密和解密 4 个存储服务中的资源。 此外介绍了 Azure 密钥保管库。

### <a name="using-azure-disk-encryption-to-encrypt-disks-used-by-your-virtual-machines"></a>使用 Azure 磁盘加密来加密虚拟机所用的磁盘
Azure 磁盘加密可加密 OS 磁盘和 IaaS 虚拟机使用的数据磁盘。 对于 Windows，驱动器是使用行业标准 BitLocker 加密技术加密的。 对于 Linux，磁盘是使用 DM-Crypt 技术加密的。 这会与 Azure 密钥保管库集成，可让你控制和管理磁盘加密密钥。

在 Microsoft Azure 中启用 IaaS VM 时，该解决方案支持以下 IaaS VM 方案：

* 与 Azure Key Vault 集成
* 标准层 VM：[A、D、DS、G、GS 等系列 IaaS VM](https://azure.microsoft.com/pricing/details/virtual-machines/)
* 在 Windows 和 Linux IaaS VM 上启用加密
* 在 Windows IaaS VM 的 OS 和数据驱动器上禁用加密
* 在 Linux IaaS VM 的数据驱动器上禁用加密
* 在运行 Windows 客户端 OS 的 IaaS VM 上启用加密
* 在包含装载路径的卷上启用加密
* 在使用 mdadm 配置了磁盘条带化 (RAID) 的 Linux VM 上启用加密
* 使用 LVM 对 Linux VM 上的数据磁盘启用加密
* 在使用存储空间配置的 Windows VM 上启用加密
* 支持所有 Azure 公共区域

该解决方案不支持版本中的以下方案、功能和技术：

* 基本层 IaaS VM
* 在 Linux IaaS VM 的 OS 驱动器上禁用加密
* 使用经典 VM 创建方法创建的 IaaS VM
* 与本地密钥管理服务集成
* Azure 文件（文件共享系统）、网络文件系统 (NFS)、动态卷，以及配置了基于软件的 RAID 系统的 Windows VM


> [!NOTE]
> 下面的 Linux 发行版当前支持 Linux OS 磁盘加密：RHEL 7.2、CentOS 7.2n 和 Ubuntu 16.04。
>
>

此功能可确保虚拟机磁盘上的所有数据在 Azure 存储中静态加密。

#### <a name="resources"></a>资源
* [Azure Disk Encryption for Windows and Linux IaaS VMs](https://docs.microsoft.com/azure/security/azure-security-disk-encryption)（适用于 Windows 和 Linux IaaS VM 的 Azure 磁盘加密）

### <a name="comparison-of-azure-disk-encryption-sse-and-client-side-encryption"></a>Azure 磁盘加密、SSE 和客户端加密的比较

#### <a name="iaas-vms-and-their-vhd-files"></a>IaaS VM 及其 VHD 文件

对于 IaaS VM 使用的数据磁盘，建议使用 Azure 磁盘加密。 如果使用 Azure 市场中的映像创建包含非托管磁盘的 VM，Azure 会在 Azure 存储中[浅层复制](https://en.wikipedia.org/wiki/Object_copying)存储帐户中的映像；即使已启用 SSE，也不会加密。 创建 VM 并启动更新映像后，SSE 将开始加密数据。 出于此原因，如果想要将它们完全加密，最好在通过 Azure 市场中的映像创建的包含非托管磁盘的 VM 上使用 Azure 磁盘加密。 如果创建包含托管磁盘的 VM，则 SSE 默认情况下会使用平台管理的密钥加密所有数据。 

如果通过本地将预先加密的 VM 带入 Azure 中，就能将加密密钥上传到 Azure 密钥保管库，并继续针对使用本地的 VM 使用加密。 启用 Azure 磁盘加密即可处理此方案。

如果你有来自本地的未加密 VHD，可以作为自定义映像将它上传到库并从中预配 VM。 如果使用 Resource Manager 模板执行此操作，可以要求它在启动 VM 时打开 Azure 磁盘加密。

添加数据磁盘并装载到 VM 时，可以在该数据磁盘上打开 Azure 磁盘加密。 它先在本地加密该数据磁盘，然后经典部署模型层将会对存储进行延迟写入，如此即可加密存储内容。

#### <a name="client-side-encryption"></a>客户端加密
客户端加密是加密数据的最安全方法，因为它会在传输前加密数据。  但是，它需要使用存储将代码添加应用程序，而你可能不想要这样做。 在以上事例中，可使用 HTTPS 保护传输中的数据。 数据到达 Azure 存储后，SSE 将会对其进行加密。

通过客户端加密，可以加密表实体、消息队列和 Blob。 

客户端加密完全通过应用程序来管理。 这是最安全的方式，但要求以编程方式更改应用程序，并将密钥管理程序放在正确的位置。 如果在传输期间想要额外的安全性，并且想要将存储的数据加密，可以使用此方法。

客户端加密会在客户端上生成更多负载，必须在缩放性计划中考虑到这一点，特别是要加密并传输大量数据时。

#### <a name="storage-service-encryption-sse"></a>存储服务加密 (SSE)

SSE 由 Azure 存储管理。 SSE 不是针对传输中数据安全性提供的，但它会在数据写入 Azure 存储时加密数据。 SSE 不影响 Azure 存储性能。

可以使用 SSE 加密任何类型的存储帐户数据（块 blob、追加 blob、页 blob、表数据、队列数据和文件）。

如果使用 VHD 文件的存档或库作为创建新虚拟机的基础，可以创建新的存储帐户，然后将 VHD 文件上传到该帐户。 这些 VHD 文件由 Azure 存储加密。

如果 Azure 磁盘加密已针对 VM 中的磁盘启用，则 SSE 和 Azure 磁盘加密会加密所有新写入的数据。

## <a name="storage-analytics"></a>存储分析
### <a name="using-storage-analytics-to-monitor-authorization-type"></a>使用存储分析来监视授权类型
对于每个存储帐户，可以启用 Azure 存储分析来执行日志记录和存储指标数据。 若要检查存储帐户的性能指标，或者由于发生性能问题而需要排查存储帐户问题，这是一个绝佳工具。

可以在存储分析日志中看到的另一部分数据是其他人访问存储时使用的身份验证方法。 例如，使用 Blob 存储，可以看到他们使用的是否为共享访问签名或存储帐户密钥，或者访问的 Blob 是否为公共的。

如果要严密监视存储的访问，这很有用。 例如，在 Blob 存储中，可以将所有容器设置为专用，并通过应用程序实现 SAS 服务的用法。 然后可以定期检查日志，以了解你的 Blob 是否是使用存储帐户密钥访问的（这可能表示出现安全违规），或者 Blob 是公共的但它们不应该是公共的。

#### <a name="what-do-the-logs-look-like"></a>日志的外观
通过 Azure 门户启用存储帐户指标和日志记录后，分析数据会开始快速累积。 每个服务的日志记录与指标是分开的；只有在该存储帐户中有活动时才将写入日志，而根据设置指标的方式，会每分钟、每小时或每天记录指标。

日志将存储在存储帐户中名为 $logs 的容器的块 Blob 中。 启用存储分析后，会自动创建此容器。 创建此容器之后，无法将它删除，但可以删除其内容。

在 $logs 容器下面，每个服务都有一个文件夹，另外还有对应于年/月/日/小时的子文件夹。 在小时下面，日志带有编号。 下面是目录结构的外观：

![日志文件视图](./media/storage-security-guide/image1.png)

针对 Azure 存储发出的每个请求都将记录。 下面是日志文件的快照，其中显示了前几个字段。

![日志文件快照](./media/storage-security-guide/image2.png)

如你所见，可以使用日志来跟踪对存储帐户的各种调用。

#### <a name="what-are-all-of-those-fields-for"></a>所有这些字段的用途是什么？
在下面列出的资源中，有一篇文章提供了这些日志中许多字段的列表及其用途。 下面是依次列出的字段列表：

![日志文件中字段的快照](./media/storage-security-guide/image3.png)

我们对于 GetBlob 的条目及其授权方法感兴趣，因此需要查找操作类型为“Get-Blob”的条目，并检查请求状态（第四</sup>列）和授权类型（第八</sup>列）。

例如，在上述列表的前几列中，请求状态为“Success”且授权类型为“authenticated”。 这意味着已使用存储帐户密钥对请求进行授权。

#### <a name="how-is-access-to-my-blobs-being-authorized"></a>如何授权对 blob 的访问？
下面提供了我们感兴趣的三种用例。

1. Blob 是公共的，可使用 URL 来访问（无需共享访问签名）。 在本例中，请求状态为“AnonymousSuccess”且授权类型为“anonymous”。

   1.0;2015-11-17T02:01:29.0488963Z;GetBlob;**AnonymousSuccess**;200;124;37;**anonymous**;;mystorage…
2. Blob 是专用的且与共享访问签名配合使用。 在本例中，请求状态为“SASSuccess”且授权类型为“sas”。

   1.0;2015-11-16T18:30:05.6556115Z;GetBlob;**SASSuccess**;200;416;64;**sas**;;mystorage…
3. Blob 是专用的，可使用存储密钥来访问。 在本例中，请求状态为“**Success**”且授权类型为“**authenticated**”。

   1.0;2015-11-16T18:32:24.3174537Z;GetBlob;**Success**;206;59;22;**authenticated**;mystorage…

可使用 Microsoft Message Analyzer 查看和分析这些日志。 它包含搜索和筛选功能。 例如，你可能想要搜索 GetBlob 的实例，以查看其使用方式是否符合预期，即要确保其他人不会以不适当的方式访问存储帐户。

#### <a name="resources"></a>资源
* [存储分析](../storage-analytics.md)

  此文概述存储分析及其启用方法。
* [Storage Analytics Log Format](https://msdn.microsoft.com/library/azure/hh343259.aspx)（存储分析日志格式）

  此文介绍存储分析日志格式，并详细说明其中的可用字段，包括身份验证类型（指示请求使用的身份验证类型）。
* [在 Azure 门户中监视存储帐户](../storage-monitor-storage-account.md)

  此文说明如何配置和监视存储帐户的指标与日志记录。
* [使用 Azure 存储度量值和日志记录、AzCopy 及 Message Analyzer 进行端到端故障排除](../storage-e2e-troubleshooting.md)

  此文介绍如何使用存储分析进行故障排除，并说明如何使用 Microsoft Message Analyzer。
* [Microsoft Message Analyzer Operating Guide](https://technet.microsoft.com/library/jj649776.aspx)（Microsoft Message Analyzer 操作指南）

  本文提供 Microsoft Message Analyzer 的参考信息，包括教程、快速入门和功能摘要的链接。

## <a name="cross-origin-resource-sharing-cors"></a>跨源资源共享 (CORS)
### <a name="cross-domain-access-of-resources"></a>跨域访问资源
在某一个域中运行的 Web 浏览器对来自不同域的资源发出 HTTP 请求称为跨域 HTTP 请求。 例如，contoso.com 中的 HTML 页面将对 fabrikam.blob.core.windows.net 上托管的 jpeg 发出请求。 出于安全原因，浏览器将限制从脚本（例如 JavaScript）中初始化的跨域 HTTP 请求。 这意味着，如果 contoso.com 的页面上有一些 JavaScript 代码请求 fabrikam.blob.core.windows.net 上的该 jpeg，浏览器将拒绝此请求。

必须对 Azure 存储执行哪些操作？ 如果要在 Blob 存储中使用 Fabrikam 存储帐户存储 JSON 或 XML 数据文件等静态资产，则资产的域将为 fabrikam.blob.core.windows.net，且由于域不同，contoso.com Web 应用程序不可使用 JavaScript 访问它们。 如果尝试调用某个 Azure 存储服务（例如表存储）以返回要通过 JavaScript 客户端处理的 JSON 数据，则这一点同样适用。

#### <a name="possible-solutions"></a>可能的解决方案
一种解决方式是向 fabrikam.blob.core.windows.net 分配类似于“storage.contoso.com”的自定义域。 问题在于，只能将该自定义域分配给一个存储帐户。 如果资产存储在多个存储帐户中该怎么办？

解决此问题的另一种方法是让 Web 应用程序充当存储调用的代理。 这意味着，如果要将文件上传到 Blob 存储，Web 应用程序可以在本地写入它，然后将它复制到 Blob 存储，或者将它全部读入内存，然后将它写入 Blob 存储。 或者，你可以编写专门的 Web 应用程序（例如 Web API），以在本地上传文件并将它们写入 Blob 存储。 无论如何，都必须在确定伸缩性需求时考虑该功能。

#### <a name="how-can-cors-help"></a>CORS 有何用途？
Azure 存储允许启用 CORS – 跨域资源共享。 对于每个存储帐户，可以指定可访问该存储帐户中的资源的域。 例如上例中，可在 fabrikam.blob.core.windows.net 存储帐户中启用 CORS，并将其设置为允许访问 contoso.com。 然后，Web 应用程序 contoso.com 就可直接访问 fabrikam.blob.core.windows.net 中的资源。

要注意的一点是，CORS 允许访问，但不提供所有对存储资源的非公共访问所需的身份验证。 这意味着，如果它们是公共的，就只能访问 Blob，或者可以包含共享访问签名来提供相应的权限。 表、队列和文件没有公共访问权限并需要 SAS。

默认情况下，对所有服务禁用了 CORS。 可以使用 REST API 或存储客户端库调用某个方法来设置服务策略，以启用 CORS。 执行该操作时，会在 XML 中包含 CORS 规则。 以下示例将针对存储帐户的 Blob 服务使用“设置服务属性”操作来设置 CORS 规则。 可以使用存储客户端库或 REST API 针对 Azure 存储执行该操作。

```xml
<Cors>    
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
        <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
        <MaxAgeInSeconds>200</MaxAgeInSeconds>
    </CorsRule>
<Cors>
```

下面是每一行的含义：

* **AllowedOrigins** 指出哪些不匹配的域可以从存储服务请求并接收数据。 这意味着，contoso.com 和 fabrikam.com 可以针对特定的存储帐户向 Blob 存储请求数据。 还可将此项设置为通配符 (\*)，允许所有域访问请求。
* **AllowedMethods** 这是发出请求时可使用的方法（HTTP 请求谓词）列表。 在本示例中，只允许 PUT 和 GET。 可将此项设置为通配符 (\*)，允许使用所有方法。
* **AllowedHeaders** 这是在发出请求时原始域可指定的请求标头。 在本示例中，允许所有以 x-ms-meta-data、x-ms-meta-target 和 x-ms-meta-abc 开头的元数据标头。 通配符 (\*) 表示允许任何以指定前缀开头的标头。
* **ExposedHeaders** 告知浏览器应向请求颁发者公开的响应标头。 在本示例中，将公开任何以“x-ms-meta-”开头的标头。
* **MaxAgeInSeconds** 表示浏览器将缓存预检 OPTIONS 请求的最长时间。 （有关预检请求的详细信息，请检查下面的第一篇文章。）

#### <a name="resources"></a>资源
有关 CORS 及其启用方法的详细信息，请参阅以下资源。

* [Azure.com 上对 Azure 存储服务的跨域资源共享 (CORS) 支持](../storage-cors-support.md)

  此文概要讲述了 CORS，并介绍了如何为不同的存储服务设置规则。
* [Cross-Origin Resource Sharing (CORS) Support for the Azure Storage Services on MSDN](https://msdn.microsoft.com/library/azure/dn535601.aspx)（MSDN 上对 Azure 存储服务的跨域资源共享 (CORS) 支持）

  这是有关对 Azure 存储服务的 CORS 支持的参考文档。 其中提供了适用于每个存储服务的文章链接，并提供示例演示，解释 CORS 文件中的每个元素。
* [Microsoft Azure 存储：CORS 简介](https://blogs.msdn.com/b/windowsazurestorage/archive/2014/02/03/windows-azure-storage-introducing-cors.aspx)

  这是宣布推出 CORS 并演示其用法的第一篇博客文章的链接。

## <a name="frequently-asked-questions-about-azure-storage-security"></a>有关 Azure 存储安全性的常见问题
1. **如果无法使用 HTTPS 协议，该如何验证传输到 Azure 存储或从中传出的 Blob 的完整性？**

   如果出于任何原因，需要使用 HTTP 而不是 HTTPS，且你正在使用块 Blob，则可以使用 MD5 检查，帮助验证正在传输的 Blob 的完整性。 这会有助于防止网络/传输层错误，但不一定可帮助防止中间攻击。

   如果可以使用提供传输级安全的 HTTPS，则使用 MD5 检查就很多余且不必要。

   有关详细信息，请查看 [Azure Blob MD5 Overview](https://blogs.msdn.com/b/windowsazurestorage/archive/2011/02/18/windows-azure-blob-md5-overview.aspx)（Azure Blob MD5 概述）。
2. **美国政府实施的 FIPS 合规性要求是怎样的？**

   美国联邦信息处理标准 (FIPS) 定义了美国联邦政府计算机系统批准使用的加密算法，以保护敏感数据。 如果在 Windows 服务器或桌面上启用 FIPS 模式，将告知 OS 仅应使用经 FIPS 验证的加密算法。 如果某个应用程序使用不合规的算法，即表示该应用程序违规。 使用 .NET Framework 4.5.2 或更高版本，应用程序可在计算机处于 FIPS 模式时自动切换加密算法来使用符合 FIPS 的算法。

   Microsoft 允许每个客户决定是否启用 FIPS 模式。 我们相信，客户没有充分的理由违反政府法规，不按默认启用 FIPS 模式。

### <a name="resources"></a>资源
* [为何建议不再使用“FIPS 模式”](https://blogs.technet.microsoft.com/secguide/2014/04/07/why-were-not-recommending-fips-mode-anymore/)

  此博客文章提供 FIPS 概述，并说明他们为什么默认不启用 FIPS 模式。
* [FIPS 140 Validation](https://technet.microsoft.com/library/cc750357.aspx)（FIPS 140 验证）

  此文提供有关 Microsoft 产品和加密模块如何遵守美国联邦政府 FIPS 标准的信息。
* [“系统加密：使用 FIPS 兼容的算法来加密、哈希和签名”在 Windows XP 和更高版本的 Windows 中的安全设置影响](https://support.microsoft.com/kb/811833)

  此文介绍如何在较旧版 Windows 计算机中使用 FIPS 模式。
