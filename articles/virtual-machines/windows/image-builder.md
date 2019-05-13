---
title: 使用 Azure 映像生成器 （预览版） 创建 Windows VM
description: 使用 Azure 映像生成器创建 Windows VM。
author: cynthn
ms.author: cynthn
ms.date: 05/02/2019
ms.topic: article
ms.service: virtual-machines-windows
manager: jeconnoc
ms.openlocfilehash: 01109aa83c12bda9b1d21ec25784d663f8abf700
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159715"
---
# <a name="preview-create-a-windows-vm-with-azure-image-builder"></a>预览版：使用 Azure 映像生成器创建 Windows VM

本文是向您显示如何创建使用 Azure VM 映像生成器的自定义的 Windows 映像。 这篇文章中的示例使用三个不同[定制员](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#properties-customize)的自定义映像：
- PowerShell (ScriptUri)-下载并运行[PowerShell 脚本](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/testPsScript.ps1)。
- Windows 重新启动-重新启动 VM。
- PowerShell （内联）-运行特定命令。 在此示例中，它创建一个目录上 VM 使用`mkdir c:\\buildActions`。
- 文件-从 GitHub 到 VM 上复制文件。 此示例将复制[index.md](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/exampleArtifacts/buildArtifacts/index.html)到`c:\buildArtifacts\index.html`在 VM 上。

我们将使用示例.json 模板要配置的映像。 下面是我们正在使用的.json 文件： [helloImageTemplateWin.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Windows_Managed_Image/helloImageTemplateWin.json)。 


> [!IMPORTANT]
> Azure 映像生成器目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


## <a name="register-the-features"></a>注册功能

若要在预览期间使用 Azure 映像生成器，您需要注册新的功能。

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

检查功能注册状态。

```azurecli-interactive
az feature show --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview | grep state
```

检查您的注册。

```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState

az provider show -n Microsoft.Storage | grep registrationState
```

如果他们未说已注册，运行以下命令：

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages

az provider register -n Microsoft.Storage
```

## <a name="create-a-resource-group"></a>创建资源组

我们将使用一些部分信息重复，因此我们将创建一些变量来存储该信息。

```azurecli-interactive
# Resource group name - we are using myImageBuilderRG in this example
imageResourceGroup=myWinImgBuilderRG
# Region location 
location=WestUS2
# Name for the image 
imageName=myWinBuilderImage
# Run output name
runOutputName=aibWindows
# name of the image to be created
imageName=aibWinImage
```

为订阅 id。 创建一个变量 可以获取此使用`az account show | grep id`。

```azurecli-interactive
subscriptionID=<Your subscription ID>
```

创建资源组。

```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```

授予该资源组中创建资源的映像生成器权限。 `--assignee`值是图像生成器服务的应用程序注册 ID。 

```azurecli-interactive
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```


## <a name="download-the-json-example"></a>下载.json 示例

下载示例.json 文件，并将其配置与你创建的变量。

```azurecli-interactive
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Windows_Managed_Image/helloImageTemplateWin.json -o helloImageTemplateWin.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateWin.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" helloImageTemplateWin.json
sed -i -e "s/<region>/$location/g" helloImageTemplateWin.json
sed -i -e "s/<imageName>/$imageName/g" helloImageTemplateWin.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateWin.json

```

## <a name="create-the-image"></a>创建映像

映像的配置提交到的 VM 映像生成器服务

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @helloImageTemplateWin.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateWin01
```

开始创建映像。

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateWin01 \
     --action Run 
```

等待生成完成。 这可能需要大约 15 分钟。

## <a name="create-the-vm"></a>创建 VM

创建使用你生成的映像的 VM。 替换*<password>* 替换为你自己密码`aibuser`在 VM 上。

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgWinVm00 \
  --admin-username aibuser \
  --admin-password <password> \
  --image $imageName \
  --location $location
```

## <a name="verify-the-customization"></a>验证自定义项

创建远程桌面连接到 VM 使用的用户名和密码创建 VM 时设置。 在 VM 中，打开 cmd 提示符并键入：

```console
dir c:\
```

应会看到这些映像自定义过程中创建的两个目录：
- buildActions
- buildArtifacts

## <a name="clean-up"></a>清理

完成后，删除的资源。

```azurecli-interactive
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateWin01
az group delete -n $imageResourceGroup
```

## <a name="next-steps"></a>后续步骤

若要了解有关本文中使用的.json 文件的组件的详细信息，请参阅[映像生成器模板引用](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

