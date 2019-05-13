---
title: 连接到 Azure Analysis Services 所需的客户端库 | Microsoft Docs
description: 介绍了客户端应用程序和工具连接 Azure Analysis Services 时所需的客户端库
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 88ded03f636155e4b1a738f388e696a09736cd3b
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470771"
---
# <a name="client-libraries-for-connecting-to-azure-analysis-services"></a>用于连接到 Azure Analysis Services 的客户端库

客户端应用程序和工具连接到 Analysis Services 服务器时需要使用客户端库。 Microsoft 客户端应用程序（例如 Power BI Desktop、Excel、SQL Server Management Studio (SSMS) 和 SQL Server Data Tools (SSDT)）会安装全部三个客户端库并通过定期的应用程序更新来更新它们。 在某些情况下，可能需要安装较新版本的客户端库。 自定义客户端应用程序还需要安装客户端库。

## <a name="download-the-latest-client-libraries-windows-installer"></a>下载最新客户端库 (Windows Installer)  

|下载  |产品版本  | 
|---------|---------|
|[MSOLAP (amd64)](https://go.microsoft.com/fwlink/?linkid=829576)    |    15.0.19.24    |
|[MSOLAP (x86)](https://go.microsoft.com/fwlink/?linkid=829575)     |    15.0.19.24      |
|[AMO](https://go.microsoft.com/fwlink/?linkid=829578)     |   16.1.0.0    |
|[ADOMD](https://go.microsoft.com/fwlink/?linkid=829577)     |    16.1.0.0     |

## <a name="amo-and-adomd-nuget-packages"></a>AMO 和 ADOMD（NuGet 包）

Analysis Services Management Objects (AMO) 和 ADOMD 客户端库在 [NuGet.org](https://www.nuget.org/) 上作为可安装的程序包提供。建议你迁移到 NuGet 引用而非使用 Windows Installer。 

|包  | 产品版本  | 
|---------|---------|
|[AMO](https://www.nuget.org/packages/Microsoft.AnalysisServices.retail.amd64/)    |    15.17.1     |
|[ADOMD](https://www.nuget.org/packages/Microsoft.AnalysisServices.AdomdClient.retail.amd64/)     |   15.17.1      |

NuGet 包程序集 AssemblyVersion 遵循语义版本控制：MAJOR.MINOR.PATCH。 NuGet 引用加载预期的版本，即使 GAC 中存在不同的版本（由 MSI 安装导致的）。 PATCH 将随每次发布递增。 AMO 和 ADOMD 版本保持同步。

## <a name="understanding-client-libraries"></a>了解客户端库

Analysis Services 利用三个客户端库（也称为数据提供程序）。 ADOMD.NET 和 Analysis Services 管理对象 (AMO) 都是托管型客户端库。 Analysis Services OLE DB 提供程序 (MSOLAP DLL) 是本机客户端库。 通常，所有三个库会同时安装。 **Azure Analysis Services 要求安装所有三个库的最新版本**。 

Microsoft 客户端应用程序（例如 Power BI Desktop 和 Excel）会安装所有这三个客户端库，有新版本可用时，会更新这些库。 某些客户端库可能不是 Azure Analysis Services 所需要的最新版本，具体取决于更新的版本或频率。 这同样适用于自定义应用程序或其他界面，例如 AsCmd、TOM、ADOMD.NET。 这些应用程序需要手动或以编程方式安装库。 用于手动安装的客户端库作为可分发程序包包含在 SQL Server 功能包中。 但是，这些客户端库与 SQL Server 版本关联，可能不是最新的。  

用于客户端连接的客户端库不同于从 Azure Analysis Services 服务器连接到数据源时所需的数据提供程序。 若要详细了解数据源连接，请参阅[数据源连接](analysis-services-datasource.md)。

## <a name="client-library-types"></a>客户端库类型

### <a name="analysis-services-ole-db-provider-msolap"></a>Analysis Services OLE DB 提供程序 (MSOLAP) 

 Analysis Services OLE DB 提供程序 (MSOLAP) 是用于建立 Analysis Services 数据库连接的本机客户端库。 ADOMD.NET 和 AMO 间接使用它向数据提供程序委托连接请求。 也可以直接从应用程序代码调用 OLE DB 提供程序。  
  
 用于访问 Analysis Services 数据库的大多数工具和客户端应用程序可自动安装 Analysis Services OLE DB 提供程序。 必须将它安装在用于访问 Analysis Services 数据的计算机上。  
  
 OLE DB 提供程序通常在连接字符串中指定。 Analysis Services 连接字符串使用不同的命名法来引用 OLE DB 提供程序：MSOLAP.\<版本>.dll。

### <a name="amo"></a>AMO  

 AMO 是用于服务器管理和数据定义的托管客户端库。 它由工具和客户端应用程序安装和使用。 例如，SQL Server Management Studio (SSMS) 使用 AMO 连接到 Analysis Services。 使用 AMO 的连接通常非常精简，由 `"data source=\<servername>"` 组成。 建立连接后，可以使用 API 来处理数据库集合和主要对象。 SSDT 和 SSMS 都使用 AMO 连接到 Analysis Services 实例。  

  
### <a name="adomd"></a>ADOMD

 ADOMD.NET 是用于查询 Analysis Services 数据的托管数据客户端库。 它由工具和客户端应用程序安装和使用。 
  
 连接到数据库时，所有三个库的连接字符串属性相似。 使用 [Microsoft.AnalysisServices.AdomdClient.AdomdConnection.ConnectionString](/dotnet/api/microsoft.analysisservices.adomdclient.adomdconnection.connectionstring#Microsoft_AnalysisServices_AdomdClient_AdomdConnection_ConnectionString) 为 ADOMD.NET 定义的几乎任何连接字符串同样适用于 AMO 和 Analysis Services OLE DB 提供程序 (MSOLAP)。 若要了解详细信息，请参阅[连接字符串属性 &#40;Analysis Services&#41;](https://docs.microsoft.com/sql/analysis-services/instances/connection-string-properties-analysis-services)。  

  
##  <a name="bkmk_LibUpdate"></a>如何确定客户端库版本   
  
### <a name="oleddb-msolap"></a>OLEDDB (MSOLAP)  
  
1.  转到  `C:\Program Files\Microsoft Analysis Services\AS OLEDB\` 。 如果有多个文件夹，请选择较大的数字。
  
2.  右键单击“msolap.dll” > “属性” > “详细信息”。 如果文件名为 msolap140.dll，则它早于最新版本并且应当升级。
    
    ![客户端库详细信息](media/analysis-services-data-providers/aas-msolap-details.png)
    
  
### <a name="amo"></a>AMO

1. 转到  `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.AnalysisServices\` 。 如果有多个文件夹，请选择较大的数字。
2. 右键单击“Microsoft.AnalysisServices” > ，选择“属性” > “详细信息”。  

### <a name="adomd"></a>ADOMD

1. 转到  `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.AnalysisServices.AdomdClient\` 。 如果有多个文件夹，请选择较大的数字。
2. 右键单击“Microsoft.AnalysisServices.AdomdClient” > ，选择“属性” > “详细信息”。  


## <a name="next-steps"></a>后续步骤
[使用 Excel 进行连接](analysis-services-connect-excel.md)    
[使用 Power BI 进行连接](analysis-services-connect-pbi.md)
