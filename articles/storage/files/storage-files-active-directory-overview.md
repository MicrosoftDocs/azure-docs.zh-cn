---
title: 通过 SMB 为 Azure 文件进行 Azure Active Directory 身份验证概述（预览）- Azure 存储
description: Azure 文件使用 Azure Active Directory (Azure AD) 域服务，支持通过 SMB（服务器消息块）实现的基于标识的身份验证（预览）。 然后，已加入域的 Windows 虚拟机 (VM) 可使用 Azure AD 凭据访问 Azure 文件共享。
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 09/19/2018
ms.author: rogarana
ms.openlocfilehash: af903ce3ce7cfa165c278e415827dda36630e7b4
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65209729"
---
# <a name="overview-of-azure-active-directory-authentication-over-smb-for-azure-files-preview"></a>通过 SMB 为 Azure 文件进行 Azure Active Directory 身份验证概述（预览）
[!INCLUDE [storage-files-aad-auth-include](../../../includes/storage-files-aad-auth-include.md)]

若要了解如何通过 SMB 为 Azure 文件启用 Azure AD 身份验证，请参阅[通过 SMB 为 Azure 文件启用 Azure Active Directory 身份验证（预览）](storage-files-active-directory-enable.md)。

## <a name="glossary"></a>术语表 
了解与通过 SMB 为 Azure 文件进行 Azure AD 身份验证相关的一些关键术语非常有用：

-   **Azure Active Directory (Azure AD)**  
    Azure Active Directory (Azure AD) 是 Microsoft 提供的多租户、基于云的目录和标识管理服务。 Azure AD 将核心目录服务、应用程序访问管理和标识保护融入一个解决方案中。 有关详细信息，请参阅[什么是 Azure Active Directory？](../../active-directory/fundamentals/active-directory-whatis.md)

-   **Azure AD 域服务**  
    Azure AD 域服务提供托管域服务，例如域加入、组策略、LDAP、Kerberos/NTLM 身份验证。 这些服务与 Windows Server Active Directory 完全兼容。 有关详细信息，请参阅 [Azure Active Directory (AD) 域服务](../../active-directory-domain-services/active-directory-ds-overview.md)。

-   **Azure 基于角色的访问控制 (RBAC)**  
    Azure 基于角色的访问控制 (RBAC) 可用于对 Azure 进行细致的访问管理。 使用 RBAC，可通过向用户授予执行其作业所需的最少权限来管理对资源的访问权限。 有关 RBAC 的详细信息，请参阅[什么是 Azure 中基于角色的访问控制 (RBAC)？](../../role-based-access-control/overview.md)

