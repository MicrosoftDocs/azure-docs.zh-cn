---
title: Azure Data Box 的故障排除使用 REST 接口 |Microsoft Docs
description: 介绍如何排查当数据复制时通过 REST 接口，在 Azure Data Box 中出现的问题。
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: article
ms.date: 04/19/2019
ms.author: alkohli
ms.openlocfilehash: c5ceeb2e6419cab7945454087edd4c821db28343
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65204221"
---
# <a name="troubleshoot-issues-related-to-azure-data-box-blob-storage"></a>对与 Azure 数据框 Blob 存储相关的问题进行故障排除

有关如何对问题进行故障排除此项目的详细信息信息可能会看到使用通过你的数据框上的 REST 接口的数据框 Blob 存储复制数据时。 Python 中都使用与其他应用程序或客户端库，如 Azure 存储资源管理器、 AzCopy 或 Azure 存储库数据框 Blob 存储时这些问题面。

## <a name="errors-seen-in-azure-storage-explorer"></a>在 Azure 存储资源管理器中出现的错误

本部分详细介绍一些与数据框 Blob 存储使用 Azure 存储资源管理器时所面临的问题。

|错误消息  |建议的操作 |
|---------|---------|
|无法检索子资源。 其中一个 HTTP 标头的值不是正确的格式。|从**编辑**菜单中，选择**目标 Azure Stack Api**。 <br>重新启动 Azure 存储资源管理器。|
|`getaddrinfo ENOTFOUND <accountname>.blob.<serialnumber>.microsoftdatabox.com` |检查终结点名称`<accountname>.blob.<serialnumber>.microsoftdatabox.com`添加到此路径的主机文件： <li>`C:\Windows\System32\drivers\etc\hosts` 在 Windows，或 </li><li> `/etc/hosts` 在 Linux。</li>|
|无法检索子资源。 <br>详细信息： 自签名的证书 |你的设备的 SSL 证书导入 Azure 存储资源管理器： <li>从 Azure 门户下载的证书。 有关详细信息，请转到[下载的证书](data-box-deploy-copy-data-via-rest.md#download-certificate)。</li><li>从**编辑**菜单中，选择**SSL 证书**，然后选择**导入证书**。</li>|

## <a name="errors-seen-in-azcopy-for-windows"></a>在 AzCopy 的 Windows 中出现的错误

本部分详细介绍一些与数据框 Blob 存储使用 AzCopy 的 Windows 时所面临的问题。

|错误消息  |建议的操作 |
|---------|---------|
|AzCopy 命令显示为挂起显示此错误前一分钟： <br>无法枚举目录 https://... 无法解析远程名称 `<accountname>.blob.<serialnumber>.microsoftdatabox.com`|检查终结点名称`<accountname>.blob.<serialnumber>.microsoftdatabox.com`添加到主机文件： `C:\Windows\System32\drivers\etc\hosts`。|
|AzCopy 命令显示为挂起显示此错误前一分钟： <br>错误分析源位置。 基础连接已关闭：无法建立 SSL/TLS 安全通道的信任关系。|你的设备的 SSL 证书导入系统的证书存储中。 有关详细信息，请转到[下载的证书](data-box-deploy-copy-data-via-rest.md#download-certificate)。|


## <a name="errors-seen-in-azcopy-for-linux"></a>适用于 Linux 的 AzCopy 中出现的错误

本部分详细介绍一些适用于 Linux 的数据框 Blob 存储使用 AzCopy 时所面临的问题。

|错误消息  |建议的操作 |
|---------|---------|
|AzCopy 命令显示为挂起的 20 分钟后再显示此错误： <br>分析源位置时出错`https://<accountname>.blob.<serialnumber>.microsoftdatabox.com/<cntnr>`。 没有此设备或地址|检查终结点名称`<accountname>.blob.<serialnumber>.microsoftdatabox.com`添加到主机文件： `/etc/hosts`。|
|AzCopy 命令显示为挂起的 20 分钟后再显示此错误： <br>错误分析源位置... 无法建立 SSL 连接。|你的设备的 SSL 证书导入系统的证书存储中。 有关详细信息，请转到[下载的证书](data-box-deploy-copy-data-via-rest.md#download-certificate)。|

## <a name="errors-seen-in-azure-storage-library-for-python"></a>Python 的 Azure 存储库中出现的错误

本部分详细介绍了在使用 Linux 客户端进行数据复制时部署 Data Box Disk 所遇到的一些常见问题。

|错误消息  |建议的操作 |
|---------|---------|
|其中一个 HTTP 标头的值不是正确的格式。 |安装用于 Python 的 Microsoft Azure 存储库的版本不支持的数据框。 请参阅受支持版本的 Azure 数据框 Blob 存储要求。|
|… [SSL:CERTIFICATE_VERIFY_FAILED] …|运行 Python 之前, 将 REQUESTS_CA_BUNDLE 环境变量设置为 Base64 编码的 SSL 证书文件的路径 (请参阅如何[下载的证书](data-box-deploy-copy-data-via-rest.md#download-certificate))。 <br>例如：<br>`export REQUESTS_CA_BUNDLE=/tmp/mycert.cer` <br>`python` <br>或者，将证书添加到系统的证书存储区，然后将此环境变量设置为该存储区的路径。 <br> 例如，在 Ubuntu 上： <br>`export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt` <br>`python`|


## <a name="common-errors"></a>常见错误

这些错误并不特定于任何应用程序。

|错误消息  |建议的操作 |
|---------|---------|
|连接超时。 |登录到 Data Box 设备并检查它未锁定。 任何时候，只要设备重启，它保持锁定状态，直到某人登录。|

## <a name="next-steps"></a>后续步骤

- 了解如何[数据框 Blob 存储的系统要求](data-box-system-requirements-rest.md)。
