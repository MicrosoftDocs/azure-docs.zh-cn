---
title: 在 Azure Stack Edge Pro GPU 设备上启用边缘容器注册表
description: 描述如何在 Azure Stack Edge Pro GPU 设备上启用本地边缘容器注册表。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/11/2020
ms.author: alkohli
ms.openlocfilehash: bccb6fa33007082737997c7282fb286c38e3bbd7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/01/2020
ms.locfileid: "96466052"
---
# <a name="enable-edge-container-registry-on-your-azure-stack-edge-pro-gpu-device"></a>在 Azure Stack Edge Pro GPU 设备上启用边缘容器注册表

本文介绍如何启用边缘容器注册表，以及如何在 Azure Stack Edge Pro 设备上的 Kubernetes 群集中使用它。 本文中使用的示例详细介绍了如何将映像从源注册表（在本例中为 Microsoft 容器注册表）推送到 Azure Stack 边缘设备上的注册表（边缘容器注册表）。

### <a name="about-edge-container-registry"></a>关于边缘容器注册表

容器化计算应用程序在容器映像上运行，这些映像存储在注册表中。 注册表可以是公共的，例如 Docker 中心、私有或云提供程序（如 Azure 容器注册表）。 有关详细信息，请参阅 [关于注册表、存储库和映像](../container-registry/container-registry-concepts.md)。

边缘容器注册表在 Azure Stack Edge Pro 设备的边缘提供一个存储库。 可以使用此注册表来存储和管理专用容器映像。

在多节点环境下，可以下载容器映像并将其推送到边缘容器注册表一次。 所有边缘应用程序均可使用边缘容器注册表进行后续部署。


## <a name="prerequisites"></a>先决条件

在开始之前，请确保：

1. 你可以访问 Azure Stack Edge Pro 设备。

1. 已按照[激活 Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md) 中所述的内容激活 Azure Stack Edge Pro 设备。

1. 已在设备上启用了计算角色。 根据在 [Azure Stack Edge Pro 设备上配置计算](azure-stack-edge-gpu-deploy-configure-compute.md)中的说明，在设备上配置计算时，还会在设备上创建 Kubernetes 群集。

1. 你具有本地 web UI 的 " **设备** " 页中的 Kubernetes API 终结点。 有关详细信息，请参阅 [Get KUBERNETES API 终结点](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints)中的说明。

