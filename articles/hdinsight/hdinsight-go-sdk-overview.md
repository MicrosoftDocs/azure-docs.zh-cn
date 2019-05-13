---
title: Azure HDInsight SDK for Go
description: Azure HDInsight sdk for Go 的引用
author: tylerfox
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/8/2019
ms.author: tyfox
ms.custom: seodec18
ms.openlocfilehash: 113948d77d87a34822f81f020b03f6628b9c5e84
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466224"
---
# <a name="hdinsight-sdk-for-go-preview"></a>HDInsight SDK for Go （预览版）

## <a name="overview"></a>概述
HDInsight SDK for Go 提供类和函数，可用于管理 HDInsight 群集。 该 SDK 包含用于创建、删除、更新、列出、调整大小、执行脚本操作，以及监视、获取 HDInsight 群集属性等操作。

> [!NOTE]  
>还可以[从此处获得](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2018-06-01-preview/hdinsight)适用于此 SDK 的GoDoc 参考资料。

## <a name="prerequisites"></a>必备组件

* 一个 Azure 帐户。 如果没有帐户，可[获取一个免费试用帐户](https://azure.microsoft.com/free/)。
* [Go](https://golang.org/dl/)。

## <a name="sdk-installation"></a>SDK 安装

从 GOPATH 位置中，运行 `go get github.com/Azure/azure-sdk-for-go/tree/master/services/preview/hdinsight/mgmt/2018-06-01-preview/hdinsight`

## <a name="authentication"></a>Authentication

首先需要使用 Azure 订阅对该 SDK 进行身份验证。  请遵循以下示例创建服务主体，然后使用该服务主体进行身份验证。 完成此操作后，将会获得 `ClustersClient` 的实例，其中包含可用于执行管理操作的许多函数（以下部分将概述这些函数）。

> [!NOTE]  
> 除了以下示例中所示的方法以外，还有其他一些身份验证方法可能更符合你的需要。 此处概述了所有函数：[Azure SDK for Go 中的身份验证函数](https://docs.microsoft.com/go/azure/azure-sdk-go-authorization)

### <a name="authentication-example-using-a-service-principal"></a>使用服务主体的身份验证示例

首先登录到 [Azure Cloud Shell](https://shell.azure.com/bash)。 验证当前使用的是要在其中创建服务主体的订阅。 

```azurecli-interactive
az account show
```

订阅信息将显示为 JSON。

```json
{
  "environmentName": "AzureCloud",
  "id": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "isDefault": true,
  "name": "XXXXXXX",
  "state": "Enabled",
  "tenantId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "user": {
    "cloudShellID": true,
    "name": "XXX@XXX.XXX",
    "type": "user"
  }
}
```

如果尚未登录到正确的订阅，请运行以下命令选择正确的订阅： 
```azurecli-interactive
az account set -s <name or ID of subscription>
```

> [!IMPORTANT]  
> 如果你尚未注册 HDInsight 资源提供程序的另一个函数 (如通过创建一个通过 Azure 门户的 HDInsight 群集)，您需要执行此操作后之前您可以进行身份验证。 可以在 [Azure Cloud Shell](https://shell.azure.com/bash) 中运行以下命令来完成此操作：
>```azurecli-interactive
>az provider register --namespace Microsoft.HDInsight
>```

接下来，选择服务主体的名称，然后使用以下命令创建服务主体：

```azurecli-interactive
az ad sp create-for-rbac --name <Service Principal Name> --sdk-auth
```

服务主体信息将以 JSON 格式显示。

```json
{
  "clientId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "clientSecret": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "subscriptionId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "tenantId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```
复制以下代码片段，并在 `TENANT_ID`、`CLIENT_ID`、`CLIENT_SECRET` 和 `SUBSCRIPTION_ID` 中填写运行创建服务主体的命令后返回的 JSON 中的字符串。

```golang
package main

import (
    "context"
    "github.com/Azure/go-autorest/autorest/azure/auth"
    hdi "github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2018-06-01-preview/hdinsight"
    "github.com/Azure/go-autorest/autorest/to"
)

func main() {
    var err error

    // Tenant ID for your Azure Subscription
    var TENANT_ID = ""
    // Your Service Principal App Client ID
    var CLIENT_ID = ""
    // Your Service Principal Client Secret
    var CLIENT_SECRET = ""
    // Azure Subscription ID
    var SUBSCRIPTION_ID = ""

    var credentials = auth.NewClientCredentialsConfig(CLIENT_ID, CLIENT_SECRET, TENANT_ID)
    var client = hdi.NewClustersClient(SUBSCRIPTION_ID)

    client.Authorizer, err = credentials.Authorizer()
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
```

## <a name="cluster-management"></a>群集管理

> [!NOTE]  
> 本部分假设你已完成身份验证，已构造 `ClusterClient` 实例并已将其存储在名为 `client` 的变量中。 在前面的“身份验证”部分中可以找到有关身份验证和获取 `ClusterClient` 的说明。

### <a name="create-a-cluster"></a>创建群集

可以通过调用 `client.Create()` 来创建新群集。 

#### <a name="example"></a>示例

本示例演示如何创建包含 2 个头节点和 1 个工作节点的 [Apache Spark](https://spark.apache.org/) 群集。

> [!NOTE]  
> 首先需要创建一个资源组和存储帐户，下面将予以介绍。 如果已创建资源组和存储帐户，则可以跳过这些步骤。

##### <a name="creating-a-resource-group"></a>创建资源组

可以在 [Azure Cloud Shell](https://shell.azure.com/bash) 中运行以下命令来创建资源组
```azurecli-interactive
az group create -l <Region Name (i.e. eastus)> --n <Resource Group Name>
```
##### <a name="creating-a-storage-account"></a>创建存储帐户

可以在 [Azure Cloud Shell](https://shell.azure.com/bash) 中运行以下命令来创建存储帐户
```azurecli-interactive
az storage account create -n <Storage Account Name> -g <Existing Resource Group Name> -l <Region Name (i.e. eastus)> --sku <SKU i.e. Standard_LRS>
```
现在，运行以下命令获取存储帐户的密钥（创建群集时需要用到）：
```azurecli-interactive
az storage account keys list -n <Storage Account Name>
```
---
以下 Go 代码片段创建包含 2 个头节点和 1 个工作节点的 Spark 群集。 按照注释中所述填写空白变量，并根据具体的需要任意更改其他参数。

```golang
// The name for the cluster you are creating
var clusterName = "";
// The name of your existing Resource Group
var resourceGroupName = "";
// Choose a username
var username = "";
// Choose a password
var password = "";
// Replace <> with the name of your storage account
var storageAccount = "<>.blob.core.windows.net";
// Storage account key you obtained above
var storageAccountKey = "";
// Choose a region
var location = "";
var container = "default";

var parameters = hdi.ClusterCreateParametersExtended {
    Location: to.StringPtr(location),
    Tags: make(map[string]*string),
    Properties: &hdi.ClusterCreateProperties {
        ClusterVersion: to.StringPtr("3.6"),
        OsType: hdi.Linux,
        ClusterDefinition: &hdi.ClusterDefinition {
            Kind: to.StringPtr("spark"),
            Configurations: map[string]map[string]interface{}{
                "gateway": {
                    "restAuthCredential.isEnabled": "True",
                    "restAuthCredential.username":  username,
                    "restAuthCredential.password":  password,
                },
            },
        },
        Tier: hdi.Standard,
        ComputeProfile: &hdi.ComputeProfile {
            Roles: &[]hdi.Role {
                hdi.Role {
                    Name: to.StringPtr("headnode"),
                    TargetInstanceCount: to.Int32Ptr(2),
                    HardwareProfile: &hdi.HardwareProfile {
                        VMSize: to.StringPtr("Large"),
                    },
                    OsProfile: &hdi.OsProfile {
                        LinuxOperatingSystemProfile: &hdi.LinuxOperatingSystemProfile {
                            Username: to.StringPtr(username),
                            Password: to.StringPtr(password),
                        },
                    },
                },
                hdi.Role {
                    Name: to.StringPtr("workernode"),
                    TargetInstanceCount: to.Int32Ptr(1),
                    HardwareProfile: &hdi.HardwareProfile {
                        VMSize: to.StringPtr("Large"),
                    },
                    OsProfile: &hdi.OsProfile {
                        LinuxOperatingSystemProfile: &hdi.LinuxOperatingSystemProfile {
                            Username: to.StringPtr(username),
                            Password: to.StringPtr(password),
                        },
                    },
                },
            },
        },
        StorageProfile: &hdi.StorageProfile {
            Storageaccounts: &[]hdi.StorageAccount {
                hdi.StorageAccount {
                    Name: to.StringPtr(storageAccount),
                    Key: to.StringPtr(storageAccountKey),
                    Container: to.StringPtr(container),
                    IsDefault: to.BoolPtr(true),
                },
            },
        },
    },
}
client.Create(context.Background(), resourceGroupName, clusterName, parameters)
```

### <a name="get-cluster-details"></a>获取群集详细信息

获取给定群集的属性：

```golang
client.Get(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

#### <a name="example"></a>示例

可以使用 `get` 来确认已成功创建群集。

```golang
cluster, err := client.Get(context.Background(), resourceGroupName, clusterName)
if (err != nil) {
    fmt.Println("Error: ", err)
}
fmt.Println(*cluster.Name)
fmt.Println(*cluster.ID
```

输出应如下所示：

```
<Cluster Name>
/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<Resource Group Name>/providers/Microsoft.HDInsight/clusters/<Cluster Name>
```

### <a name="list-clusters"></a>列出群集

#### <a name="list-clusters-under-the-subscription"></a>列出订阅下的群集
```golang
client.List()
```
#### <a name="list-clusters-by-resource-group"></a>按资源组列出群集
```golang
client.ListByResourceGroup("<Resource Group Name>")
```

> [!NOTE]  
> `List()` 和 `ListByResourceGroup()` 都返回 `ClusterListResultPage` 结构。 若要获取下一个页面，可以调用 `Next()`。 可以反复执行此调用，直到 `ClusterListResultPage.NotDone()` 返回 `false`，如以下示例中所示。

#### <a name="example"></a>示例
以下示例列显当前订阅的所有群集的属性：

```golang
page, err := client.List(context.Background())
if (err != nil) {
    fmt.Println("Error: ", err)
}
for (page.NotDone()) {
    for _, cluster := range page.Values() {
        fmt.Println(*cluster.Name)
    }
    err = page.Next();
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
}
```

### <a name="delete-a-cluster"></a>删除群集

删除群集：

```golang
client.Delete(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

### <a name="update-cluster-tags"></a>更新群集标记

可按如下所示更新给定群集的标记：

```golang
client.Update(context.Background(), "<Resource Group Name>", "<Cluster Name>", hdi.ClusterPatchParameters{<map[string]*string} of Tags>)
```
#### <a name="example"></a>示例

```golang
client.Update(context.Background(), "SDKTestRG", "SDKTest", hdi.ClusterPatchParameters{map[string]*string{"tag1Name" : to.StringPtr("tag1Value"), "tag2Name" : to.StringPtr("tag2Value")}})
```

### <a name="resize-cluster"></a>调整群集大小

可以通过指定新大小来调整给定群集的工作节点数，如下所示：

```golang
client.Resize(context.Background(), "<Resource Group Name>", "<Cluster Name>", hdi.ClusterResizeParameters{<Num of Worker Nodes (int)>})
```

## <a name="cluster-monitoring"></a>群集监视

使用 HDInsight 管理 SDK 还可以通过 Operations Management Suite (OMS) 来管理群集的监视。

与创建 `ClusterClient` 以用于管理操作类似，你需要创建一个 `ExtensionClient` 来用于监视操作。 完成上面的“身份验证”部分后，可以创建一个 `ExtensionClient`，如下所示：

```golang
extClient := hdi.NewExtensionsClient(SUBSCRIPTION_ID)
extClient.Authorizer, _ = credentials.Authorizer()
```

> [!NOTE]  
> 下面的监视示例假定你已如上所示初始化了一个名为 `extClient` 的 `ExtensionClient` 并设置了其 `Authorizer`。

### <a name="enable-oms-monitoring"></a>启用 OMS 监视

> [!NOTE]  
> 若要启用 OMS 监视，必须已有一个 Log Analytics 工作区。 如果尚未创建工作区，可在此了解创建方法：[在 Azure 门户中创建 Log Analytics 工作区](https://docs.microsoft.com/azure/log-analytics/log-analytics-quick-create-workspace)。

在群集上启用 OMS 监视：

```golang
extClient.EnableMonitoring(context.Background(), "<Resource Group Name", "Cluster Name", hdi.ClusterMonitoringRequest {WorkspaceID: to.StringPtr("<Workspace Id>")})
```

### <a name="view-status-of-oms-monitoring"></a>查看 OMS 监视状态

获取群集上的 OMS 状态：

```golang
extClient.GetMonitoringStatus(context.Background(), "<Resource Group Name", "Cluster Name")
```

### <a name="disable-oms-monitoring"></a>禁用 OMS 监视

在群集上禁用 OMS：

```golang
extClient.DisableMonitoring(context.Background(), "<Resource Group Name", "Cluster Name")
```

## <a name="script-actions"></a>脚本操作

HDInsight 提供了一个称为“脚本操作”的配置函数，用以调用自定义脚本来自定义群集。

> [!NOTE]  
> 有关如何使用脚本操作的详细信息见此处：[使用脚本操作自定义基于 Linux 的 HDInsight 群集](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux)

### <a name="execute-script-actions"></a>执行脚本操作

可按如下所示在给定的群集上执行脚本操作：

```golang
var scriptAction1 = hdi.RuntimeScriptAction{Name: to.StringPtr("<Script Name>"), URI: to.StringPtr("<URL To Script>"), Roles: <&[]string of roles>} //valid roles are "headnode", "workernode", "zookeepernode", and "edgenode"
client.ExecuteScriptActions(context.Background(), "<Resource Group Name>", "<Cluster Name>", hdi.ExecuteScriptActionParameters{PersistOnSuccess: to.BoolPtr(true), ScriptActions: &[]hdi.RuntimeScriptAction{scriptAction1}}) //add more RuntimeScriptActions to the list to execute multiple scripts
```

对于“删除脚本操作”和“列出持久化脚本操作”操作，你需要创建一个 `ScriptActionsClient`，类似于创建 `ClusterClient` 以用于管理操作。 完成上面的“身份验证”部分后，可以创建一个 `ScriptActionsClient`，如下所示：

```golang
scriptActionsClient := hdi.NewScriptActionsClient(SUBSCRIPTION_ID)
scriptActionsClient.Authorizer, _ = credentials.Authorizer()
```

> [!NOTE]  
> 下面的脚本操作示例假定你已如上所示初始化了一个名为 `scriptActionsClient` 的 `ScriptActionsClient` 并设置了其 `Authorizer`。

### <a name="delete-script-action"></a>删除脚本操作

删除给定群集上指定的持久化脚本操作：

```golang
scriptActionsClient.Delete(context.Background(), "<Resource Group Name>", "<Cluster Name>", "<Script Name>")
```

### <a name="list-persisted-script-actions"></a>列出持久化脚本操作

> [!NOTE]  
> `ListByCluster()` 返回一个 `ScriptActionsListPage` 结构。 若要获取下一个页面，可以调用 `Next()`。 可以反复执行此调用，直到 `ClusterListResultPage.NotDone()` 返回 `false`，如以下示例中所示。

列出指定群集的所有持久化脚本操作：
```golang
scriptActionsClient.ListByCluster(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

#### <a name="example"></a>示例

```golang
page, err := scriptActionsClient.ListByCluster(context.Background(), resourceGroupName, clusterName)
if (err != nil) {
    fmt.Println("Error: ", err)
}
for (page.NotDone()) {
    for _, script := range page.Values() {
        fmt.Println(*script.Name) //There are functions to get other properties of RuntimeScriptActionDetail besides Name, such as Status, Operation, StartTime, EndTime, etc. See reference documentation.
    }
    err = page.Next();
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
}
```

### <a name="list-all-scripts-execution-history"></a>列出所有脚本的执行历史记录

对于此操作，与创建 `ScriptExecutionHistoryClient` 以用于管理操作的方式类似，你需要创建一个 `ClusterClient`。 完成上面的“身份验证”部分后，可以创建一个 `ScriptActionsClient`，如下所示：

```golang
scriptExecutionHistoryClient := hdi.NewScriptExecutionHistoryClient(SUBSCRIPTION_ID)
scriptExecutionHistoryClient.Authorizer, _ = credentials.Authorizer()
```

> [!NOTE]  
> 以下示例假定你已如上所示初始化了一个名为 `scriptExecutionHistoryClient` 的 `ScriptExecutionHistoryClient` 并设置了其 `Authorizer`。

列出指定群集的所有脚本的执行历史记录：

```golang
scriptExecutionHistoryClient.ListByCluster(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

#### <a name="example"></a>示例

此示例列显以往所有脚本执行活动的所有详细信息。

```golang
page, err := scriptExecutionHistoryClient.ListByCluster(context.Background(), resourceGroupName, clusterName)
if (err != nil) {
    fmt.Println("Error: ", err)
}
for (page.NotDone()) {
    for _, script := range page.Values() {
        fmt.Println(*script.Name) //There are functions to get other properties of RuntimeScriptActionDetail besides Name, such as Status, Operation, StartTime, EndTime, etc. See reference documentation.
    }
    err = page.Next();
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
}
```

## <a name="next-steps"></a>后续步骤

* 探究 [GoDoc 参考资料](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2018-06-01-preview/hdinsight)。 GoDoc 提供了 SDK 中的所有函数的参考文档。
