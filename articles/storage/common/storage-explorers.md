---
title: 适用于 Azure 存储的工具 | Microsoft Docs
description: 可用于查看 Azure 存储数据或与之交互的工具列表。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 09/06/2017
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: common
ms.openlocfilehash: d7debbc760e103046ce9bb1a8bdf25a954d9891c
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65138555"
---
# <a name="azure-storage-client-tools"></a>Azure 存储客户端工具
Azure 存储用户经常要使用 Azure 存储客户端工具来查看其数据或者与数据交互。 下表列出了可用于实现此目的的多种工具。 我们在每个能够枚举和/或访问数据抽象的块中打上“X”。 该表还指明了工具是否免费。 “试用”表示存在免费试用版，但完整版产品不是免费的。 “是/否”表示有一个免费版本，而提供的其他版本需要购买。

我们只提供了可用的 Azure 存储客户端工具的快照。 这些工具的功能可能会不断改进和扩展。 如果有任何更正或更新，请发表评论告知我们。 同样，如果认为还应在此处列出其他工具，也请发表评论告知我们 - 我们会很乐意添加它们。

**Microsoft Azure 存储客户端工具**

<table>
  <tr>
    <th rowspan="2">Azure 存储客户端工具</th>
    <th rowspan="2">块 blob</th>
    <th rowspan="2">页 blob</th>
    <th rowspan="2">追加 Blob</th>
    <th rowspan="2">表</th>
    <th rowspan="2">队列</th>
    <th rowspan="2">文件</th>
    <th rowspan="2">免费</th>
    <th colspan="4">平台</th>
  </tr>
  <tr>
    <td>Web</td>
    <td>Windows</td>
    <td>OSX</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td><a href="https://azure.microsoft.com/features/azure-portal/">Microsoft Azure 门户</a></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>Y</td>
    <td>X</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="https://storageexplorer.com/">Microsoft Azure 存储资源管理器</a></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>Y</td>
    <td></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
  </tr>
  <tr>
    <td><a href="https://www.visualstudio.com/features/azure-tools-vs.aspx">Microsoft Visual Studio 服务器资源管理器</a></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td>Y</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
</table>

**第三方 Azure 存储客户端工具**

我们尚未验证由以下第三方工具声称的功能或质量，并且其列出的内容并不暗示获得 Microsoft 的认可。

<table>
  <tr>
    <th rowspan="2">Azure 存储客户端工具</th>
    <th rowspan="2">块 blob</th>
    <th rowspan="2">页 blob</th>
    <th rowspan="2">追加 Blob</th>
    <th rowspan="2">表</th>
    <th rowspan="2">队列</th>
    <th rowspan="2">文件</th>
    <th rowspan="2">免费</th>
    <th colspan="4">平台</th>
  </tr>
  <tr>
    <td>Web</td>
    <td>Windows</td>
    <td>OSX</td>
    <td>Linux</td>
  </tr>
  <tr>
    <td><a href="https://www.cerebrata.com/products/azure-management-studio/introduction">Cerabrata：Azure Management Studio</a></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>试用</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="https://www.red-gate.com/products/azure-development/azure-explorer/index">Redgate：Azure 资源管理器</a></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td></td>
    <td></td>
    <td>Y</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="https://github.com/sebagomez/azurestorageexplorer">Azure Web 存储资源管理器</a></td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td>Y</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="https://www.cloudberrylab.com/explorer/microsoft-azure.aspx">CloudBerry Explorer</a></td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td></td>
    <td></td>
    <td>X</td>
    <td>Y/N</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="https://www.gapotchenko.com/cloudcombine">Cloud Combine</a></td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td>X</td>
    <td>X</td>
    <td></td>
    <td>试用</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="http://clumsyleaf.com">ClumsyLeaf：AzureXplorer、CloudXplorer、TableXplorer</a></td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>X</td>
    <td>Y</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td><a href="https://www.gladinet.com/Azure-Storage/index.htm">Gladinet Cloud</a></td>
    <td>X</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>试用</td>
    <td></td>
    <td>X</td>
    <td></td>
    <td></td>
  </tr>
</table>
