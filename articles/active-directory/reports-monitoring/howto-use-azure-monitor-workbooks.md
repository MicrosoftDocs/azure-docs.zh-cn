---
title: 使用 Azure Monitor 的工作簿的报表的 Azure Active Directory |Microsoft Docs
description: 了解如何使用 Azure Monitor 工作簿获取 Azure Active Directory 报告。
services: active-directory
author: MarkusVi
manager: daveba
ms.assetid: 4066725c-c430-42b8-a75b-fe2360699b82
ms.service: active-directory
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.openlocfilehash: 6ae14ec152975717af5d55780bcc39aa87c4b01a
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406595"
---
# <a name="how-to-use-azure-monitor-workbooks-for-azure-active-directory-reports"></a>如何使用 Azure Monitor 的工作簿的报表的 Azure Active Directory

想：

- 了解的影响你[条件性访问策略](../conditional-access/overview.md)上用户的登录体验？

- 对登录失败以获取更好的视图组织的登录运行状况并快速解决问题进行故障排除？

- 知道谁在使用旧的身份验证登录到你的环境？ (通过[阻止旧式身份验证](../conditional-access/block-legacy-authentication.md)，可以改进你的租户的保护。)

为了帮助你解决这些问题，Active Directory 提供用于监视的工作簿。 [Azure 监视器工作簿](https://docs.microsoft.com/azure/azure-monitor/app/usage-workbooks)将文本、 分析查询、 指标和参数组合到丰富的交互式报表。 

本文：

- 假定您熟悉如何[使用监视器工作簿创建交互式报表](https://docs.microsoft.com/azure/azure-monitor/app/usage-workbooks)。

- 介绍如何使用监视工作簿来了解条件性访问策略，来解决登录失败，并识别旧身份验证的影响。
 


## <a name="prerequisites"></a>必备组件

若要使用监视器工作簿，需要：

- 高级 （P1 或 P2） 许可证与一个 Active Directory 租户。 了解如何[获得高级许可证](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-get-started-premium)。

- 一个[Log Analytics 工作区](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)。

## <a name="workbook-access"></a>工作簿的访问 

若要访问工作簿：

1. 登录到 [Azure 门户](https://portal.azure.com)。

2. 在左侧的导航窗格中，选择**Azure Active Directory**。

3. 在中**监视**部分中，选择**Insights**。 

    ![选择 Insights](./media/howto-use-azure-monitor-workbooks/41.png)

4. 选择报表或模板，或在工具栏上选择**打开**。 

    ![选择打开](./media/howto-use-azure-monitor-workbooks/42.png)


## <a name="sign-in-analysis"></a>在登录分析

若要访问登录 analysis 工作簿中，在**使用情况**部分中，选择**登录名**。 

此工作簿显示了以下登录趋势：

- 所有登录

- Success

- 挂起的用户操作

- 失败

可以按以下类别来筛选每个趋势：

- 时间范围

- 应用程序

- 用户

![在登录分析](./media/howto-use-azure-monitor-workbooks/43.png)


每个趋势，按以下类别获取细分：

- Location

    ![按位置的登录名](./media/howto-use-azure-monitor-workbooks/45.png)

- 设备

    ![通过设备的登录名](./media/howto-use-azure-monitor-workbooks/46.png)


## <a name="sign-ins-using-legacy-authentication"></a>使用传统的身份验证登录名 


若要访问工作簿中的使用的登录名[旧式身份验证](../conditional-access/block-legacy-authentication.md)，在**使用情况**部分中，选择**登录名使用传统的身份验证**。 

此工作簿显示了以下登录趋势：

- 所有登录

- Success


可以按以下类别来筛选每个趋势：

- 时间范围

- 应用程序

- 用户

- 协议

![通过传统的身份验证的登录名](./media/howto-use-azure-monitor-workbooks/47.png)


对于每个趋势，您获取按应用程序和协议细分。

![按应用程序和协议 legacy 身份验证登录名](./media/howto-use-azure-monitor-workbooks/48.png)



## <a name="sign-ins-by-conditional-access"></a>通过条件性访问的登录名 


若要访问工作簿中的登录名由[条件性访问策略](../conditional-access/overview.md)，在**条件性访问**部分中，选择**通过条件性访问的登录名**。 

此工作簿显示了已禁用的登录名的趋势。可以按以下类别来筛选每个趋势：

- 时间范围

- 应用程序

- 用户

![登录名使用条件性访问](./media/howto-use-azure-monitor-workbooks/49.png)


为已禁用的登录名，可获取细分的条件访问状态。

![条件性访问状态](./media/howto-use-azure-monitor-workbooks/conditional-access-status.png)








## <a name="sign-ins-by-grant-controls"></a>通过登录授权控件

若要访问工作簿中的登录名由[授权控件](../conditional-access/controls.md)，在**条件性访问**部分中，选择**登录名授予控制通过**。 

此工作簿显示下列已禁用登录趋势：

- 要求 MFA
 
- 需要使用条款

- 需要隐私声明

- 其他


可以按以下类别来筛选每个趋势：

- 时间范围

- 应用程序

- 用户

![通过登录授权控件](./media/howto-use-azure-monitor-workbooks/50.png)


对于每个趋势，您获取按应用程序和协议细分。

![最新的登录名的细目](./media/howto-use-azure-monitor-workbooks/51.png)




## <a name="sign-ins-failure-analysis"></a>登录失败分析

使用**登录名失败分析**工作簿以解决以下错误：

- 登录
- 条件访问策略
- 旧身份验证 


若要访问的条件性访问的数据，登录名，在**进行故障排除**部分中，选择**登录名使用旧版身份验证**。 

此工作簿显示了以下登录趋势：

- 所有登录

- Success

- 挂起的操作

- 失败


可以按以下类别来筛选每个趋势：

- 时间范围

- 应用程序

- 用户

![故障排除的登录名](./media/howto-use-azure-monitor-workbooks/52.png)


为了帮助您解决登录，Azure Monitor 提供细分按以下类别：

- 最常见的错误

    ![最常见的错误的摘要](./media/howto-use-azure-monitor-workbooks/53.png)

- 等待用户操作的登录名

    ![登录名等待用户操作的摘要](./media/howto-use-azure-monitor-workbooks/54.png)






## <a name="next-steps"></a>后续步骤

[使用监视器工作簿创建交互式报表](https://docs.microsoft.com/azure/azure-monitor/app/usage-workbooks)。