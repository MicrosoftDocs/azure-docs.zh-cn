---
title: Azure 云服务 (扩展支持) NetworkConfiguration 架构 |Microsoft Docs
description: '与云服务的网络配置架构相关的信息 (扩展支持) '
ms.topic: article
ms.service: cloud-services-extended-support
ms.date: 10/14/2020
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 13fe83710c94c1ca37f05d59cb91f31aa8ca1bae
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "98744265"
---
# <a name="azure-cloud-services-extended-support-config-networkconfiguration-schema"></a>Azure 云服务 (扩展支持) config networkConfiguration 架构

服务配置文件的 `NetworkConfiguration` 元素指定虚拟网络和 DNS 值。 对于云服务，这些设置是可选的。

可以通过以下资源深入了解虚拟网络和关联的架构：

- [云服务 (扩展支持) 配置架构](schema-cscfg-file.md)。
- [云服务 (扩展支持) 定义架构](schema-csdef-file.md)。
- [创建虚拟网络](https://docs.microsoft.com/azure/virtual-network/manage-virtual-network)。

## <a name="networkconfiguration-element"></a>NetworkConfiguration 元素
下面的示例显示了 `NetworkConfiguration` 元素及其子元素。

```xml
<ServiceConfiguration>
  <NetworkConfiguration>
    <AccessControls>
      <AccessControl name="aclName1">
        <Rule order="<rule-order>" action="<rule-action>" remoteSubnet="<subnet-address>" description="rule-description"/>
      </AccessControl>
    </AccessControls>
    <EndpointAcls>
      <EndpointAcl role="<role-name>" endpoint="<endpoint-name>" accessControl="<acl-name>"/>
    </EndpointAcls>
    <Dns>
      <DnsServers>
        <DnsServer name="<server-name>" IPAddress="<server-address>" />
      </DnsServers>
    </Dns>
    <VirtualNetworkSite name="Group <RG-VNet> <VNet-name>"/>
    <AddressAssignments>
      <InstanceAddress roleName="<role-name>">
        <Subnets>
          <Subnet name="<subnet-name>"/>
        </Subnets>
      </InstanceAddress>
      <ReservedIPs>
        <ReservedIP name="<reserved-ip-name>"/>
      </ReservedIPs>
    </AddressAssignments>
  </NetworkConfiguration>
</ServiceConfiguration>
```

下表介绍了 `NetworkConfiguration` 元素的子元素。

| 元素       | 说明 |
| ------------- | ----------- |
| AccessControl | 可选。 指定在云服务中访问终结点的规则。 访问控制名称由 `name` 属性的字符串定义。 `AccessControl` 元素包含一个或多个 `Rule` 元素。 可定义多个 `AccessControl` 元素。|
| 规则 | 可选。 指定应对指定的 IP 地址子网范围执行的操作。 规则的顺序由 `order` 属性的字符串值定义。 规则编号越低，优先级越高。 例如，可使用序号 100、200 和 300 指定规则。 序号为 100 的规则优先于序号为 200 的规则。<br /><br /> 规则的操作由 `action` 属性的字符串定义。 可能的值包括：<br /><br /> -   `permit` – 指定只有指定子网范围中的数据包才可以与终结点通信。<br />-   `deny` – 指定拒绝对指定子网范围中终结点的访问。<br /><br /> 受规则影响的 IP 地址的子网范围由 `remoteSubnet` 属性的字符串定义。 对规则的说明由 `description` 属性的字符串定义。|
| EndpointAcl | 可选。 指定向终结点分配访问控制规则。 包含终结点的角色的名称由 `role` 属性的字符串定义。 终结点的名称由 `endpoint` 属性的字符串定义。 对于应该应用到终结点的 `AccessControl` 规则的集合，其名称在 `accessControl` 属性的字符串中定义。 可定义多个 `EndpointAcl` 元素。|
| DnsServer | 可选。 指定 DNS 服务器的设置。 可以指定不使用虚拟网络的 DNS 服务器的设置。 DNS 服务器的名称由 `name` 属性的字符串定义。 DNS 服务器的 IP 地址由 `IPAddress` 属性的字符串定义。 该 IP 地址必须是有效的 IPv4 地址。|
| VirtualNetworkSite | 可选。 指定要在其中部署云服务的虚拟网络站点的名称。 此设置不会创建虚拟网络站点。 它引用之前已在虚拟网络的网络文件中定义的站点。 云服务只能是一个虚拟网络的成员。 如果未指定此设置，则不会将云服务部署到虚拟网络。 虚拟网络站点的名称由 `name` 属性的字符串定义。|
| InstanceAddress | 可选。 指定角色与虚拟网络中的子网或子网集的关联。 将角色名称关联到实例地址时，可以指定要将此角色与之关联的子网。 `InstanceAddress` 包含 Subnets 元素。 与一个或多个子网相关联的角色的名称由 `roleName` 属性的字符串定义。|
| 子网 | 可选。 指定与网络配置文件中的子网名称相对应的子网。 子网的名称由 `name` 属性的字符串定义。|
| ReservedIP | 可选。 指定应与部署关联的保留 IP 地址。 必须使用“创建保留 IP 地址”创建保留的 IP 地址。 云服务中的每个部署都可以与一个保留 IP 地址相关联。 保留 IP 地址的名称由 `name` 属性的字符串定义。|

## <a name="see-also"></a>另请参阅
[云服务 (扩展支持) 配置架构](schema-cscfg-file.md)。