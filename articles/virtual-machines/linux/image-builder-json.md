---
title: 创建 Azure 映像生成器模板 （预览版）
description: 了解如何创建使用 Azure 映像生成器使用的模板。
author: cynthn
ms.author: cynthn
ms.date: 05/02/2019
ms.topic: article
ms.service: virtual-machines-linux
manager: jeconnoc
ms.openlocfilehash: b4646879eb7eeecf41852baab7ab64e4053b05e1
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159595"
---
# <a name="preview-create-an-azure-image-builder-template"></a>预览版：创建 Azure 映像生成器模板 

Azure 映像生成器使用的.json 文件以将信息传递到映像生成器服务。 在本文中我们将经过的部分的 json 文件，以便可以开始构建您自己。 若要查看完整的.json 文件的示例，请参阅[Azure 映像生成器 GitHub](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts)。

这是基本模板格式：

```json
 { 
    "type": "Microsoft.VirtualMachineImages/imageTemplates", 
    "apiVersion": "2019-05-01-preview", 
    "location": "<region>", 
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
             }
    "identity":{},           
    "dependsOn": [], 
    "properties": { 
        "<build timeout in minutes>": {}, 
        "build": {}, 
        "customize": {}, 
        "distribute": {} 
      } 
 } 
```



## <a name="type-and-api-version"></a>类型和 API 版本

`type`是资源类型，它必须是`"Microsoft.VirtualMachineImages/imageTemplates"`。 `apiVersion`会随时间随 API 的更改，但应为`"2019-05-01-preview"`预览版。

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2019-05-01-preview",
```

## <a name="location"></a>Location

位置是在其中创建自定义映像的区域。 映像生成器预览版支持以下区域：

- 美国东部
- 美国东部 2
- 美国中西部
- 美国西部
- 美国西部 2


```json
    "location": "<region>",
```
    
## <a name="depends-on-optional"></a>取决于 （可选）

此可选部分可以用于确保依赖项在继续之前完成。 

```json
    "dependsOn": [],
```

有关详细信息，请参阅[定义的资源依赖项](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-define-dependencies#dependson)。

## <a name="identity"></a>标识
默认情况下，映像生成器支持使用脚本，或从多个位置，例如 GitHub 和 Azure 存储复制文件。 若要使用这些模板，它们必须是可公开访问。

Azure User-Assigned 托管标识，由你定义还可用于允许图像生成器访问 Azure 存储中，只要标识已获得 Azure 存储帐户上的存储 Blob 数据读取器最少。 这意味着不需要从外部访问的存储 blob 或安装程序的 SAS 令牌。


```json
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<imgBuilderId>": {}
        }
        },
```

有关完整示例，请参阅[访问 Azure 存储中的文件使用 Azure User-Assigned 托管标识](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage)。

图像生成器支持为用户分配标识: • • 不支持自定义域名只支持单个标识

若要了解详细信息，请参阅[什么是 Azure 资源的管理的标识？](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)。
有关部署此功能的详细信息，请参阅[配置管理使用 Azure CLI 在 Azure VM 上的 Azure 资源的标识](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm#user-assigned-managed-identity)。

## <a name="properties-source"></a>属性： 源

`source`部分包含有关图像生成器将使用的源映像的信息。

此 API 要求定义图像生成的源 SourceType，目前有三种类型：
- ISO-使用此源，则 RHEL ISO。
- PlatformImage-指示源映像是一个 Marketplace 映像。
- ManagedImage-从常规的托管映像启动时使用此。
- SharedImageVersion-这使用作为源共享映像库中使用的映像版本时。

### <a name="iso-source"></a>ISO 源

Azure 映像生成器仅支持使用已发布的 Red Hat Enterprise Linux 7.x 二进制 DVD Iso 和预览版。 映像生成器支持：
- RHEL 7.3 
- RHEL 7.4 
- RHEL 7.5 
 
```json
"source": {
       "type": "ISO",
       "sourceURI": "<sourceURI from the download center>",
       "sha256Checksum": "<checksum associated with ISO>"
}
```

若要获取`sourceURI`并`sha256Checksum`值，请转到`https://access.redhat.com/downloads`然后选择的产品**Red Hat Enterprise Linux**，和受支持的版本。 

在列表中**安装程序和 Red Hat Enterprise Linux Server 映像**，您需要将复制为 Red Hat Enterprise Linux 7.x 二进制 DVD 和校验和的链接。

> [!NOTE]
> 频繁刷新链接的访问令牌，因此每次你想要提交模板，必须检查如果 RH 链接地址已更改。
 
### <a name="platformimage-source"></a>PlatformImage 源 
Azure 映像生成器支持以下 Azure Marketplace 映像：
* Ubuntu 18.04
* Ubuntu 16.04
* RHEL 7.6
* CentOS 7.6
* Windows 2016
* Windows 2019

