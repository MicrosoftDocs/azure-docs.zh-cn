---
title: 最佳实践，以使用 Azure Key Vault 的 Key Vault-|Microsoft Docs
description: 本文档介绍了某些使用密钥保管库的最佳做法
services: key-vault
author: msmbaldwin
manager: barbkess
tags: azure-key-vault
ms.service: key-vault
ms.topic: conceptual
ms.date: 03/07/2019
ms.author: mbaldwin
ms.openlocfilehash: eb7150d0b1c3a4a312b0c05ba7612960aaf640f6
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65227931"
---
# <a name="best-practices-to-use-key-vault"></a>使用密钥保管库的最佳做法

## <a name="control-access-to-your-vault"></a>控制对你的保管库的访问

Azure 密钥保管库是一种云服务，用于保护加密密钥和机密（例如证书、连接字符串和密码）。 因为此数据是敏感数据和业务关键数据，所以需要保护对密钥保管库的访问，只允许得到授权的应用程序和用户进行访问。 这[一文](key-vault-secure-your-key-vault.md)提供密钥保管库访问模型的概述。 其中介绍了身份验证和授权，以及如何保护对密钥保管库的访问。

控制对你的保管库的访问时的建议如下所示：
1. 锁定对你的订阅、 资源组和密钥保管库 (RBAC) 访问
2. 创建每个保管库访问的策略
3. 使用最低特权访问主体授予访问权限
4. 打开防火墙和[VNET 服务终结点](key-vault-overview-vnet-service-endpoints.md)

## <a name="use-separate-key-vault"></a>使用单独的密钥保管库

我们的建议是使用每个应用程序每个环境 （开发、 预生产和生产） 的保管库。 这可帮助你跨环境共享机密，并还可以减少发生违规时的威胁。

## <a name="backup"></a>备份

请确保采取常规回 ups 的你[保管库](https://blogs.technet.microsoft.com/kv/2018/07/20/announcing-backup-and-restore-of-keys-secrets-and-certificates/)上更新/删除/创建的保管库中的对象。

## <a name="turn-on-logging"></a>启用日志记录

[启用日志记录](key-vault-logging.md)保管库。 此外设置警报。

## <a name="turn-on-recovery-options"></a>打开恢复选项

1. 开启[软删除](key-vault-ovw-soft-delete.md)。
2. 如果你想要防止强制删除的机密 / 保管库启用软删除后，即使，启用清除保护。