-   **Kerberos 身份验证**

    Kerberos 是一种身份验证协议，用于验证用户或主机的标识。 有关 Kerberos 的详细信息，请参阅 [Kerberos 身份验证概述](https://docs.microsoft.com/windows-server/security/kerberos/kerberos-authentication-overview)。

-  **服务器消息块 (SMB) 协议**  
    SMB 是行业标准网络文件共享协议。 也称为通用 Internet 文件系统或 CIFS。 有关 SMB 的详细信息，请参阅 [Microsoft SMB 协议和 CIFS协议概述](https://docs.microsoft.com/windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview)。

## <a name="advantages-of-azure-ad-authentication"></a>Azure AD 身份验证的优势
通过 SMB 为 Azure 文件进行 Azure AD 身份验证与采用共享密钥身份验证相比，具有以下几个优势：

-   **使用 Azure AD 将传统的基于标识的文件共享访问体验扩展到云**  
    如果计划将应用程序“直接迁移”到云，用 Azure 文件替代传统的文件服务器，则可能需要使用 Azure AD 对应用程序进行身份验证以访问文件数据。 Azure 文件支持使用已加入域的 VM 中的 Azure AD 凭据通过 SMB 访问文件共享、目录或文件。 还可选择将所有本地 Active Directory 对象同步到 Azure AD，以保留用户名、密码和其他组分配。

-   **对 Azure 文件共享强制实施粒度访问控制**  
    通过 SMB 进行 Azure AD 身份验证，可在共享、目录或文件级别为特定标识授予权限。 例如，假设多个团队使用一个 Azure 文件共享进行项目协作。 你可以向所有团队授予访问非敏感目录的权限，而仅授权财务团队访问含敏感财务数据的目录。 

-   **备份 ACL 以及你的数据**  
    可使用 Azure 文件备份现有的本地文件共享。 将文件共享通过 SMB 备份到 Azure 文件时，Azure 文件会保留 ACL 和你的数据。

## <a name="how-it-works"></a>工作原理
Azure 文件使用 Azure AD 域服务，支持通过已加入域的 VM 中的 Azure AD 凭据进行 Kerberos 身份验证。 在将 Azure AD 用于 Azure 文件之前，必须先启用 Azure AD 域服务，再从计划访问文件数据的 VM 加入域。 已加入域的 VM 必须驻留在与 Azure AD 域服务位于同一虚拟网络 (VNET)。 

当与 VM 上运行的应用程序相关联的标识尝试访问 Azure 文件中的数据时，该请求将发送到 Azure AD 域服务以验证标识。 如果身份验证成功，Azure AD 域服务会返回一个 Kerberos 令牌。 应用程序发送一个包含该 Kerberos 令牌的请求，Azure 文件使用该令牌对该请求授权​​。 Azure 文件仅接收令牌，不保存 Azure AD 凭据。

![屏幕截图显示了通过 SMB 进行 Azure AD 身份验证的关系图](media/storage-files-active-directory-overview/azure-active-directory-over-smb-for-files-overview.png)

### <a name="enable-azure-ad-authentication-over-smb"></a>启用通过 SMB 进行 Azure AD 身份验证
可对 2018 年 9 月 24 日之后创建的新存储帐户和现有存储帐户启用通过 SMB 为 Azure 文件进行 Azure AD 身份验证这一功能。 

在启用通过 SMB 进行 Azure AD 身份验证之前，请验证是否已为与存储帐户关联的主 Azure AD 租户部署了 Azure AD 域服务。 如果尚未设置 Azure AD 域服务，请按照[使用 Azure 门户启用 Azure Active Directory 域服务](../../active-directory-domain-services/active-directory-ds-getting-started.md)中提供的分步指导进行操作。

部署 Azure AD 域服务通常需要 10 至 15 分钟。 部署 Azure AD 域服务后，即可启用通过 SMB 为 Azure 文件进行 Azure AD 身份验证这一功能。 有关详细信息，请参阅[通过 SMB 为 Azure 文件启用 Azure Active Directory 身份验证（预览）](storage-files-active-directory-enable.md)。 

### <a name="configure-share-level-permissions-for-azure-files"></a>为 Azure 文件配置共享级别权限
启用 Azure AD 身份验证后，可以为 Azure AD 标识配置自定义 RBAC 角色，并为存储帐户中的任何文件共享分配访问权限。

在已加入域的 VM 上运行的应用程序尝试装载 Azure 文件共享或者访问目录或文件时，系统会验证应用程序的 Azure AD 凭据以确保共享级别权限和 NTFS 权限正确。 有关配置共享级别权限的信息，请参阅[启用通过 SMB 进行 Azure Active Directory 身份验证（预览）](storage-files-active-directory-enable.md)。

### <a name="configure-directory--or-file-level-permissions-for-azure-files"></a>为 Azure 文件配置目录级别或文件级别权限 
Azure 文件在目录级别和文件级别（包括根目录）强制实施标准 NTFS 文件权限。 仅支持通过 SMB 配置目录级别或文件级权限。 从 VM 装载目标文件共享，并使用 Windows [icacls](https://docs.microsoft.com/windows-server/administration/windows-commands/icacls) 或 [Set-ACL](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/get-acl) 命令配置权限。 

> [!NOTE]
> 预览版不支持通过 Windows 文件资源管理器配置 NTFS 权限。

### <a name="use-the-storage-account-key-for-superuser-permissions"></a>使用存储帐户密钥获取超级用户权限 
拥有存储帐户密钥的用户可使用超级用户权限访问 Azure 文件。 超级用户权限超越使用 RBAC 在共享级别配置并由 Azure AD 强制实施的所有访问控制限制。 装载 Azure 文件共享需要超级用户权限。 

> [!IMPORTANT]
> 作为安全最佳做法的一部分，请避免共享存储帐户密钥，并尽可能利用 Azure AD 权限。

### <a name="preserve-directory-and-file-acls-for-data-import-to-azure-file-shares"></a>保留目录和文件 ACL，以便将数据导入 Azure 文件共享
将数据复制到 Azure 文件共享时，通过 SMB 进行 Azure AD 身份验证支持保留目录或文件 ACL。 在预览版中，可将目录或文件上的 ACL 复制到 Azure 文件。 例如，可使用带 `/copy:s` 标志的 [robocopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) 将数据和 ACL 复制到 Azure 文件共享。

## <a name="pricing"></a>定价
对存储帐户启用通过 SMB 进行 Azure AD 身份验证无需额外的服务费。 有关定价的详细信息，请参阅 [Azure 文件定价](https://azure.microsoft.com/pricing/details/storage/files/)和 [Azure AD 域服务定价](https://azure.microsoft.com/pricing/details/active-directory-ds/)页面。

## <a name="next-steps"></a>后续步骤
有关 Azure 文件和通过 SMB 进行 Azure AD 身份验证的详细信息，请参阅以下资源：

- [Azure 文件简介](storage-files-introduction.md)
- [启用通过 SMB 为 Azure 文件进行 Azure Active Directory 身份验证（预览）](storage-files-active-directory-enable.md)
- [常见问题](storage-files-faq.md)