```json
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "UbuntuServer",
                "sku": "18.04-LTS",
                "version": "18.04.201903060"
        },
```


此处的属性是一样的用于创建 VM，使用 AZ CLI，运行下面获取的属性： 
 
```azurecli-interactive
az vm image list -l westus -f UbuntuServer -p Canonical --output table –-all 
```

> [!NOTE]
> 版本不能为 latest，必须使用上面的命令来获取版本号。 

### <a name="managedimage-source"></a>ManagedImage 源

将源映像设置为通用化的 VHD 或 VM 的现有托管映像。 源托管的映像必须是支持的操作系统，并在 Azure 映像生成器模板所在的同一区域中。 

```json
        "source": { 
            "type": "ManagedImage", 
                "imageId": "/subscriptions/<subscriptionId>/resourceGroups/{destinationResourceGroupName}/providers/Microsoft.Compute/images/<imageName>"
        }
```

`imageId`应为的托管映像资源 Id。 使用`az image list`若要列出可用的映像。


### <a name="sharedimageversion-source"></a>SharedImageVersion 源
设置共享映像库中的源映像的现有的映像版本。 映像版本必须是受支持的 OS 和该映像必须复制到 Azure 映像生成器模板所在的同一区域。 

```json
        "source": { 
            "type": "SharedImageVersion", 
            "imageVersionID": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/p  roviders/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageDefinitionName/versions/<imageVersion>" 
   } 
```

