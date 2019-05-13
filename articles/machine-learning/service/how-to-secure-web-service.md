---
title: 使用 SSL 保护 Web 服务
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务部署通过启用 HTTPS 的 web 服务安全。 HTTPS 保护通过使用传输层安全性 (TLS) 安全套接字层 (SSL) 的替换客户端提交的数据。 它还用于通过客户端验证 web 服务的标识。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: aashishb
author: aashishb
ms.date: 04/29/2019
ms.custom: seodec18
ms.openlocfilehash: 50e42172af6ca6b966f9f60d3e037f9ae3dc5cbe
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65023765"
---
# <a name="use-ssl-to-secure-web-services-with-azure-machine-learning-service"></a>通过 Azure 机器学习服务使用 SSL 保护 Web 服务

在本文中，你将了解如何保护使用 Azure 机器学习服务部署的 Web 服务。 可以限制对 web 服务的访问和保护通过使用客户端提交的数据[超文本传输协议安全 (HTTPS)](https://en.wikipedia.org/wiki/HTTPS)。

使用 HTTPS 来保护客户端和 web 服务之间的通信进行加密两者之间的通信。 使用处理加密[传输层安全性 (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security)。 有时 TLS 仍指代为安全套接字层 (SSL)，这是 TLS 的前身。

> [!TIP]
> Azure 机器学习 SDK 使用术语 SSL 属性与启用安全通信。 这并不意味着您的 web 服务不使用 TLS，只是该 SSL 是为许多读者更易识别的术语。

TLS 和 SSL 都依赖__数字证书__，用于执行加密和身份验证。 数字的证书的工作方式的详细信息，请参阅 Wikipedia 条目上[公钥基础结构 (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure)。

> [!Warning]
> 如果不执行操作启用并针对您的 web 服务使用 HTTPS，发往和来自该服务的数据可能会给其他人在 internet 上可见。
>
> HTTPS 还使客户端，以验证连接到服务器的真实性。 这可避免对客户端[拦截](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)攻击。

保护新的 web 服务或一个现有的一般过程如下所示：

1. 获取域名。

2. 获取数字证书。

3. 部署或更新启用了 SSL 设置的 Web 服务。

4. 更新 DNS，使其指向该 Web 服务。

> [!IMPORTANT]
> 如果要部署到 Azure Kubernetes 服务 (AKS)，可以提供自己的证书或使用由 Microsoft 提供的证书。 如果使用 Microsoft 提供的证书，您不需要获取的域名或 SSL 证书。 有关详细信息，请参阅[启用 SSL 并部署](#enable)部分。

跨[部署目标](how-to-deploy-and-where.md)保护 Web 服务时，步骤稍有不同。

## <a name="get-a-domain-name"></a>获取域名

如果还没有属于自己的域名，可从域名注册机构购买一个。 不同注册机构的流程不同，价格也不同。 注册机构还会提供管理域名的工具。 这些工具用于映射的完全限定的域名 (例如 www\.contoso.com) 到承载 web 服务的 IP 地址。

## <a name="get-an-ssl-certificate"></a>获取 SSL 证书

有许多方法来获取的 SSL 证书 （数字证书）。 最常用的方式是从证书颁发机构 (CA) 购买。 无论证书来自哪里，都需要以下文件：

* 证书。 证书必须包含完整的证书链，并必须使用 PEM 编码。
* 密钥。 密钥必须使用 PEM 编码。

请求证书时，必须提供 Web 服务打算使用的地址的完全限定域名 (FQDN)。 例如，www\.contoso.com。 验证 Web 服务的身份时，会对比证书上标记的地址和客户端使用的地址。 如果两个地址不匹配，客户端将收到错误。

> [!TIP]
> 如果证书颁发机构不能以 PEM 编码文件的方式提供证书和密钥，可使用 [OpenSSL](https://www.openssl.org/) 等实用程序来更改格式。

> [!WARNING]
> 自签名证书只能用于开发。 不应在产品中使用。 自签名证书可能会导致客户端应用程序出现问题。 有关详细信息，客户端应用程序中使用的网络库的文档。

## <a id="enable"></a> 启用 SSL 和部署

若要部署 （或重新部署） 服务已启用 ssl，设置`ssl_enabled`参数`True`、 适用。 将 `ssl_certificate` 参数设置为证书文件的值，将 `ssl_key` 设置为密钥文件的值。

+ **Visual 接口-创建安全 Azure Kubernetes 服务 (AKS) 部署** 
    
    如果你尝试创建可视界面的安全部署计算，请参阅此。 预配 AKS 群集，同时为与 SSL 相关的参数提供值，然后创建新的 AKS。  请参阅下面的代码段：
    

    > [!TIP]
    >  如果您不熟悉 Python SDK，从开始[Azure 机器学习 Python SDK 概述。](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)


    ```python
    from azureml.core.compute import AksCompute, ComputeTarget

    # Provide SSL-related parameters when provisioning the AKS cluster
    prov_config = AksCompute.provisioning_configuration(ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")   
 
    aks_name = 'secure-aks'
    # Create the cluster
    aks_target = ComputeTarget.create(workspace = ws,
                                        name = aks_name,
                                        provisioning_configuration = prov_config)
    
    # Wait for the create process to complete
    aks_target.wait_for_completion(show_output = True)
    print(aks_target.provisioning_state)
    print(aks_target.provisioning_errors)
    ```
    
   

+ **在 Azure Kubernetes 服务 (AKS) 上部署和 FPGA**

  部署到 AKS 时，可以创建新的 AKS 群集或附加现有。 创建新群集使用[AksCompute.provisionining_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#provisioning-configuration-agent-count-none--vm-size-none--ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--location-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--service-cidr-none--dns-service-ip-none--docker-bridge-cidr-none-)而附加现有的群集使用[AksCompute.attach_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py#attach-configuration-resource-group-none--cluster-name-none--resource-id-none-)。 两者都返回配置对象具有`enable_ssl`方法。

  `enable_ssl`方法可以使用 Microsoft 或您提供的一个由提供的证书。

  * 当使用证书__由 Microsoft 提供__，则必须使用`leaf_domain_label`参数。 使用此参数将创建使用由 Microsoft 提供的证书的服务。 `leaf_domain_label`用于生成该服务的 DNS 名称。 例如，值`myservice`创建的域名`myservice<6-random-characters>.<azureregion>.cloudapp.azure.com`，其中`<azureregion>`是包含该服务的区域。 （可选） 可以使用`overwrite_existing_domain`参数来覆盖现有叶域标签。

    若要部署（或重新部署）启用了 SSL 的服务，请根据情况将 `ssl_enabled` 参数设置为 `True`。 将 `ssl_certificate` 参数设置为证书文件的值，将 `ssl_key` 设置为密钥文件的值。

    > [!IMPORTANT]
    > 使用 Microsoft 提供的证书时，您不需要购买自己的证书或域的名称。

    下面的示例演示如何创建启用 SSL 证书由 Microsoft 创建的配置：

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable SSL
    provisioning_config = AksCompute.provisioning_configuration().enable_ssl(leaf_domain_label = "myservice")
    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                          cluster_name = cluster_name).enable_ssl(leaf_domain_label = "myservice")
    ```

  * 使用时__购买的证书__，使用`ssl_cert_pem_file`， `ssl_key_pem_file`，和`ssl_cname`参数。  下面的示例演示如何创建使用你提供使用的 SSL 证书的配置`.pem`文件：

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable SSL
    provisioning_config = AksCompute.provisioning_configuration(ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    provisioning_config = AksCompute.provisioning_configuration().enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name).enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

  有关详细信息`enable_ssl`，请参阅[AksProvisioningConfiguration.enable_ssl()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksprovisioningconfiguration?view=azure-ml-py#enable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-)并[AksAttachConfiguration.enable_ssl()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.aks.aksattachconfiguration?view=azure-ml-py#enable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-)。

+ **在 Azure 容器实例 (ACI) 上进行部署**

  部署到 ACI 时，请为 SSL 相关参数提供值，如以下代码片段中所示：

    ```python
    from azureml.core.webservice import AciWebservice

    aci_config = AciWebservice.deploy_configuration(ssl_enabled=True, ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

  有关详细信息，请参阅[AciWebservice.deploy_configuration()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none-)。

## <a name="update-your-dns"></a>更新 DNS

接下来，必须更新 DNS，使其指向该 Web 服务。

+ **对于 ACI**：

  使用域名注册机构提供的工具来更新域名的 DNS 记录。 该记录必须指向服务的 IP 地址。

  可能几分钟到几小时之后客户端才能解析域名，具体取决于注册机构和为域名配置的生存时间 (TTL)。

+ **对于 AKS**：

  > [!WARNING]
  > 如果您使用`leaf_domain_label`要使用由 Microsoft 提供的证书创建服务，不手动更新群集的 DNS 值。 应会自动设置的值。

  在 AKS 群集的“公用 IP 地址”的“配置”选项卡下更新 DNS，如下图中所示。 可以在包含 AKS 代理节点和其他网络资源的资源组下找到公用 IP 地址，它是该资源组中创建的资源类型之一。

  ![Azure 机器学习服务：使用 SSL 保护 Web 服务](./media/how-to-secure-web-service/aks-public-ip-address.png)



## <a name="next-steps"></a>后续步骤
了解如何：
+ [使用部署为 Web 服务的机器学习模型](how-to-consume-web-service.md)
+ [在 Azure 虚拟网络中安全运行试验和推断](how-to-enable-virtual-network.md)

