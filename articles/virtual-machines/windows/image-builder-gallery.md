---
title: Azure 映像生成器使用映像库的 Windows 虚拟机 （预览版）
description: 使用 Azure 映像生成器和共享映像库创建 Windows 映像。
author: cynthn
ms.author: cynthn
ms.date: 05/02/2019
ms.topic: article
ms.service: virtual-machines-widows
manager: jeconnoc
ms.openlocfilehash: 2453d37720bcf48b95b428cf78c6186de40b31aa
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65160105"
---
# <a name="preview-create-a-windows-image-and-distribute-it-to-a-shared-image-gallery"></a>预览版：创建 Windows 映像，并将其分发给共享映像库 

本文旨在向您展示如何使用 Azure 映像生成器创建的映像版本中[共享映像库](shared-image-galleries.md)，然后将映像分发全局范围内。

我们将使用.json 模板要配置的映像。 下面是我们正在使用的.json 文件： [helloImageTemplateforWinSIG.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Win_Shared_Image_Gallery_Image/helloImageTemplateforWinSIG.json)。 

若要将映像分发到共享映像库，该模板，请使用[sharedImage](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#distribute-sharedimage)的值作为`distribute`模板部分。

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
az provider show -n Microsoft.Compute | grep registrationState
```

如果他们未说已注册，运行以下命令：

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages
az provider register -n Microsoft.Storage
az provider register -n Microsoft.Compute
```

## <a name="set-variables-and-permissions"></a>设置变量和权限 

我们将使用一些部分信息重复，因此我们将创建一些变量来存储该信息。 值替换为变量，如`username`和`vmpassword`，使用你自己的信息。

```azurecli-interactive
# Resource group name - we are using ibsigRG in this example
sigResourceGroup=myIBWinRG
# Datacenter location - we are using West US 2 in this example
location=westus
# Additional region to replicate the image to - we are using East US in this example
additionalregion=eastus
# name of the shared image gallery - in this example we are using myGallery
sigName=my22stSIG
# name of the image definition to be created - in this example we are using myImageDef
imageDefName=winSvrimages
# image distribution metadata reference name
runOutputName=w2019SigRo
# User name and password for the VM
username="azureuser"
vmpassword="passwordfortheVM"
```

为订阅 id。 创建一个变量 可以获取此使用`az account show | grep id`。

```azurecli-interactive
subscriptionID="Subscription ID"
```

创建资源组。

```azurecli-interactive
az group create -n $sigResourceGroup -l $location
```


授予该资源组中创建资源的 Azure 映像生成器权限。 `--assignee`值是图像生成器服务的应用程序注册 ID。 

```azurecli-interactive
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup
```





## <a name="create-an-image-definition-and-gallery"></a>创建图像定义和库

创建一个映像库。 

```azurecli-interactive
az sig create \
    -g $sigResourceGroup \
    --gallery-name $sigName
```

创建图像定义。

```azurecli-interactive
az sig image-definition create \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --publisher corpIT \
   --offer myOffer \
   --sku 2019 \
   --os-type Windows
```


## <a name="download-and-configure-the-json"></a>下载并配置.json

下载.json 模板并将其配置与你的变量。

```azurecli-interactive
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Win_Shared_Image_Gallery_Image/helloImageTemplateforWinSIG.json -o helloImageTemplateforWinSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforWinSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforWinSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforWinSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforWinSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforWinSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforWinSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforWinSIG.json
```

## <a name="create-the-image-version"></a>创建的映像版本

此下一步部分将在库中创建的映像版本。 

映像的配置提交到 Azure 映像生成器服务。

```azurecli-interactive
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforWinSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforWinSIG01
```

开始创建映像。

```azurecli-interactive
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateforWinSIG01 \
     --action Run 
```

创建映像并将其复制到这两个区域可能需要一段时间。 等待，直到创建的 VM 继续之前完成此部分。


## <a name="create-the-vm"></a>创建 VM

从 Azure 映像生成器创建的映像版本创建的 VM。

```azurecli-interactive
az vm create \
  --resource-group $sigResourceGroup \
  --name aibImgWinVm001 \
  --admin-username $username \
  --admin-password $vmpassword \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --location $location
```


## <a name="verify-the-customization"></a>验证自定义项
创建远程桌面连接到 VM 使用的用户名和密码创建 VM 时设置。 在 VM 中，打开 cmd 提示符并键入：

```console
dir c:\
```

你应看到一个名为目录`buildActions`在映像自定义过程中创建。


## <a name="clean-up-resources"></a>清理资源
如果你想要现在，请尝试重新自定义映像版本来创建相同的映像的新版本**跳过此步骤**并转到[使用 Azure 映像生成器来创建另一个映像版本](image-builder-gallery-update-image-version.md)。


这将删除已创建，以及所有其他资源文件的映像。 请确保您已完成此部署，然后删除资源。

时删除图像库资源，你需要删除所有映像版本，然后才能删除映像定义用来创建它们。 若要删除库，首先需要删除了所有库中的图像定义。

删除映像生成器模板。

```azurecli-interactive
az resource delete \
    --resource-group $sigResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforWinSIG01
```

获取由映像生成器创建的映像版本，这始终始于`0.`，然后删除映像版本

```azurecli-interactive
sigDefImgVersion=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'name' -o json | grep 0. | tr -d '"')
az sig image-version delete \
   -g $sigResourceGroup \
   --gallery-image-version $sigDefImgVersion \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID
```   


删除映像定义。

```azurecli-interactive
az sig image-definition delete \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID
```

删除库。

```azurecli-interactive
az sig delete -r $sigName -g $sigResourceGroup
```

删除该资源组。

```azurecli-interactive
az group delete -n $sigResourceGroup -y
```

## <a name="next-steps"></a>后续步骤

若要了解如何更新您创建的映像版本，请参阅[使用 Azure 映像生成器来创建另一个映像版本](image-builder-gallery-update-image-version.md)。