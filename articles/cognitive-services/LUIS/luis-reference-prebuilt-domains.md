---
title: 预构建的域参考
titleSuffix: Azure
description: 预构建的域参考，这些参考是语言理解智能服务 (LUIS) 中意向和实体的预构建集合。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/07/2019
ms.author: diberry
ms.openlocfilehash: 3265477108b7e74d65050408add6c5d5c94b4852
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/07/2019
ms.locfileid: "65233894"
---
# <a name="prebuilt-domain-reference-for-your-luis-app"></a>LUIS 应用的预构建的域参考
此参考提供有关[预构建的域](luis-how-to-use-prebuilt-domains.md)的信息，这些参考是 LUIS 提供的意向和实体的预构建集合。

相比之下，[自定义域](luis-how-to-start-new-app.md)从无意向和模型开始。 可将任何预构建的域意向和实体添加到自定义模型中。

# <a name="supported-domains-across-cultures"></a>各个区域性受支持的域

唯一支持的区域性为英语。 

<!--

The table below summarizes the currently supported domains. Support for English is usually more complete than others.


| Entity Type       | EN-US      | ZH-CN   | DE    | FR     | ES    | IT      | PT-BR |  JP  |      KO |        NL |    TR |
|:-----------------:|:-------:|:-------:|:-----:|:------:|:-----:|:-------:| :-------:| :-------:| :-------:| :-------:|  :-------:| 
| Calendar    | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | -      | -    | -    | -     | -  |
| Communication   | ✓    | -       | ✓    | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| Email           | ✓    | ✓       | ✓   | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| HomeAutomation           | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| Notes      | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| Places    | ✓    | -       | ✓    | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| RestaurantReservation   | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| ToDo     | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | -  | -      | -    | -    | -     | -  |
| ToDo_IPA        | ✓    | ✓       | ✓    | ✓      | ✓     | ✓       | -  | -      | -    | -    | -     | -  |
| Utilities          | ✓    | ✓        | ✓    | ✓      | ✓     | ✓       | -  | -      | -    | -    | -     | -  |
| Weather        | ✓    | ✓        | ✓    | ✓      | ✓     | ✓       | -  | -      | -    | -    | -     | -  |
| Web    | ✓    | -        | ✓    | ✓      | ✓     | ✓       | -  | -      | -    | -    | -     | -  |
||||||||||||| 

-->

<br><br>

|实体类型|description|
|--|--|
|日历|有关个人会议和约会的日历是任何内容_不_公共事件 (如 world 杯计划、 西雅图的活动日历或泛型日历 (如哪些天目前是这样，什么才 fall 开始，何时劳动节)。|
|通信|请求进行调用，发送短信或即时消息，查找并添加联系人和各种其他通信与相关的请求 （通常传出）。 联系人姓名仅查询不属于域通信。|
|电子邮件|电子邮件是通信域的子域。 它主要包含发送和接收电子邮件消息的请求。|
|家庭自动化|HomeAutomation 域提供意图和实体与控制智能家庭设备相关。 它主要支持灯和空调的控制命令，但它具有其他 electric 设备一些泛化功能。|
|说明|请注意域提供意图和实体，可用于创建说明和写下为用户的项。|
|场所|位置包括企业、 机构、 餐馆、 公共场所和地址。 域支持位置，查找和询问有关如位置、 操作系统小时和距离的公共位置的信息。|
|餐位预订|餐馆预订域支持意向处理餐馆的保留项。|
|ToDo|ToDo 域提供的用户可以添加、 标记和删除他们的 todo 项的任务列表的类型。|
|公用事业|实用程序是在所有 LUIS 之间常规域预构建的模型，其中包含常见意向和区别方案中的语音样本。|
|天气|天气域侧重于检查天气条件和建议的位置和时间或时间检查由天气条件。|
|Web|Web 域提供意向和实体搜索网站。|