1. 你可以使用 [受支持的操作系统](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device)访问客户端系统。 如果使用 Windows 客户端，则系统应运行 PowerShell 5.0 或更高版本来访问设备。

    1. 如果要请求并推送自己的容器映像，请确保系统已安装 Docker 客户端。 如果使用 Windows 客户端，请 [在 windows 上安装 Docker Desktop](https://docs.docker.com/docker-for-windows/install/)。  


## <a name="enable-container-registry-as-add-on"></a>启用容器注册表作为加载项

第一步是启用边缘容器注册表作为外接程序。

1. [连接到设备的 PowerShell 接口](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface)。 

1. 若要启用容器注册表作为外接程序，请键入： 

    `Set-HcsKubernetesContainerRegistry`
    
    此操作可能需要几分钟才能完成。

    下面是此命令的示例输出：  
            
    ```powershell
    [10.128.44.40]: PS>Set-HcsKubernetesContainerRegistry
    Operation completed successfully. Use Get-HcsKubernetesContainerRegistryInfo for credentials    
    ```
            
1. 若要获取容器注册表的详细信息，请键入：

    `Get-HcsKubernetesContainerRegistryInfo`

    下面是此命令的示例：  
    
    ```powershell
    [10.128.44.40]: PS> Get-HcsKubernetesContainerRegistryInfo
                
    Endpoint                                   IPAddress    Username     Password
    --------                                   ---------    --------     --------
    ecr.dbe-hw6h1t2.microsoftdatabox.com:31001 10.128.44.41 ase-ecr-user i3eTsU4zGYyIgxV
    ``` 

1. 记下的 "用户名" 和 "密码" `Get-HcsKubernetesContainerRegistryInfo` 。 这些凭据用于在推送映像时登录到边缘容器注册表。         


## <a name="manage-container-registry-images"></a>管理容器注册表映像

你可能想要从 Azure Stack 边缘设备之外访问容器注册表。 你可能还需要在注册表中推送或请求映像。

按照以下步骤访问边缘容器注册表：

1. 获取边缘容器注册表的终结点详细信息。
    1. 在设备的本地 UI 中，中转到 " **设备**"。
    1. 找到 **边缘容器注册表终结点**。
        ![设备页上的边缘容器注册表终结点](media/azure-stack-edge-gpu-edge-container-registry/get-edge-container-registry-endpoint-1.png) 
    1. 复制此终结点，并在客户端的文件中创建相应的 DNS 条目， `C:\Windows\System32\Drivers\etc\hosts` 以连接到 Edge 容器注册表终结点。 

        <IP address of the Kubernetes main node>    <Edge container registry endpoint> 
        
        ![添加边缘容器注册表终结点的 DNS 条目](media/azure-stack-edge-gpu-edge-container-registry/add-domain-name-service-entry-hosts-1.png)    

1. 从本地 UI 下载 Edge 容器注册表证书。 
    1. 在设备的本地 UI 中，请参阅 " **证书**"。
    1. 找到 " **边缘容器注册表证书**" 条目。 在此项的右侧，选择 " **下载** " 以下载客户端系统上用于访问设备的边缘容器注册表证书。 

        ![下载边缘容器注册表终结点证书](media/azure-stack-edge-gpu-edge-container-registry/download-edge-container-registry-endpoint-certificate-1.png)  

1. 将下载的证书安装在客户端上。 如果使用 Windows 客户端，请执行以下步骤： 
    1. 选择证书，然后在 **证书导入向导** 中，选择 "将位置存储为 **本地计算机**"。 

        ![安装证书1](media/azure-stack-edge-gpu-edge-container-registry/install-certificate-1.png) 
    
    1. 在受信任的根存储区中的本地计算机上安装证书。 

        ![安装证书2](media/azure-stack-edge-gpu-edge-container-registry/install-certificate-2.png) 

1. 安装证书后，重新启动系统上的 Docker 客户端。

1. 登录到边缘容器注册表。 类型：

    `docker login <Edge container registry endpoint> -u <username> -p <password>`

    从 " **设备** " 页提供边缘容器注册表终结点，以及从的输出中获取的用户名和密码 `Get-HcsKubernetesContainerRegistryInfo` 。 

1. 使用 docker push 或 pull 命令从容器注册表推送或提取容器映像。
 
    1. 从 Microsoft 容器注册表映像中提取映像。 类型：
        
        `docker pull <Full path to the container image in the Microsoft Container Registry>`
       
    1. 使用注册表的完全限定路径创建请求的映像的别名。

        `docker tag <Path to the image in the Microsoft container registry> <Path to the image in the Edge container registry/Image name with tag>`

    1. 将映像推送到注册表。
    
        `docker push <Path to the image in the Edge container registry/Image name with tag>`

    1. 运行推送到注册表中的映像。
    
        `docker run -it --rm -p 8080:80 <Path to the image in the Edge container registry/Image name with tag>`

        下面是请求和推送命令的示例输出：

        ```powershell
        PS C:\WINDOWS\system32> docker login ecr.dbe-hw6h1t2.microsoftdatabox.com:31001 -u ase-ecr-user -p 3bbo2sOtDe8FouD
        WARNING! Using --password via the CLI is insecure. Use --password-stdin.
        Login Succeeded
        PS C:\WINDOWS\system32> docker pull mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        1.17.5-alpine: Pulling from oss/nginx/nginx
        Digest: sha256:5466bbc0a989bd1cd283c0ba86d9c2fc133491ccfaea63160089f47b32ae973b
        Status: Image is up to date for mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        PS C:\WINDOWS\system32> docker tag mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        PS C:\WINDOWS\system32> docker push ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        The push refers to repository [ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx]
        bba7d2385bc1: Pushed
        77cae8ab23bf: Pushed
        2.0: digest: sha256:b4c0378c841cd76f0b75bc63454bfc6fe194a5220d4eab0d75963bccdbc327ff size: 739
        PS C:\WINDOWS\system32> docker run -it --rm -p 8080:80 ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        2020/11/10 00:00:49 [error] 6#6: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"
        172.17.0.1 - - [10/Nov/2020:00:00:49 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36" "-"
        ^C
        PS C:\WINDOWS\system32>    
        ```
    
1. 浏览到 `http://localhost:8080` 以查看正在运行的容器。 在这种情况下，你将看到 nginx web 服务器正在运行。

    ![查看正在运行的容器](media/azure-stack-edge-gpu-edge-container-registry/view-running-container-1.png)

    若要停止并删除容器，请按 `Control+C`。

 

## <a name="use-edge-container-registry-images-via-kubernetes-pods"></a>通过 Kubernetes pod 使用边缘容器注册表映像

你现在可以从 Kubernetes pod 中部署推送到 Edge 容器注册表中的映像。

1. 若要部署映像，需要通过 *kubectl* 配置群集访问。 创建命名空间、用户、向用户授予对命名空间的访问权限，并获取 *配置* 文件。 请确保可以连接到 Kubernetes pod。 
    
    按照 [Azure Stack Edge PRO GPU 设备上的 kubectl 中的 "连接到" 和 "管理 Kubernetes 群集](azure-stack-edge-gpu-create-kubernetes-cluster.md)" 中的所有步骤进行操作。 

    下面是设备上的命名空间的示例输出，用户可在其中访问 Kubernetes 群集。

    ```powershell
    [10.128.44.40]: PS>New-HcsKubernetesNamespace -Namespace myecr
    [10.128.44.40]: PS>New-HcsKubernetesUser -UserName ecruser
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01URXdOVEF6TkRJek1Gb1hEVE13TVRFd016QXpOREl6TUZvd0ZURVRNQkVnNjOVRLWndCQ042cm1XQms2eXFwcXI1MUx6bApTaXMyTy91UEJ2YXNSSUUzdzgrbmEwdG1aTERZZ2F6MkQwMm42Q29mUmtyUTR2d1lLTnR1MlpzR3pUdz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
        server: https://compute.dbe-hw6h1t2.microsoftdatabox.com:6443
        name: kubernetes
        ===================CUT=========================================CUT==============
        client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwRENDQWJpZ0F3SUJBZ0lJYmVWRGJSTzZ3ell3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURFeE1EVXdNelF5TXpCYUZ3MHlNVEV4TURreU16UTRNal
        ===================CUT=========================================CUT==============
        DMVUvN3lFOG5UU3k3b2VPWitUeHdzCjF1UDByMjhDZ1lCdHdRY0ZpcFh1blN5ak16dTNIYjhveFI2V3VWWmZldFFKNElKWEFXOStVWGhKTFhyQ2x4bUcKWHRtbCt4UU5UTzFjQVNKRVZWVDd6Tjg2ay9kSU43S3JIVkdUdUxlUDd4eGVjV2VRcWJrZEVScUsxN0liTXpiVApmbnNxc0dobEdmLzdmM21kTGtyOENrcWs5TU5aM3MvUVIwRlFCdk94ZVpuUlpTeDVGbUR5S1E9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

    [10.128.44.40]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace myecr -UserName ecruser
    [10.128.44.40]: PS>kubectl get pods -n "myecr"
    No resources found.
    PS C:\WINDOWS\system32>
    ```  

2. 已在设备上的所有 Kubernetes 命名空间中设置图像提取机密。 你可以使用命令获取机密 `get secrets` 。 下面是示例输出：

    ```powershell
    PS C:\WINDOWS\system32> .\kubectl.exe get secrets -n myecr
    NAME                  TYPE                                  DATA   AGE
    ase-ecr-credentials   kubernetes.io/dockerconfigjson        1      99m
    default-token-c7kww   kubernetes.io/service-account-token   3      107m
    sec-smbcredentials    microsoft.com/smb                     2      99m
    PS C:\WINDOWS\system32>   
    ```    

3. 使用 kubectl 将 pod 部署到命名空间。 使用以下项 `yaml` 。 

    替换映像： `<image-name>` 并将映像推送到容器注册表。 使用名为的 imagePullSecrets 引用命名空间中的机密 `ase-ecr-credentials` 。
    
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        imagePullPolicy: Always
      imagePullSecrets:
      - name: ase-ecr-credentials
    ```

4. 使用 apply 命令在创建的命名空间中应用部署。 验证容器是否正在运行。 下面是示例输出：
   
    ```yml
    PS C:\Windows\System32> .\kubectl.exe apply -f .\deployment.yml -n myecr
    pod/nginx configured
    PS C:\Windows\System32> .\kubectl.exe get pods -n myecr
    NAME    READY   STATUS    RESTARTS   AGE
    nginx   1/1     Running   0          27m
    PS C:\Windows\System32>
    ```

## <a name="delete-container-registry-images"></a>删除容器注册表映像

边缘容器注册表存储位于 Azure Stack Edge Pro 设备内的本地共享上，该共享受设备上的可用存储的限制。 你负责使用 Docker HTTP v2 API (从容器注册表中删除未使用的 docker 映像 https://docs.docker.com/registry/spec/api/) 。  

若要删除一个或多个容器映像，请执行以下步骤：

1. 将映像名称设置为要删除的映像。

    ```powershell
    PS C:\WINDOWS\system32> $imageName="nginx"    
    ```

1. 将容器注册表的用户名和密码设置为 PS 凭据

    ```powershell
    PS C:\WINDOWS\system32> $username="ase-ecr-user"
    PS C:\WINDOWS\system32> $password="3bbo2sOtDe8FouD"
    PS C:\WINDOWS\system32> $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
    PS C:\WINDOWS\system32> $credential = New-Object System.Management.Automation.PSCredential ($username, $securePassword)    
    ```

1. 列出与图像关联的标记

    ```powershell
    PS C:\WINDOWS\system32> $tags = Invoke-RestMethod -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/nginx/tags/list" | Select-Object -ExpandProperty tags
    PS C:\WINDOWS\system32> $tags
    2.0
    PS C:\WINDOWS\system32> $tags = Invoke-RestMethod -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/tags/list" | Select-Object -ExpandProperty tags
    PS C:\WINDOWS\system32> $tags
    2.0
    PS C:\WINDOWS\system32>    
    ```
  
1. 列出与你要删除的标记关联的摘要。 这将使用以上命令的输出中的 $tags。 如果有多个标记，请选择其中一个，然后在下一个命令中使用。

    ```powershell
    PS C:\WINDOWS\system32> $response = Invoke-WebRequest -Method Head -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/manifests/$tags" -Headers @{ 'Accept' = 'application/vnd.docker.distribution.manifest.v2+json' }
    PS C:\WINDOWS\system32> $digest = $response.Headers['Docker-Content-Digest']
    PS C:\WINDOWS\system32> $digest
    sha256:b4c0378c841cd76f0b75bc63454bfc6fe194a5220d4eab0d75963bccdbc327ff
    PS C:\WINDOWS\system32>    
    ```
1. 使用映像摘要：标记删除映像

    ```powershell
    PS C:\WINDOWS\system32> Invoke-WebRequest -Method Delete -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/manifests/$digest" | Select-Object -ExpandProperty StatusDescription    
    ```

删除未使用的映像后，与未引用的映像关联的空间将由每夜运行的进程自动回收。 

## <a name="next-steps"></a>后续步骤

- [在 Azure Stack Edge Pro 上部署无状态应用程序](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md)。
