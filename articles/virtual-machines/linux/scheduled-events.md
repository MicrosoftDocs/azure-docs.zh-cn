---
title: Azure 中 Linux VM 的计划事件 | Microsoft Docs
description: 通过为 Linux 虚拟机使用 Azure 元数据服务来计划事件。
services: virtual-machines-windows, virtual-machines-linux, cloud-services
documentationcenter: ''
author: ericrad
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2018
ms.author: ericrad
ms.openlocfilehash: 0831f08eaa3e8e6f6a0d3f68bc50cd927167b7ba
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65507921"
---
# <a name="azure-metadata-service-scheduled-events-for-linux-vms"></a>Azure 元数据服务：适用于 Linux VM 的计划事件

计划事件是一个 Azure 元数据服务，可提供应用程序时间用于准备虚拟机 (VM) 维护。 它提供有关即将发生的维护事件的信息（例如重新启动），以便应用程序可以为其进行准备并限制中断。 它可用于 Windows 和 Linux 上的所有 Azure 虚拟机类型（包括 PaaS 和 IaaS）。 

有关 Windows 上计划事件的信息，请参阅 [Windows VM 的计划事件](../windows/scheduled-events.md)。

> [!Note] 
> 计划事件在所有 Azure 区域中正式发布。 有关最新版本信息，请参阅[版本和区域可用性](#version-and-region-availability)。

## <a name="why-use-scheduled-events"></a>为何使用计划事件？

许多应用程序都可以受益于时间来准备 VM 维护。 时间可用于执行特定于应用程序的任务，这些任务可提升可用性、可靠性和可维护性，包括： 

- 检查点和还原。
- 连接清空。
- 主要副本故障转移。
- 从负载均衡器池删除。
- 事件日志记录。
- 正常关闭。

使用计划事件，应用程序可以发现维护的发生，并触发任务以限制其影响。  

预定事件提供以下用例中的事件：

- [平台启动维护](https://docs.microsoft.com/azure/virtual-machines/linux/maintenance-and-updates)（例如，VM 重启、 实时迁移或内存保留更新的主机）
- 降级的硬件
- 用户启动的维护（例如，用户重启或重新部署 VM）
- [低优先级 VM 逐出](https://azure.microsoft.com/blog/low-priority-scale-sets)中缩放设置

## <a name="the-basics"></a>基础知识  

  元数据服务使用可从 VM 内访问的 REST 终结点公开有关正在运行的虚拟机的信息。 该信息通过不可路由的 IP 提供，因此不会在 VM 外部公开。

### <a name="scope"></a>范围
计划的事件传送到：

- 独立虚拟机。
- 云服务中的所有 VM。
- 可用性集中的所有 VM。
- 规模集放置组中的所有 VM。 

因此，应查看事件中的 `Resources` 字段，确定哪些 VM 会受到影响。

### <a name="endpoint-discovery"></a>终结点发现
对于启用了 VNET 的 VM，元数据服务可通过不可路由的静态 IP (`169.254.169.254`) 使用。 最新版本的计划事件的完整终结点是： 

 > `http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01`

如果不是在虚拟网络中创建 VM（云服务和经典 VM 的默认情况），则需使用额外的逻辑以发现要使用的 IP 地址。 请参阅此示例，了解如何[发现主机终结点](https://github.com/azure-samples/virtual-machines-python-scheduled-events-discover-endpoint-for-non-vnet-vm)。

### <a name="version-and-region-availability"></a>版本和区域可用性
计划事件服务受版本控制。 版本是必需的，当前版本为 `2017-11-01`。

| Version | 发布类型 | 区域 | 发行说明 | 
| - | - | - | - | 
| 2017-11-01 | 正式版 | 全部 | <li> 添加了的对低优先级 VM 逐出 EventType 抢占<br> | 
| 2017-08-01 | 正式版 | 全部 | <li> 已从 IaaS VM 的资源名称中删除前置下划线<br><li>针对所有请求强制执行元数据标头要求 | 
| 2017-03-01 | Preview | 全部 | <li>初始版本


> [!NOTE] 
> 支持的计划事件的早期预览版发布 {最新} 为 api-version。 此格式不再受支持，并且会在未来被弃用。

### <a name="enabling-and-disabling-scheduled-events"></a>启用和禁用计划事件
首次为事件发出请求时，为服务启用了计划事件。 首次调用时应该会延迟响应最多两分钟。

如果 24 小时未发出请求，将为服务禁用计划事件。

### <a name="user-initiated-maintenance"></a>用户启动的维护
用户通过 Azure 门户、API、CLI 或 PowerShell 启动的 VM 维护会生成计划事件。 然后便可以在应用程序中测试维护准备逻辑，并可以针对用户启动的维护让应用程序做准备。

如果重新启动 VM，会计划一个 `Reboot` 类型的事件。 如果重新部署 VM，会计划一个 `Redeploy` 类型的事件。

## <a name="use-the-api"></a>使用 API

### <a name="headers"></a>标头
查询元数据服务时，必须提供标头 `Metadata:true`，以确保不会意外将请求重定向。 `Metadata:true` 标头对于所有预定事件请求是必需的。 不在请求中包含标头会导致元数据服务发出的“错误的请求”响应。

### <a name="query-for-events"></a>查询事件
可通过进行以下调用查询计划事件：

#### <a name="bash"></a>Bash
```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2017-08-01
```

响应包含计划事件的数组。 空数组表示目前没有计划事件。
在有计划事件的情况下，响应包含事件数组。 
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Reboot" | "Redeploy" | "Freeze" | "Preempt",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled" | "Started",
            "NotBefore": {timeInUTC},              
        }
    ]
}
```

### <a name="event-properties"></a>事件属性
|属性  |  描述 |
| - | - |
| EventId | 此事件的全局唯一标识符。 <br><br> 示例： <br><ul><li>602d9444-d2cd-49c7-8624-8643e7171297  |
| EventType | 此事件造成的影响。 <br><br> 值: <br><ul><li> `Freeze`：虚拟机将计划暂停几秒。 CPU 和网络连接可能会挂起，但不是会对内存或打开的文件没有影响。<li>`Reboot`：计划重启虚拟机（非永久性内存丢失）。 <li>`Redeploy`：计划将虚拟机移到另一节点（临时磁盘将丢失）。 <li>`Preempt`：正在删除低优先级虚拟机 （临时磁盘将丢失）。|
| ResourceType | 此事件影响的资源的类型。 <br><br> 值: <ul><li>`VirtualMachine`|
| 资源| 此事件影响的资源的列表。 此列表确保包含来自最多一个[更新域](manage-availability.md)的计算机，但可能不包含 UD 中的所有计算机。 <br><br> 示例： <br><ul><li> ["FrontEnd_IN_0", "BackEnd_IN_0"] |
| EventStatus | 此事件的状态。 <br><br> 值: <ul><li>`Scheduled`：此事件计划在 `NotBefore` 属性指定的时间之后启动。<li>`Started`：此事件已启动。</ul> 从不提供 `Completed` 或类似状态。 事件完成后不再返回事件。
| NotBefore| 一个时间，此事件可在该时间之后启动。 <br><br> 示例： <br><ul><li> 2016 年 9 月 19 日星期一 18:29:47 GMT  |

### <a name="event-scheduling"></a>事件计划
将根据事件类型为每个事件计划将来的最小量时间。 此时间将反映在事件的 `NotBefore` 属性中。 

|事件类型  | 最小值通知 |
| - | - |
| 冻结| 15 分钟 |
| 重新启动 | 15 分钟 |
| 重新部署 | 10 分钟 |
| 抢占 | 30 秒 |

### <a name="start-an-event"></a>启动事件 

了解即将发生的事件并完成正常关闭的逻辑后，可通过使用 `EventId` 对元数据服务进行 `POST` 调用，批准未完成事件。 此调用向 Azure 指示可缩短最短通知时间（如有可能）。 

下面是 `POST` 请求正文中所需的 JSON 示例。 请求应包含 `StartRequests` 列表。 每个 `StartRequest` 包含想要加速的事件的 `EventId`：
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

#### <a name="bash-sample"></a>Bash 示例
```
curl -H Metadata:true -X POST -d '{"StartRequests": [{"EventId": "f020ba2e-3bc0-4c40-a10b-86575a9eabd5"}]}' http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01
```

> [!NOTE] 
> 确认事件后，即可允许事件针对事件中所有的 `Resources` 继续进行，而不仅仅是确认该事件的 VM。 因此，可选择一个领导来协调确认，这可能与 `Resources` 字段中的第一个计算机一样简单。

## <a name="python-sample"></a>Python 示例 

以下示例将查询计划事件的元数据服务并审核每个未完成事件：

```python
#!/usr/bin/python

import json
import urllib2
import socket
import sys

metadata_url = "http://169.254.169.254/metadata/scheduledevents?api-version=2017-11-01"
headers = "{Metadata:true}"
this_host = socket.gethostname()

def get_scheduled_events():
   req = urllib2.Request(metadata_url)
   req.add_header('Metadata', 'true')
   resp = urllib2.urlopen(req)
   data = json.loads(resp.read())
   return data

def handle_scheduled_events(data):
    for evt in data['Events']:
        eventid = evt['EventId']
        status = evt['EventStatus']
        resources = evt['Resources']
        eventtype = evt['EventType']
        resourcetype = evt['ResourceType']
        notbefore = evt['NotBefore'].replace(" ","_")
        if this_host in resources:
            print "+ Scheduled Event. This host " + this_host + " is scheduled for " + eventtype + " not before " + notbefore
            # Add logic for handling events here


def main():
   data = get_scheduled_events()
   handle_scheduled_events(data)

if __name__ == '__main__':
  main()
  sys.exit(0)
```

## <a name="next-steps"></a>后续步骤 
- 观看 [Azure Friday 上的计划事件](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance)以查看演示。 
- 在 [Azure 实例元数据计划事件 GitHub 存储库](https://github.com/Azure-Samples/virtual-machines-scheduled-events-discover-endpoint-for-non-vnet-vm)中查看计划事件代码示例。
- 详细了解[实例元数据服务](instance-metadata-service.md)中可用的 API。
- 了解 [Azure 中 Linux 虚拟机的计划内维护](planned-maintenance.md)。