`imageVersionId`应为映像版本的资源 Id。 使用[az sig 映像版本列表](/cli/azure/sig/image-version#az-sig-image-version-list)到列出映像版本。

## <a name="properties-customize"></a>属性： 自定义


映像生成器支持多个定制员。 自定义程序是用于自定义映像，例如，运行脚本，或重新启动服务器函数。 

当使用`customize`: 
- 您可以使用多个定制员，但它们必须具有一个唯一`name`。
- 在模板中指定的顺序执行定制员。
- 如果一个自定义失败，则整个自定义组件将失败并报告错误。
- 请考虑图像生成将要求，并调整 buildTimeoutInMinutes 属性，以允许足够的时间来完成的映像生成器多少时间。
- 强烈建议在模板中使用前进行全面测试脚本。 在自己的 VM 上的脚本调试将更加简单。
- 不要将敏感数据放在脚本中。 
- 脚本位置需要可公开访问，除非你使用[MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage)。

```json
        "customize": [
            {
                "type": "Shell",
                "name": "<name>",
                "scriptUri": "<path to script>"
            },
            {
                "type": "Shell",
                "name": "<name>",
                "inline": [
                    "<command to run inline>",
                ]
            }

        ],
```     

 
自定义部分是一个数组。 通过按先后顺序定制员将运行 azure 映像生成器。 任何自定义中的出现任何失败将会生成过程失败。 
 
 
### <a name="shell-customizer"></a>Shell 定制员

命令行程序自定义支持运行 shell 脚本，这些必须可公开访问的 IB 若要对其进行访问。

```json
    "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "scriptUri": "<link to script>"        
        }, 
    ], 
        "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "inline": "<commands to run>"
        }, 
    ], 
```

OS 支持：Linux 
 
自定义属性：

- **类型**– Shell 
- **名称**-用于跟踪自定义项名称 
- **scriptUri** -文件的位置的 URI 
- **内联**-shell 命令，用逗号分隔的数组。
 
> [!NOTE]
> 运行 shell 定制员和 RHEL ISO 源，您需要确保第一个自定义 shell 句柄注册到 Red Hat 授权服务器，进行任何自定义之前。 自定义完成后，该脚本应取消注册与授权服务器。

### <a name="windows-restart-customizer"></a>Windows 重新启动自定义 
重新启动自定义，可重启 Windows VM，并等待它重新恢复联机状态，这使您可以安装需要重启的软件。  

```json 
     "customize": [ 
            "type{ ": "WindowsRestart", 
            "restartCommand": "shutdown /r /f /t 0 /c", 
            "restartCheckCommand": "echo Azure-Image-Builder-Restarted-the-VM  > buildArtifacts/azureImageBuilderRestart.txt",
            "restartTimeout": "5m"
         }],
```

OS 支持：Windows
 
自定义属性：
- **类型**：WindowsRestart
- **restartCommand** -（可选） 重新启动时执行命令。 默认为 `'shutdown /r /f /t 0 /c \"packer restart\"'`。
- **restartCheckCommand** – 命令来检查重新启动成功 （可选）。 
- **restartTimeout** -重新启动超时指定为字符串的大小和单位。 例如， `5m` （5 分钟） 或`2h`（2 小时）。 默认值为：5 分钟


### <a name="powershell-customizer"></a>PowerShell 自定义 
运行 PowerShell 脚本和内联命令 shell 自定义支持，脚本必须是可公开访问的 IB 若要对其进行访问。

```json 
     "customize": [
        { 
             "type": "PowerShell",
             "name":   "<name>",  
             "scriptUri": "<path to script>" 
        },  
        { 
             "type": "PowerShell", 
             "name": "<name>", 
             "inline": "<PowerShell syntax to run>", 
             "valid_exit_codes": "<exit code>" 
         } 
    ], 
```

OS 支持：Windows 和 Linux

自定义属性：

- **类型**– PowerShell。
- **scriptUri** -PowerShell 脚本文件的位置的 URI。 
- **内联**– 内联命令来运行，并以逗号隔开。
- **valid_exit_codes** – 可选的可以从脚本/内联返回的有效代码命令时，这会避免报告的失败的内联脚本/命令。

### <a name="file-customizer"></a>文件自定义

文件自定义允许从 GitHub 或 Azure 存储下载文件的映像生成器。 如果您有一个映像生成管道，它依赖于生成项目，然后可以设置文件定制员可以从生成共享中，下载并将项目移到映像。  

```json
     "customize": [ 
         { 
            "type": "File", 
             "name": "<name>", 
             "sourceUri": "<source location>",
             "destination": "<destination>" 
         }
     ]
```

OS 支持：Linux 和 Windows 

文件自定义属性：

- **sourceUri**的可访问的存储终结点，这可以是 GitHub 或 Azure 存储。 仅可以下载一个文件，不是整个目录。 如果你需要下载一个目录，使用压缩的文件，然后使用 Shell 或 PowerShell 定制员将其解压缩。 
- **目标**– 这是完整的目标路径和文件名称。 任何引用的路径和子目录必须存在，请使用命令行程序或 PowerShell 定制员事先为这些设置。 脚本定制人员可用于创建路径。 

这支持的 Windows 目录和 Linux 路径，但有一些差异： 
- Linux OS 的 – 映像生成器可以写入的唯一路径是 /tmp。
- Windows-没有路径限制，但路径必须存在。
 
 
如果没有错误尝试下载文件，或将其放在指定目录中，自定义步骤将失败，并且这将在 customization.log。

可以从 Azure 存储下载文件自定义中的文件使用[MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage)。

### <a name="generalize"></a>通用化 
默认情况下，Azure 映像生成器还会取消预配代码末尾的每个映像自定义阶段，来通用化该映像。 通用化是其中的映像设置，以便可以重用它来创建多个 Vm 的过程。 对于 Windows Vm，Azure 映像生成器使用 Sysprep。 对于 Linux，Azure 映像生成器运行 waagent-deprovision。 

映像生成器用户的命令进行通用化处理可能不适合每种情况下，因此 Azure 映像生成器将可用于自定义此命令中，如果需要。 

如果要迁移现有自定义，并且使用不同的 Sysprep/waagent 命令，可以使用映像生成器泛型命令，并创建 VM 的操作失败，如果使用自己的 Sysprep 或 waagent 命令。

如果 Azure 映像生成器创建 Windows 自定义映像已成功，并从它，然后找到的 VM 创建失败或未成功完成创建 VM，需要查看 Windows Server Sysprep 文档，或使用的支持请求Windows Server Sysprep 客户服务支持团队，他们就可以进行故障排除并向正确的 Sysprep 使用建议。


#### <a name="default-sysprep-command"></a>默认 Sysprep 命令
```powershell
echo '>>> Waiting for GA to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
while ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running') { Start-Sleep -s 5 }
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
echo '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\\windows\\system32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\windows\\system32\\Sysprep\\unattend.xml -Force} & $Env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit
while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 5  } else { break } }
```
#### <a name="default-linux-deprovision-command"></a>默认 Linux 取消设置命令

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

#### <a name="overriding-the-commands"></a>重写命令
若要重写命令，请使用 PowerShell 或 Shell 脚本配置程序创建命令文件的确切的文件名，并将其放在正确的目录中：

* Windows: c:\DeprovisioningScript.ps1
* Linux: /tmp/DeprovisioningScript.sh

映像生成器将读取这些命令，这些被写出到 AIB 日志中，customization.log。 请参阅[故障排除](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#collecting-and-reviewing-aib-logs)如何收集日志。
 
## <a name="properties-distribute"></a>属性： 分发

Azure 映像生成器支持三个分发目标： 

- **managedImage** -托管映像。
- **sharedImage** -共享映像库。
- **VHD** -存储帐户中的 VHD。

可以将映像分发到这两个相同的配置中的目标类型，请参阅[示例](https://github.com/danielsollondon/azvmimagebuilder/blob/7f3d8c01eb3bf960d8b6df20ecd5c244988d13b6/armTemplates/azplatform_image_deploy_sigmdi.json#L80)。

由于你可以有多个目标分发给，图像生成器将保持的状态可以通过查询访问每个分发目标`runOutputName`。  `runOutputName`是您可以查询的对象发布有关该分布的信息的分发。 例如，您可以查询的 VHD 或其中的映像版本已复制到的区域的位置。 这是每个分发目标的属性。 `runOutputName`必须是唯一的分发的每个目标。
 
### <a name="distribute-managedimage"></a>分发： managedImage

图像输出将为托管的映像资源。

```json
"distribute": [
        {
"type":"managedImage",
       "imageId": "<resource ID>",
       "location": "<region>",
       "runOutputName": "<name>",
       "artifactTags": {
            "<name": "<value>",
             "<name>": "<value>"
               }
         }]
```
 
分发包属性：
- **类型**– managedImage 
- **imageId** – 目标图像的资源 ID 格式应： /subscriptions/<subscriptionId>/resourceGroups/<destinationResourceGroupName>/providers/Microsoft.Compute/images/<imageName>
- **位置**的托管映像的位置。  
- **runOutputName** – 唯一名称标识分发。  
- **artifactTags** -可选的用户指定的键值对标记。
 
 
> [!NOTE]
> 目标资源组必须存在。
> 如果要将映像分发到不同的区域，它将会增加部署时间。 

### <a name="distribute-sharedimage"></a>分发： sharedImage 
Azure 共享映像库是一种新的映像管理服务，允许的图像区域复制、 管理版本控制和共享自定义映像。 Azure 映像生成器支持分发与此服务，因此您可以将映像分发到支持共享图像库的区域。 
 
共享映像库是组成： 
 
- 库的多个共享的映像的容器。 库被部署在一个区域中。
- 图像定义-映像的概念分组。 
- 映像版本-这是用于部署 VM 或规模集的图像类型。 映像版本可以复制到 Vm 需要部署其他区域。
 
你可以分发到映像库之前，必须创建一个库和图像定义，请参阅[共享的映像](shared-images.md)。 

```json
{
     "type": "sharedImage",
     "galleryImageId": “<resource ID>”,
     "runOutputName": "<name>",
     "artifactTags": {
          "<name": "<value>",
           "<name>": "<value>"
             }
     "replicationRegions": [
        "<region where the gallery is deployed>",
        "<region>"
    ]}
``` 

分发共享的图像库的属性：

- **type** - sharedImage  
- **galleryImageId** – 共享的映像库的 ID。 格式是： /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageGalleryName>。
- **runOutputName** – 唯一名称标识分发。  
- **artifactTags** -可选的用户指定的键值对标记。
- **replicationRegions** -的区域复制的数组。 一个区域必须是部署库所在的区域。
 
> [!NOTE]
> 可以在不同的区域到库中，使用 Azure 映像生成器，但 Azure 映像生成器服务将需要在数据中心之间传输图像和需要较长时间。 映像生成器将自动版本映像，根据单调的整数，则目前无法指定它。 

### <a name="distribute-vhd"></a>分发包：VHD  
您可以输出到 VHD。 然后可以将 VHD 复制，并使用它来发布到 Azure MarketPlace 或与 Azure Stack 配合使用。  

```json
 { 
     "type": "VHD",
     "runOutputName": "<VHD name>",
     "tags": {
          "<name": "<value>",
           "<name>": "<value>"
             }
 }
```
 
OS 支持：Windows 和 Linux

分发 VHD 参数：

- **类型**-VHD。
- **runOutputName** – 唯一名称标识分发。  
- **标记**-可选的用户指定的键值对标记。
 
Azure 映像生成器不允许用户指定的存储帐户位置，但您可以查询的状态`runOutputs`获取的位置。  

```azurecli-interactive
az resource show \
   --ids "/subscriptions/$subscriptionId/resourcegroups/<imageResourceGroup>/providers/Microsoft.VirtualMachineImages/imageTemplates/<imageTemplateName>/runOutputs/<runOutputName>"  | grep artifactUri 
```

> [!NOTE]
> 一旦创建 VHD 后，将其复制到另一个位置，越早越好。 VHD 存储在映像模板提交到 Azure 映像生成器服务时创建的临时资源组中的存储帐户。 如果删除映像模板时，您将丢失 VHD。 
 
## <a name="next-steps"></a>后续步骤

有示例.json 文件中的不同情形[Azure 映像生成器 GitHub](https://github.com/danielsollondon/azvmimagebuilder)。
 
 
 
 
 
 
 
 
 
 
