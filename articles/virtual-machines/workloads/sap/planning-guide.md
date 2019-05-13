---
title: SAP NetWeaver 的 Azure 虚拟机规划和实施指南 | Microsoft Docs
description: SAP NetWeaver 的 Azure 虚拟机规划和实施指南
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: d7c59cc1-b2d0-4d90-9126-628f9c7a5538
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/07/2019
ms.author: sedusch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2ddcf1f38d3d92f9d9bdd12203ebf99f20600478
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65409769"
---
# <a name="azure-virtual-machines-planning-and-implementation-for-sap-netweaver"></a>SAP NetWeaver 的 Azure 虚拟机规划和实施指南

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1139904]:https://launchpad.support.sap.com/#/notes/1139904
[1173395]:https://launchpad.support.sap.com/#/notes/1173395
[1245200]:https://launchpad.support.sap.com/#/notes/1245200
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1558958]:https://launchpad.support.sap.com/#/notes/1558958
[1585981]:https://launchpad.support.sap.com/#/notes/1585981
[1588316]:https://launchpad.support.sap.com/#/notes/1588316
[1590719]:https://launchpad.support.sap.com/#/notes/1590719
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1605680]:https://launchpad.support.sap.com/#/notes/1605680
[1619720]:https://launchpad.support.sap.com/#/notes/1619720
[1619726]:https://launchpad.support.sap.com/#/notes/1619726
[1619967]:https://launchpad.support.sap.com/#/notes/1619967
[1750510]:https://launchpad.support.sap.com/#/notes/1750510
[1752266]:https://launchpad.support.sap.com/#/notes/1752266
[1757924]:https://launchpad.support.sap.com/#/notes/1757924
[1757928]:https://launchpad.support.sap.com/#/notes/1757928
[1758182]:https://launchpad.support.sap.com/#/notes/1758182
[1758496]:https://launchpad.support.sap.com/#/notes/1758496
[1772688]:https://launchpad.support.sap.com/#/notes/1772688
[1814258]:https://launchpad.support.sap.com/#/notes/1814258
[1882376]:https://launchpad.support.sap.com/#/notes/1882376
[1909114]:https://launchpad.support.sap.com/#/notes/1909114
[1922555]:https://launchpad.support.sap.com/#/notes/1922555
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1941500]:https://launchpad.support.sap.com/#/notes/1941500
[1956005]:https://launchpad.support.sap.com/#/notes/1956005
[1973241]:https://launchpad.support.sap.com/#/notes/1973241
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[azure-cli]:../../../cli-install-nodejs.md
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azureps-cmdlets-docs
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-subscription-service-limits]:../../../azure-subscription-service-limits.md
[azure-subscription-service-limits-subscription]:../../../azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide.md
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]:deployment-guide.md
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8

[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d
[deployment-guide-figure-100]:media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]:media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]:deployment-guide.md#figure-11
[deployment-guide-figure-1100]:media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]:media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]:media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]:deployment-guide.md#figure-14
[deployment-guide-figure-1400]:media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]:media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]:media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]:deployment-guide.md#figure-5
[deployment-guide-figure-50]:media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]:media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]:deployment-guide.md#figure-6
[deployment-guide-figure-600]:media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]:deployment-guide.md#figure-7
[deployment-guide-figure-700]:media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]:media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]:media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]:deployment-guide.md#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]:deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]:deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b

[deploy-template-cli]:../../../resource-group-template-deploy-cli.md
[deploy-template-portal]:../../../resource-group-template-deploy-portal.md
[deploy-template-powershell]:../../../resource-group-template-deploy.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[getting-started-dbms]:get-started.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]:get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]:get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]:../../virtual-machines-windows-classic-sap-get-started.md
[getting-started-windows-classic-dbms]:../../virtual-machines-windows-classic-sap-get-started.md#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]:../../virtual-machines-windows-classic-sap-get-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]:../../virtual-machines-windows-classic-sap-get-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]:../../virtual-machines-windows-classic-sap-get-started.md#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]:../../virtual-machines-windows-classic-sap-get-started.md#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]:https://go.microsoft.com/fwlink/?LinkId=613056

[install-extension-cli]:virtual-machines-linux-enable-aem.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-Azvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

[planning-guide]:planning-guide.md  
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77
[planning-guide-3.3.2]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d
[planning-guide-7.1]:planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30
[planning-guide-7]:planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3
[planning-guide-azure-premium-storage]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92

[planning-guide-figure-100]:media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]:media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]:media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]:media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]:media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]:media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]:media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]:media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]:media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]:media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]:media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/planning-monitoring-overview-2502.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-2901]:media/virtual-machines-shared-sap-planning-guide/2901-azure-ha-sap-ha-md.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-3201]:media/virtual-machines-shared-sap-planning-guide/3201-sap-ha-with-sql-md.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[powershell-install-configure]:https://docs.microsoft.com/powershell/azure/install-az-ps
[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/resource-group-overview.md
[resource-groups-networking]:../../../networking/networking-overview.md
[sap-pam]:https://support.sap.com/pam
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[storage-azure-cli]:../../../storage/common/storage-azure-cli.md
[storage-azure-cli-copy-blobs]:../../../storage/common/storage-azure-cli.md#copy-blobs
[storage-introduction]:../../../storage/common/storage-introduction.md
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md
[storage-premium-storage-preview-portal]:../../windows/disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/storage-scalability-targets.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-Az-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-windows.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-windows-capture-image-resource-manager]:../../windows/capture-image.md
[virtual-machines-windows-capture-image]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-windows-capture-image-prepare-the-vm-for-image-capture]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-create-upload-vhd-oracle]:../../linux/oracle-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability]:../../linux/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:./../../windows/sql/virtual-machines-windows-sql-high-availability-dr.md
[virtual-machines-sql-server-infrastructure-services]:./../../windows/sql/virtual-machines-windows-sql-server-iaas-overview.md
[virtual-machines-sql-server-performance-best-practices]:./../../windows/sql/virtual-machines-windows-sql-performance.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]:../../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics-windows]:../../windows/multiple-nics.md
[virtual-networks-multiple-nics-linux]:../../linux/multiple-nics.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-about-vpngateways.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md
[capture-image-linux-step-2-create-vm-image]:../../linux/capture-image.md#step-2-create-vm-image

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

公司可以使用 Microsoft Azure 在最短的时间内获取计算和存储资源，而无需经历冗长的采购周期。 Azure 虚拟机服务可让公司将典型的应用程序（例如，基于 SAP NetWeaver 的应用程序）部署到 Azure 中，并提高其可靠性和可用性，且不需要在本地提供其他资源。 Azure 虚拟机服务还支持跨场地连接，使公司能够主动地将 Azure 虚拟机集成到其本地域、私有云和 SAP 系统布局中。
本白皮书介绍 Microsoft Azure 虚拟机的基础知识，并以演练的方式提供有关在 Azure 中规划和实施 SAP NetWeaver 安装的注意事项，因此，在 Azure 中实际部署 SAP NetWeaver 之前，应该先阅读本文档。
本文是对 SAP 安装文档和 SAP 说明的补充，这些文档代表在给定平台上安装和部署 SAP 软件的主要资源。

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="summary"></a>摘要
云计算是一个广泛使用的术语，它在 IT 行业（从小公司到大型跨国企业）中受到越来越多的重视。

Microsoft Azure 是 Microsoft 提供的一个云服务平台，它提供了各种新的可能性。 现在，客户可以快速地将应用程序设置和取消设置为云中的服务，因此可以摆脱技术和预算方面的束缚。 公司不用在硬件基础结构中投入时间和预算，而是可以重点关注应用程序、业务流程以及它为客户和用户带来的好处。

借助 Microsoft Azure 虚拟机服务，Microsoft 提供了全面的基础结构即服务 (IaaS) 平台。 Azure 虚拟机 (IaaS) 支持基于 SAP NetWeaver 的应用程序。 本白皮书介绍在作为所选平台的 Microsoft Azure 中，如何规划和实现基于 SAP NetWeaver 的应用程序。

本白皮书本身重点介绍两个主要方面：

* 第一部分介绍 Azure 上基于 SAP NetWeaver 的应用程序支持的两种部署模式。 另外，还介绍在考虑 SAP 部署的同时如何一般性地处理 Azure。
* 第二部分详细介绍如何实施第一部分中所述的两种不同方案。

有关其他资源，请参阅本文档中的[资源][planning-guide-1.2]一章。

### <a name="definitions-upfront"></a>定义预释
整个文档使用以下术语：

* IaaS：基础结构即服务
* PaaS：平台即服务
* SaaS：软件即服务
* SAP 组件：单个 SAP 应用程序，例如 ECC、BW、Solution Manager 或 S/4HANA。  SAP 组件可以基于传统的 ABAP 或 Java 技术，也可以是不基于 NetWeaver 的应用程序，例如业务对象。
* SAP 环境：以逻辑方式组合在一起，用于执行开发、QAS、培训、DR 或生产等业务功能的一个或多个 SAP 组件。
* SAP 布局：表示客户 IT 布局中所有 SAP 资产的整个格局。 SAP 布局包括所有生产和非生产环境。
* SAP 系统：诸如 SAP ERP 开发系统、SAP BW 测试系统、SAP CRM 生产系统等的 DBMS 层与应用程序层的组合。在 Azure 部署中，不支持在本地和 Azure 之间分割这两个层。 这意味着，某个 SAP 系统要么部署在本地，要么部署在 Azure 中。 但是，可以将 SAP 布局中的不同系统部署到 Azure 或本地。 例如，可以将 SAP CRM 开发系统和测试系统部署在 Azure 中，同时将 SAP CRM 生产系统部署在本地。
* 跨界或混合：描述这样一种方案：将 VM 部署到在本地数据中心与 Azure 之间建立了站点到站点、多站点或 ExpressRoute 连接的 Azure 订阅。 在一般的 Azure 文档中，此类部署也称为跨界或混合方案。 连接的原因是为了将本地域、本地 Active Directory/OpenLDAP 和本地 DNS 扩展到 Azure。 本地布局会扩展到订阅的 Azure 资产。 经过这种扩展后，VM 可以成为本地域的一部分。 本地域的域用户可以访问服务器，并可在这些 VM 上运行服务（例如 DBMS 服务）。 但无法在本地的 VM 和 Azure 部署的 VM 之间进行通信和名称解析。 这是最常见的方案，几乎是将 SAP 资产部署到 Azure 的专用方案。 有关详细信息，请参阅[此文][vpn-gateway-cross-premises-options]和[此文][vpn-gateway-site-to-site-create]。

> [!NOTE]
> SAP 生产系统支持对 SAP 系统进行这种跨界或混合部署：运行 SAP 系统的 Azure 虚拟机是本地域的成员。 跨界或混合配置可将部分或完整 SAP 布局部署到 Azure。 即使在 Azure 中执行完整 SAP 布局，也需要这些 VM 成为本地域和 ADS/OpenLDAP 的一部分。 
>
>



### <a name="e55d1e22-c2c8-460b-9897-64622a34fdff"></a>资源
Azure 文档中 SAP 工作负荷的入口点位于[此处](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)。 从此入口点开始，会找到许多涵盖以下内容主题的文章：

- Azure 的 SAP NetWeaver 和 Business One
- Azure 中各种 DBMS 系统的 SAP DBMS 指南
- Azure 上的 SAP 工作负荷的高可用性和灾难恢复
- 有关在 Azure 上运行 SAP HANA 的特定指南
- 特定于 SAP HANA DBMS 的 Azure HANA 大型实例的指南 


> [!IMPORTANT]
> 在可能的情况下，请访问指向《SAP 安装指南》或其他 SAP 文档的链接（InstGuide-01 参考文档，请参阅 <http://service.sap.com/instguides>）。 涉及到特定 SAP 功能的先决条件、安装过程或详细信息，始终应仔细阅读 SAP 文档和指南，因为 Microsoft 文档仅涵盖在 Microsoft Azure 虚拟机中安装和操作的 SAP 软件的特定任务。
>
>

以下 SAP 说明与 Azure 上的 SAP 主题相关：

| 说明文档编号 | 标题 |
| --- | --- |
| [1928533] |Azure 上的 SAP 应用程序：支持的产品和大小 |
| [2015553] |Microsoft Azure 上的 SAP：支持先决条件 |
| [1999351] |适用于 SAP 的增强型 Azure 监视故障排除 |
| [2178632] |Microsoft Azure 上的 SAP 关键监视度量值 |
| [1409604] |Windows 上的虚拟化：增强型监视 |
| [2191498] |Azure 的 Linux 上的 SAP：增强型监视 |
| [2243692] |Microsoft Azure (IaaS) VM 上的 Linux：SAP 许可证问题 |
| [1984787] |SUSE LINUX Enterprise Server 12：安装说明 |
| [2002167] |Red Hat Enterprise Linux 7.x：安装和升级 |
| [2069760] |Oracle Linux 7.x SAP 安装和升级 |
| [1597355] |适用于 Linux 的交换空间建议 |

另请阅读 [SCN Wiki](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes)，其中包含适用于 Linux 的所有 SAP 说明。

可以在[此文][azure-subscription-service-limits-subscription]中找到有关 Azure 订阅的常规默认限制和最大限制。

## <a name="possible-scenarios"></a>可能的方案
SAP 通常被视为企业中最关键的应用程序之一。 通常，这些应用程序的体系结构和操作非常复杂，因此，符合可用性与性能方面的要求十分重要。

企业必须考虑仔细有关要选择用于运行此类业务的云提供程序关键业务流程上。 Azure 是业务关键 SAP 应用程序和业务流程的理想的公有云平台。 给定各种各样的 Azure 基础结构，几乎所有的现有 SAP NetWeaver 和 S/4HANA 系统可以在 Azure 中托管今天。 Azure 提供了许多千吉字节的内存和 Cpu，200 多个 Vm。 Azure 提供的更高版本[HANA 大型实例](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)，其允许的最大 24 TB 和向外缩放 ANA 横向扩展 HANA 部署最多 120 TB 的部署。 


要成功地将 SAP 系统部署到 Azure IaaS 或者一般的 IaaS，必须了解传统外购商或托管商的产品与 IaaS 产品之间的明显差异。 传统托管商或外包商适应客户想要承载的工作负荷的基础结构 （网络、 存储和服务器类型），而它负责改为客户或合作伙伴的特征化工作负荷和选择正确的 AzureVm、 存储和网络为 IaaS 部署的组件。

首先，客户需要确认以下事项：

* SAP 支持的 Azure VM 类型
* Azure 中 SAP 支持的产品/版本
* Azure 中特定 SAP 版本支持的 OS 和 DBMS 版本
* 不同的 Azure SKU 提供的 SAPS 吞吐量

可以在 SAP 说明 [1928533] 中找到这些问题的答案。

其次，需要将 Azure 资源和带宽限制与本地系统的实际资源消耗进行比较。 因此，客户需要熟悉 SAP 支持的 Azure 类型在以下方面的各项能力：

* 不同 VM 类型的 CPU 和内存资源
* 不同 VM 类型的 IOPS 带宽
* 不同 VM 类型的网络能力。

可在[此处 (Linux)][virtual-machines-sizes-linux] 和[此处 (Windows)][virtual-machines-sizes-windows]找到其中大部分数据。

请注意，上述链接中列出的限制是上限。 这并不意味着在所有情况下都能提供任何资源（例如 IOPS）的限制。 例外情况包括所选 VM 类型的 CPU 资源和内存。 对于 SAP 支持的 VM 类型，CPU 和内存资源是保留的，因此，这些资源在任意时间点都能够在 VM 中提供消耗。

Microsoft Azure 平台是多租户平台。 因此，存储、网络和其他资源在租户间是共享的。 其中使用了智能限制和配额逻辑来防止一个租户严重影响另一个租户的性能（干扰性邻居）。 尤其是对 Azure 平台的 SAP HANA 进行认证，Microsoft 需要证明位置的多个 Vm 可以运行定期在同一主机 SAP 的情况下的资源隔离。 尽管 Azure 中的逻辑会尝试在带宽中保留的方差可能会出现在资源/带宽可用性较大差异不是客户在其本地部署可能会遇到小的、 高度共享平台。 需要考虑 Azure 上 SAP 系统遇到的差异大于本地系统中的差异的可能性。

最后，需要评估可用性要求。 有时，底层 Azure 基础结构需要更新，并要求重新启动运行 VM 的主机。 Microsoft 文档中的不同用例[在 Azure 中的虚拟机的维护](https://docs.microsoft.com/azure/virtual-machines/windows/maintenance-and-updates)。 若要缓解极少数情况下，Vm 被强制重新启动，但您需要对修补程序来宾 OS 或 DBMS 组件，更重要的情况下进行开发所需在生产 SAP 系统有效的高可用性概念。 此要求不是任何不同于您面临着在本地的要求。 Microsoft 正在不断推进 Azure 平台，以减少平台更改所造成的停机时间。 

要成功将 SAP 系统部署到 Azure，本地 SAP 系统的操作系统、数据库和 SAP 应用程序必须出现在 SAP Azure 支持矩阵中、符合 Azure 基础结构可提供的资源限制，并且可配合 Microsoft Azure 提供的可用性 SLA 使用。 在确定这些系统后，需要决定采用下列两种部署方案中的哪一种。





### <a name="f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10"></a>跨界 - 将单个或多个 SAP VM 部署到 Azure 中并要求完全集成到本地网络
![使用站点到站点连接的 VPN（跨界）][planning-guide-figure-300]

这是一种跨界方案，存在多种可能的部署模式。 该方案就是在本地运行 SAP 布局的一部分，在 Azure 上运行 SAP 布局的另一部分。 部分 SAP 组件在 Azure 上运行这一事实的所有方面对于最终用户都应该是透明的。 因此，SAP 传输纠正系统 (STMS)、RFC 通信、打印、安全性（例如 SSO）等都可以对 Azure 上运行的 SAP 系统无差错地工作。 但是，跨界方案还描述了在 Azure 中运行完整 SAP 布局，并将客户的域和 DNS 扩展到 Azure 的方案。

> [!NOTE]
> 这是支持运行 SAP 生产系统的部署方案。
>
>

有关如何将本地网络连接到 Microsoft Azure 的详细信息，请参阅[此文][vpn-gateway-create-site-to-site-rm-powershell]

> [!IMPORTANT]
> 讨论 Azure 与本地客户部署之间的跨界方案时，请注意整个 SAP 系统的粒度。 不支持用于跨界方案的方案包括：
>
> * 以不同的部署方法运行不同的 SAP 应用程序层。 例如，在本地运行 DBMS 层，同时将 SAP 应用程序层作为 Azure VM 部署在 VM 中；反之亦然。
> * 将 SAP 层的一些组件部署在 Azure 中，将另一些组件部署在本地。 例如，在本地和 Azure VM 之间拆分 SAP 应用程序层的实例。
> * 不支持跨多个 Azure 区域分布运行某个系统的 SAP 实例的 VM。
>
> 施加这些限制的原因在于，要求在一个 SAP 系统中的应用程序实例与 DBMS 层之间提供低延迟高性能网络。
>
> 使用高度集成的多个 SAP 系统时，可能必须发生的系统和区域的特殊的规划。 请确保将部署这些系统尽可能地靠近彼此，尽量降低网络延迟。 高度集成 SAP 系统的示例包括：
> * SAP BW 从 ERP、 CRM 或 SRM; 等 SAP OLTP 系统中读取数据或
> * SAP SLT 正在用于将数据复制多个 SPA 系统之间、 甚至之间 SAP 和非 SAP 系统;或
> * SAP S/4 连接到 SAP ERP 系统;等等。


### <a name="supported-os-and-database-releases"></a>支持的 OS 和数据库版本
* 以下文章中列出了 Azure 虚拟机服务支持的 Microsoft 服务器软件：<https://support.microsoft.com/kb/2721672>。
* SAP 说明 [1928533] 中阐述了 Azure 虚拟机服务上支持与 SAP 软件结合使用的操作系统版本以及数据库系统和版本。
* SAP 说明 [1928533] 中阐述了 Azure 虚拟机服务上支持的 SAP 应用程序和版本。
* 对于 SAP 方案，仅支持将 64 位映像作为 Azure 中的来宾 VM 运行。 因此，仅支持 64 位 SAP 应用程序和数据库。

## <a name="microsoft-azure-virtual-machine-services"></a>Microsoft Azure 虚拟机服务
Microsoft Azure Platform 是在 Microsoft 数据中心内托管和运行的 Internet 规模的云服务平台。 该平台包括 Microsoft Azure 虚拟机服务（基础结构即服务，简称 IaaS）和丰富的平台即服务 (PaaS) 功能集。

Azure 平台减少了采购前沿技术和基础结构的需要。 它可以按需提供计算资源和存储用于托管、缩放及管理 Web 应用程序与连接的应用程序，因此简化了应用程序的维护和操作。 借助采用高可用性设计、可根据使用需求动态缩放并允许用户选择即用即付定价模式的平台，可以自动进行基础结构管理。

![Microsoft Azure 虚拟机服务的定位][planning-guide-figure-400]

通过 Azure 虚拟机服务，Microsoft 可让你将自定义服务器映像作为 IaaS 实例部署到 Azure（参阅图 4）。 Azure 中的虚拟机基于 Hyper-V 虚拟硬盘驱动器 (VHD) 并可以运行不同的操作系统（作为来宾 OS）。

从操作的角度讲，Azure 虚拟机服务提供的体验与本地部署的虚拟机类似。 但是，它的一个明显优势在于客户不需要采购和管理基础结构。 开发人员和管理员能够完全控制这些虚拟机中的操作系统映像。 管理员可以远程登录到这些虚拟机，以执行维护和故障排除任务以及软件部署任务。 在部署方面，唯一的限制在于 Azure VM 的规模和功能。 可能无法像在本地虚拟机上那样细致地配置这些大小。 可以选择代表下列配置组合的各种 VM 类型：

* vCPU 的数量
* 内存
* 可附加的 VHD 数量
* 网络和存储带宽

[此文 (Linux)][virtual-machines-sizes-linux] 和[此文 (Windows)][virtual-machines-sizes-windows] 中的表格内显示了所提供的各种不同虚拟机的大小和限制。

并非每个 Azure 区域都提供所有不同的 VM 系列（有关 Azure 区域，请参阅下一章）。 另请注意，并非所有 VM 或 VM 系列都已通过 SAP 认证。

> [!IMPORTANT]
> 使用 SAP NetWeaver 应用程序时，只支持 SAP 说明 [1928533] 中所列的一部分 VM 类型和配置。
>
>

### <a name="be80d1b9-a463-4845-bd35-f4cebdb5424a"></a>Azure 区域
虚拟机将部署到所谓的 *Azure 区域*。 Azure 区域可以是位置很近的一个或多个数据中心。 针对全球大多数地缘政治区域，Microsoft 至少有两个 Azure 区域。 例如，在欧洲，有一个“北欧”和一个“西欧”Azure 区域。 一个地缘政治区域中的两个 Azure 区域之间相隔足够的距离，以确保自然或技术灾难不会同时影响同一地缘政治区域中的两个 Azure 区域。 由于 Microsoft 在全球不同的地缘政治区域中稳定扩建新的 Azure 区域，因此这些区域的数目稳定增长，截至 2015 年 12 月已达 20 个 Azure 区域，并且已声明了更多的区域。 客户可以将 SAP 系统部署到所有区域，包括中国的两个 Azure 区域。 有关 Azure 区域的最新信息，请参阅以下网站：<https://azure.microsoft.com/regions/>

### <a name="8d8ad4b8-6093-4b91-ac36-ea56d80dbf77"></a>Microsoft Azure 虚拟机的概念
Microsoft Azure 提供基础结构即服务 (IaaS) 解决方案用于托管与本地虚拟化解决方案具有类似功能的虚拟机。 可以在 Azure 门户、PowerShell 或 CLI 中创建虚拟机，这些工具还提供部署和管理功能。

Azure 资源管理器可让你使用声明性模板预配应用程序。 在单个模板中，可以部署多个服务及其依赖项。 可在应用程序生命周期的每个阶段中使用相同模板重复部署应用程序。

有关使用资源管理器模板的详细信息，请参阅以下文档：

* [使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机](../../linux/create-ssh-secured-vm-from-template.md)
* [使用 Azure 资源管理器与 PowerShell 管理虚拟机][virtual-machines-deploy-rmtemplates-powershell]
* <https://azure.microsoft.com/documentation/templates/>

另一个引人注目的功能就是能够从虚拟机创建映像，这样，便可以准备某些存储库，并从中快速部署符合要求的虚拟机实例。

有关从虚拟机创建映像的详细信息，可参阅[此文 (Linux)][virtual-machines-linux-capture-image-resource-manager] 和[此文 (Windows)][virtual-machines-windows-capture-image-resource-manager]。

#### <a name="df49dc09-141b-4f34-a4a2-990913b30358"></a>容错域
容错域代表一个物理故障单元，它与数据中心包含的物理基础结构密切相关；物理刀片服务器或机架可被视为一个容错域，两者之间不存在直接的一对一映射。

在将多个虚拟机部署为 Microsoft Azure 虚拟机服务中某个 SAP 系统的一部分时，可以操控 Azure 结构控制器，以将应用程序部署到不同的容错域中，从而达到 Microsoft Azure SLA 的要求。 但是，如果在 Azure 缩放单元（数百个计算节点或存储节点和网络的集合）中分布容错域，则在一定的程度上，无法直接控制向特定容错域分配 VM 的操作。 若要指示 Azure 结构控制器跨不同容错域部署一组 VM，需要在部署时向 VM 分配一个 Azure 可用性集。 有关 Azure 可用性集的详细信息，请参阅本文档中的 [Azure 可用性集][planning-guide-3.2.3]。

#### <a name="fc1ac8b2-e54a-487c-8581-d3cc6625e560"></a>升级域
升级域代表一个逻辑单元，可帮助确定如何更新 SAP 系统中的某个 VM，即多个 VM 中运行的 SAP 实例构建的 VM。 在进行升级时，Microsoft Azure 会经历逐个更新这些升级域的整个过程。 通过在部署时跨不同的升级域分布 VM，可以在一定程度上防止 SAP 系统产生潜在的停机。 为了强制 Azure 跨不同的升级域分散部署 SAP 系统的 VM，需要在部署时为每个 VM 设置特定的属性。 与容错域类似，Azure 缩放单元已分割到多个升级域中。 若要指示 Azure 结构控制器跨不同升级域部署一组 VM，需要在部署时向 VM 分配一个 Azure 可用性集。 有关 Azure 可用性集的详细信息，请参阅下面的 [Azure 可用性集][planning-guide-3.2.3]一章。

#### <a name="18810088-f9be-4c97-958a-27996255c665"></a>Azure 可用性集
Azure 结构控制器跨不同的容错域和升级域分布一个 Azure 可用性集中的 Azure 虚拟机。 跨不同容错域和升级域进行分布的目的是防止在进行基础结构维护或者在一个容错域中发生故障时关闭 SAP 系统的所有 VM。 默认情况下，VM 并不属于某个可用性集。 VM 加入可用性集的操作是在部署时定义的，或者以后通过重新配置并重新部署 VM 来定义的。

若要了解 Azure 可用性集的概念以及可用性集与容错域和升级域的关联方式，请阅读[此文][virtual-machines-manage-availability]

若要通过 JSON 模板定义 Azure 资源管理器的可用性集，请参阅 [rest-api 规范](https://github.com/Azure/azure-rest-api-specs/blob/master/arm-compute/2015-06-15/swagger/compute.json)并搜索“availability”。

### <a name="a72afa26-4bf4-4a25-8cf7-855d6032157f"></a>存储：Microsoft Azure 存储和数据磁盘
Microsoft Azure 虚拟机使用不同的存储类型。 在 Azure 虚拟机服务上实施 SAP 时，必须了解以下两种主要存储类型之间的差异：

* 非持久性、易失性存储。
* 持久性存储。

部署 VM 后，Azure VM 提供非永久磁盘。 如果 VM 重启，这些驱动器上的所有内容将被擦除。因此，数据库的数据文件和日志/重做文件决不能放置在这些非永久性驱动器上。 但一些数据库例外，在此情况下，这些非永久性驱动器对于 tempdb 和临时表空间可能是适合的。 但是，应避免将这些驱动器用于 A 系列 VM，因为这些非永久性驱动器在该 VM 系列中的吞吐量有限。 有关进一步详细信息，请参阅[了解 Azure 中 Windows VM 上的临时驱动器](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)一文

- - -
> ![Windows][Logo_Windows] Windows
> 
> Azure VM 中的驱动器 D:\ 是一个非持久性驱动器，由 Azure 计算节点上的部分本地磁盘提供支持。 非持久性意味着，当 VM 重新启动时，将丢失对 D:\ 驱动器上的内容所做的任何更改。 “任何更改”是指已存储的文件、已创建的目录、已安装的应用程序等等。
> 
> ![Linux][Logo_Linux] Linux
> 
> Linux Azure VM 会在 /mnt/resource 上自动装载一个非持久性驱动器，该驱动器由 Azure 计算节点上的本地磁盘提供支持。 非持久性意味着，当 VM 重新启动时，将丢失对 /mnt/resource 中的内容所做的任何更改。 “任何更改”是指已存储的文件、已创建的目录、已安装的应用程序等等。
> 
> 

- - -

Microsoft Azure 存储提供持久性存储，以及 SAN 存储所具有的典型保护和冗余级别。 基于 Azure 存储的磁盘是 Azure 存储服务中的虚拟硬盘容器 (VHD)。 本地 OS 磁盘 (Windows C:\,Linux /dev/sda1) 存储在 Azure 存储中，装载到 VM 的其他卷/磁盘也存储在此处。

可以从本地上传现有的 VHD 容器，或者从 Azure 内部创建空的 VHD 容器，然后将其附加到部署的 VM。

在 Azure 存储中创建或上传 VHD 容器后，可将这些容器装载和附加到现有的虚拟机，以及复制现有的（未装载的）VHD 容器。

由于这些存储容器是持久性的，因此，在重新启动和重新创建虚拟机实例时，这些容器中的数据和更改会安全保留。 即使删除了某个实例或整个托管服务，这些 VHD 也会保持安全并可重新部署；另外，非 OS 磁盘还可装载到其他 VM。

有关 Azure 存储的详细信息，请参阅：

* <https://azure.microsoft.com/documentation/services/storage/>
* <https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>
* <https://blogs.msdn.com/b/azuresecurity/archive/2015/11/17/azure-disk-encryption-for-linux-and-windows-virtual-machines-public-preview.aspx>

#### <a name="azure-standard-storage"></a>Azure 标准存储
Azure 标准存储是 Azure IaaS 发布时可用的存储类型。 针对每个磁盘强制实施 IOPS 配额。 体验到的延迟并不如一般针对本地托管的高级 SAP 系统所部署的 SAN/NAS 设备好。 但是，Azure 标准存储证明对于部署在 Azure 中的数百个 SAP 系统便已足够。

存储于 Azure 标准存储帐户上的磁盘收费依据包括存储的实际数据、存储交易量、出站数据传输及所选的冗余选项。 许多磁盘可创建的大小上限为 1TB，但只要保持空白，就不产生任何费用。 如果以后对一个 VHD 每次填充 100GB，则会针对存储 100GB 而不是正常创建 VHD 的大小向你收费。

#### <a name="ff5ad0f9-f7f4-4022-9102-af07aef3bc92"></a>Azure 高级存储
推出 Azure 高级存储的目标是提供：

* 更短的 I/O 延迟。
* 更高吞吐量。
* 更小的 I/O 延迟变化性。

因此引入了许多更改，其中两项最重要的更改是：

* Azure 存储节点中的 SSD 磁盘使用方式
* Azure 计算节点的本地 SSD 支持的新读取缓存

相比于标准存储的容量不随磁盘（或 VHD）的大小而更改，高级存储目前有 3 种不同的磁盘类别，如本文所示：<https://azure.microsoft.com/pricing/details/storage/unmanaged-disks/>

如你所见，IOPS/磁盘和磁盘吞吐量/磁盘依赖于磁盘的大小类别

高级存储的成本基础不是存储在此类磁盘中的实际数据量，而是此类磁盘的大小类别，与存储在磁盘中的数据量无关。

还可在高级存储中创建不直接映射到所示大小类别的磁盘。 尤其是将磁盘从标准存储复制到高级存储时，可能发生此情况。 在此情况下，执行下一个最大高级存储磁盘选项的映射。

大多数已通过 SAP 认证的 Azure VM 系列能够使用高级存储，和/或 Azure 标准存储与高级存储的搭配形式。

如果查看[此文 (Linux)][virtual-machines-sizes-linux] 和[此文 (Windows)][virtual-machines-sizes-windows] 中有关 DS 系列 VM 的部分，就可以发现高级存储磁盘对 VM 级别粒度的数据量有所限制。 不同的 DS 系列或 GS 系列 VM 对可装载的 数据磁盘数也有不同的限制。 上述文章也阐述了这些限制。 但基本上，这意味着如果将 32 x P30 磁盘装载到单个 DS14 VM，则无法获得 32 x P30 磁盘的最大吞吐量。 如文中所述，VM 级别的最大吞吐量会限制数据吞吐量。

有关高级存储的详细信息，请参阅此处：<https://azure.microsoft.com/blog/2015/04/16/azure-premium-storage-now-generally-available-2>

#### <a name="azure-storage-accounts"></a>Azure 存储帐户

在 Azure 中部署服务或 VM 时，可在名为 Azure 存储帐户的容器中组织 VHD 和 VM 映像的部署。 规划 Azure 部署时，需要认真考虑 Azure 的限制。 另一方面，每个 Azure 订阅的存储帐户数也有限制。 尽管每个 Azure 存储帐户可以保留大量的 VHD 文件，但是，每个存储帐户的总 IOPS 却存在固定的限制。 在部署数百个 SAP VM、并且这些 VM 中的 DBMS 系统会产生大量的 IO 调用时，建议将 IOPS 较高的 DBMS VM 分散在多个 Azure 存储帐户之间。 必须小心不要超出每个订阅的当前 Azure 存储帐户数限制。 由于存储是 SAP 系统的数据库部署的关键部分，前面提到的 [DBMS Deployment Guide][dbms-guide]（DBMS 部署指南）中详细介绍了有关存储的概念。

可在[此文][storage-scalability-targets]中找到有关 Azure 存储帐户的详细信息。 阅读此文后，即可了解 Azure 标准存储帐户与高级存储帐户之间的限制差异。 主要差异是可存储在存储帐户中的数据量。 标准存储中的数据量大于高级存储中的数据量。 另一方面，标准存储帐户的 IOPS 限制很严格（请参阅“总请求率”列），而 Azure 高级存储帐户没有此限制。 谈到 SAP 系统（特别是 DBMS 服务器）的部署时，我们将探讨这些差异的详细信息和结果。

在存储帐户中，可以创建不同的容器来组织和分类不同的 VHD。 例如，这些容器用于隔离不同 VM 的 VHD。 在单个 Azure 存储帐户下只使用一个容器或使用多个容器对性能没有太大的影响。

在 Azure 中，VHD 名称遵循以下命名约定，需要为 Azure 中的 VHD 提供唯一的名称：

    http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>

此字符串必须唯一标识存储在 Azure 存储中的 VHD。

#### <a name="c55b2c6e-3ca1-4476-be16-16c81927550f"></a>托管磁盘

托管磁盘是 Azure 资源管理器中的一种新资源类型，可用于替代 Azure 存储帐户中存储的 VHD。 托管磁盘与其所附加到的虚拟机可用性集自动对齐，因而增加了虚拟机和虚拟机上所运行服务的可用性。 有关详细信息，请阅读[概述文章](https://docs.microsoft.com/azure/storage/storage-managed-disks-overview)。

建议使用托管磁盘，因为托管磁盘可简化虚拟机的部署和管理。
SAP 当前仅支持高级托管磁盘。 有关详细信息，请参阅 SAP 说明 [1928533]。

#### <a name="microsoft-azure-storage-resiliency"></a>Microsoft Azure 存储复原

Microsoft Azure 存储将基础 VHD（含 OS）以及附加磁盘或 BLOB 存储到至少三个不同的存储节点。 这称为本地冗余存储 (LRS)。 LRS 是 Azure 中所有存储类型的默认设置。 

此外，还有一些其他冗余方法，请参阅 [Azure 存储复制](https://docs.microsoft.com/azure/storage/common/storage-redundancy?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)一文。

> [!NOTE]
>自 Azure 高级存储（这是针对 DBMS VM 和存储数据库和日志/重做文件的磁盘的建议存储类型）起，LRS 是唯一可用的方法。 因此，需要配置数据库方法（例如 SQL Server Always On、Oracle Data Guard 或 HANA 系统复制），将数据库数据复制到其他 Azure 区域或其他 Azure 可用性区域。


> [!NOTE]
> 对于 DBMS 部署，不建议使用 Azure 标准存储的异地冗余存储，因为它会严重影响性能，且不遵循附加到 VM 的不同 VHD 中的写入顺序。 由于不遵循跨不同 VHD 的写入顺序，因此如果数据库和日志/重做文件分布在源 VM 端的不同 VHD 中（大多数情况是这样），则极有可能在复制目标端的不一致数据库中结束。


### <a name="61678387-8868-435d-9f8c-450b2424f5bd"></a>Microsoft Azure 网络

Microsoft Azure 提供一种网络基础结构，允许映射想要使用 SAP 软件实现的所有方案。 功能包括：

* 通过 Windows 终端服务或 ssh/VNC 从外部直接访问 VM
* 访问 VM 中的应用程序使用的服务和特定端口
* 在部署为 Azure VM 的一组 VM 之间进行内部通信和名称解析
* 在客户的本地网络与 Azure 网络之间建立跨界连接
* 在 Azure 站点之间建立跨 Azure 区域或数据中心的连接

可在此处 <https://azure.microsoft.com/documentation/services/virtual-network/> 找到更多信息

可以通过许多不同的方式配置 Azure 中的名称解析和 IP 解析。 此外，还可使用 Azure DNS 服务，而不是设置自己的 DNS 服务器。 有关详细信息，请参阅[此文][virtual-networks-manage-dns-in-vnet]的[此页](https://azure.microsoft.com/services/dns/)。

对于跨界或混合方案，我们将依赖于本地 AD/OpenLDAP/DNS 已通过 VPN 或专用连接扩展到 Azure 这一事实。 对于此处所述的某些方案，可能需要在 Azure 中安装 AD/OpenLDAP 副本。

由于网络和名称解析是 SAP 系统的数据库部署的关键部分，[DBMS Deployment Guide][dbms-guide]（DBMS 部署指南）中详细介绍了与此相关的概念。

##### <a name="azure-virtual-networks"></a>Azure 虚拟网络

通过构建 Azure 虚拟网络，可以定义 Azure DHCP 功能分配的专用 IP 地址范围。 在跨界方案中，Azure 仍以 DHCP 方式分配定义的 IP 地址范围。 但是，域名解析在本地运行（假设 VM 是本地域的一部分），因此可解析不同 Azure 云服务以外的地址。

Azure 中的每个虚拟机都需要连接到虚拟网络。

有关详细信息，请参阅[此文][resource-groups-networking]的[此页](https://azure.microsoft.com/documentation/services/virtual-network/)。


> [!NOTE]
> 默认情况下，部署 VM 后，无法更改虚拟网络配置。 TCP/IP 设置必须留给 Azure DHCP 服务器。 默认行为是动态 IP 配置。
>
>

虚拟网络卡的 MAC 地址可能会发生更改，例如，在调整大小后，Windows 或 Linux 来宾 OS 会选择新的网卡，并自动使用 DHCP 来分配 IP 和 DNS 地址。

##### <a name="static-ip-assignment"></a>静态 IP 分配
可以将固定或保留的 IP 地址分配给 Azure 虚拟网络中的 VM。 在 Azure 虚拟网络中运行 VM 可以有更多机会在需要进行某些方案时使用此功能。 IP 配置在整个 VM 存在期间保持有效，而无论 VM 是在运行还是已关机。 因此，当为虚拟网络定义 IP 地址范围时，必须考虑 VM（正在运行和已停止的 VM）总数。 在删除 VM 及其网络接口之前，或者在 IP 地址再次取消分配之前，保持分配此 IP 地址。 有关详细信息，请阅读[此文][virtual-networks-static-private-ip-arm-pportal]。

> [!NOTE]
> 应该通过 Azure 方式将静态 IP 地址分配给单个 vNIC。 不应将来宾 OS 中的静态 IP 地址分配给 vNIC。 某些 Azure 服务（例如 Azure 备份服务）依赖于至少主 vNIC 设置为 DHCP 而不是静态 IP 地址这一事实。 另请参阅文档[排查 Azure 虚拟机备份问题](https://docs.microsoft.com/azure/backup/backup-azure-vms-troubleshoot#networking)。
>
>

##### <a name="multiple-nics-per-vm"></a>每个 VM 可以有多个 NIC

可以为一个 Azure 虚拟机定义多个虚拟网络接口卡 (vNIC)。 由于可拥有多个 vNIC，因此可以开始设置网络流量分隔，例如通过一个 vNIC 路由客户端流量，并通过第二个 vNIC 路由后端流量。 不同的 VM 类型有不同的 vNIC 数目限制。 有关确切详细信息、功能和限制，请参阅以下文章：

* [创建具有多个 NIC 的 Windows VM][virtual-networks-multiple-nics-windows]
* [创建具有多个 NIC 的 Linux VM][virtual-networks-multiple-nics-linux]
* [使用模板部署多 NIC VM][virtual-network-deploy-multinic-arm-template]
* [使用 PowerShell 部署多 NIC VM][virtual-network-deploy-multinic-arm-ps]
* [使用 Azure CLI 部署多 NIC VM][virtual-network-deploy-multinic-arm-cli]

#### <a name="site-to-site-connectivity"></a>站点到站点连接

跨界方案使用透明的 VPN 永久性连接将 Azure VM 和本地 VM 链接在一起。 预期它将成为 Azure 中最常见的 SAP 部署模式。 其假设条件是 Azure 中 SAP 实例的操作过程和流程以透明方式进行。 这意味着，应该能够输出这些系统的信息，并使用 SAP 传输管理系统 (TMS) 将更改从 Azure 中的开发系统传输到本地部署的测试系统。 可在[此文][vpn-gateway-create-site-to-site-rm-powershell]中找到有关站点到站点连接的更多文档

##### <a name="vpn-tunnel-device"></a>VPN 隧道设备

为了创建站点到站点连接（本地数据中心到 Azure 数据中心的连接），需要获取并配置 VPN 设备，或使用 Windows Server 2012 中以软件组件形式引入的路由和远程访问服务 (RRAS)。

* [使用 PowerShell 创建具有站点到站点 VPN 连接的虚拟网络][vpn-gateway-create-site-to-site-rm-powershell]
* [关于站点到站点 VPN 网关连接的 VPN 设备][vpn-gateway-about-vpn-devices]
* [VPN 网关常见问题][vpn-gateway-vpn-faq]

![本地与 Azure 之间的站点到站点连接][planning-guide-figure-600]

上图显示了两个 Azure 订阅，其 IP 地址子范围已保留给 Azure 中的虚拟网络使用。 从本地网络到 Azure 的连接是通过 VPN 建立的。

#### <a name="point-to-site-vpn"></a>点到站点 VPN

点到站点 VPN 要求每台客户端计算机使用自身的 VPN 连接到 Azure。 对于我们目前正在探讨的 SAP 方案，点到站点连接是不可行的。 因此，不提供有关点到站点 VPN 连接的更多参考。

可在此处找到更多信息
* [使用 Azure 门户配置与 VNet 的点到站点连接](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal)
* [使用 PowerShell 配置与 VNet 的点到站点连接](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps)

#### <a name="multi-site-vpn"></a>多站点 VPN

Azure 现在也可让为一个 Azure 订阅创建多站点 VPN 连接。 以前，单个订阅仅限于一个站点到站点 VPN 连接。 此限制在单个订阅可以有多站点 VPN 连接之后已不存在。 可以通过跨界配置，针对特定订阅使用多个 Azure 区域。

有关更多文档，请参阅[此文][vpn-gateway-create-site-to-site-rm-powershell]

#### <a name="vnet-to-vnet-connection"></a>VNet 到 VNet 连接

使用多站点 VPN 时，需要在每个区域中配置不同的 Azure 虚拟网络。 但是，通常必须让不同区域中的软件组件能够彼此通信。 在理想情况下，此通信应该从一个 Azure 区域路由到本地，再由此路由到其他 Azure 区域。 简而言之，Azure 可让设置从某个区域中的一个 Azure 虚拟网络到托管在另一个区域中的另一个 Azure 虚拟网络的连接。 此功能称为 VNet 对 VNet 连接。 可在此处找到关于此功能的更多详细信息：<https://azure.microsoft.com/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/>。

#### <a name="private-connection-to-azure-expressroute"></a>与 Azure ExpressRoute 建立专用连接

使用 Microsoft Azure ExpressRoute 可在 Azure 数据中心与客户本地环境或共置环境中的基础结构之间建立专用连接。 ExpressRoute 由不同的 MPLS（数据包交换）VPN 提供商或其他网络服务提供商提供。 ExpressRoute 连接不通过公共 Internet 。 与通过 Internet 建立的典型连接相比，ExpressRoute 连接可通过多个并行线路提供更高的安全性、更高的可靠性、更快的速度和更低的延迟。

有关 Azure ExpressRoute 和产品的详细信息，请参阅：

* <https://azure.microsoft.com/documentation/services/expressroute/>
* <https://azure.microsoft.com/pricing/details/expressroute/>
* <https://azure.microsoft.com/documentation/articles/expressroute-faqs/>

如文中所述，使用 Express Route 可通过一条 ExpressRoute 线路启用多个 Azure 订阅

* <https://azure.microsoft.com/documentation/articles/expressroute-howto-linkvnet-arm/>
* <https://azure.microsoft.com/documentation/articles/expressroute-howto-circuit-arm/>

#### <a name="forced-tunneling-in-case-of-cross-premises"></a>跨界方案中的强制隧道
要使 VM 通过站点到站点连接、点到站点连接或 ExpressRoute 加入到本地域，还需要确保为这些 VM 中的所有用户部署 Internet 代理设置。 默认情况下，这些 VM 中运行的软件或者使用浏览器访问 Internet 的用户不会通过公司代理，而是直接通过 Azure 连接到 Internet。 但是，即使进行了代理设置，也不能完全保证能够通过公司代理定向流量，因为检查代理是软件和服务的责任。 如果 VM 中运行的软件不执行这种检查，或者由管理员处理这些设置，则定向到 Internet 的流量可能会再次直接通过 Azure 改道发往 Internet。

若要避免此类直接 Internet 连接，可以在本地与 Azure 之间配置具有站点到站点连接的强制隧道。 <https://azure.microsoft.com/documentation/articles/vpn-gateway-forced-tunneling-rm/> 上发布了关于强制隧道功能的详细说明

ExpressRoute 强制隧道由通过 ExpressRoute BGP 对等互连会话广告默认路由的客户启用。

#### <a name="summary-of-azure-networking"></a>Azure 网络摘要

本章包含有关 Azure 网络的许多要点。 下面是要点摘要：

* Azure 虚拟网络允许将网络结构放入 Azure 部署。 VNet 可以相互独立，也可以借助 VNet 之间的网络安全组流量对其进行控制。
* 可以利用 Azure 虚拟专用网络向 VM 分配 IP 地址范围，或者向 VM 分配固定 IP 地址
* 若要设置站点到站点或点到站点连接，首先需要创建 Azure 虚拟网络
* 部署某个虚拟机后，不再能够更改分配给该 VM 的虚拟网络

### <a name="quotas-in-azure-virtual-machine-services"></a>Azure 虚拟机服务中的配额
我们需要明确这个事实：存储和网络基础结构是在运行 Azure 基础结构中的各种服务的 VM 之间共享的。 就如同在客户自有的数据中心内一样，过度预配某些基础结构资源的情况确实时有发生。 Microsoft Azure Platform 使用磁盘、CPU、网络和其他配额来限制资源消耗，并维持一致性、确定性的性能。  不同的 VM 类型（A5、A6 等）针对磁盘、CPU、RAM 和网络资源的数量采用不同的配额。

> [!NOTE]
> SAP 支持的 VM 类型的 CPU 和内存资源是在主机节点上预先分配的。 这意味着，部署 VM 后，该主机会根据 VM 类型的定义提供资源。
>
>

在 Azure 解决方案中进行 SAP 规划和选型时，必须考虑适用于每个虚拟机大小的配额。 [此文 (Linux)][virtual-machines-sizes-linux] 和[此文 (Windows)][virtual-machines-sizes-windows] 介绍了 VM 配额。

所述的配额代表理论最大值。  使用较小的 IO (8kb) 也许能够满足每个磁盘的 IOPS 限制，但是，使用较大的 IO (1Mb) 可能无法满足该限制。  IOPS 限制是针对单个磁盘的粒度强制实施的。

可以使用以下粗糙决策树来确定某个 SAP 系统是否能够适应 Azure 虚拟机服务及其功能，或者是否需要以不同的方式配置某个现有系统，以将其部署在 Azure 上：

![用于确定能否在 Azure 上部署 SAP 的决策树][planning-guide-figure-700]

**步骤 1**：首先要获取的最重要信息是有关给定 SAP 系统的 SAPS 要求。 需要将 SAPS 要求划分为 DBMS 部分和 SAP 应用程序部分，即使该 SAP 系统已部署在本地的双层配置中。 对于现有系统，通常可以根据现有的 SAP 基准来确定或估测使用中的硬件的相关 SAPS。 可在此处 <https://sap.com/about/benchmark.html> 找到结果。
对于新部署的 SAP 系统，应该事先完成整个大小调整活动，该活动应得出系统的 SAPS 要求。
有关在 Azure 上调整 SAP 大小的信息，另请参阅此博客及附加文档：<https://blogs.msdn.com/b/saponsqlserver/archive/2015/12/01/new-white-paper-on-sizing-sap-solutions-on-azure-public-cloud.aspx>

**步骤 2**：对于现有系统，应该度量 DBMS 服务器上的每秒 I/O 量和 I/O 运算次数。 对于新规划的系统，在针对新的系统完成选型活动后，应该也能给出 DBMS 端 I/O 要求的粗略观点。 如果对这种结果没有把握，最终需要开展概念认证。

**步骤 3**：将 DBMS 服务器的 SAPS 要求与 Azure 的不同 VM 类型可以提供的 SAPS 进行比较。 SAP 说明 [1928533] 中阐述了有关不同 Azure VM 类型的 SAPS 的信息。 首先应该将注意力集中在 DBMS VM 上，因为数据库层是 SAP NetWeaver 系统上的、不能在大多数部署中横向扩展的层。 相比之下，SAP 应用程序层可以横向扩展。如果 SAP 支持的 Azure VM 类型都不能提供所需的 SAPS，则规划的 SAP 系统的工作负荷不能在 Azure 上运行。 在此情况下，需要将系统部署在本地，或者更改系统的工作负荷量。

**步骤 4**：如[此文 (Linux)][virtual-machines-sizes-linux] 和[此文 (Windows)][virtual-machines-sizes-windows] 所述，Azure 针对每个磁盘强制实施 IOPS 配额，而不管使用的是标准存储还是高级存储。 可装载的数据磁盘数量根据 VM 类型的不同而异。 因此，可以计算出使用每个不同 VM 类型能够实现的最大 IOPS 数。 根据数据库文件布局，可将磁盘条带化，使之成为来宾 OS 中的一个卷。 但是，如果所部署 SAP 系统的当前 IOPS 量即将超出最大 Azure VM 类型的计算得出的限制，并且无法使用更多的内存来予以补偿，那么，SAP 系统的工作负荷可能会受到严重的影响。 在这种情况下，可以认定不应将该系统部署在 Azure 上。

**步骤 5**：很有可能需要将系统配置在 Azure 上的 3 层配置中，尤其是在部署于本地双层配置中的 SAP 系统中。 在此步骤中，需要检查 SAP 应用程序层的某个组件是否无法横向扩展，并且无法适应不同 Azure VM 类型提供的 CPU 和内存资源。 如果确实存在这样一个组件，则不能将 SAP 系统及其工作负荷部署到 Azure 中。 但是，如果可以将 SAP 应用程序组件横向扩展到多个 Azure VM 中，则可以将该系统部署到 Azure 中。

**步骤 6**：如果 DBMS 和 SAP 应用程序层组件可以在 Azure VM 中运行，则需要定义有关以下各项的配置：

* Azure VM 的数量
* 各个组件的 VM 类型
* 为了提供足够的 IOPS 而需要在 DBMS VM 中配置的 VHD 数量

## <a name="managing-azure-assets"></a>管理 Azure 资产

### <a name="azure-portal"></a>Azure 门户

Azure 门户是用于管理 Azure VM 部署的三个界面之一。 基本的管理任务（例如，从映像部署 VM）可以通过该 Azure 门户来完成。 此外，Azure 门户也能很好地处理创建存储帐户、虚拟网络和其他 Azure 组件这样的任务。 但是，若要执行将 VHD 从本地上传到 Azure 或者复制 Azure 中的 VHD 等任务，需使用第三方工具，或者通过 PowerShell 或 CLI 进行管理。

![Microsoft Azure 门户 - 虚拟机概览][planning-guide-figure-800]


可以从 Azure 门户内部完成虚拟机实例的管理和配置任务。

除了重启和关闭虚拟机以外，还可以为虚拟机实例附加、分离和创建数据磁盘，以捕获实例进行映像准备并配置虚拟机实例的大小。

Azure 门户提供了基本的功能，用于部署和配置 VM 以及其他许多 Azure 服务。 但是，Azure 门户并未包括所有可用的功能。 在 Azure 门户中，无法执行如下所述的任务：

* 将 VHD 上传到 Azure
* 复制 VM


### <a name="management-via-microsoft-azure-powershell-cmdlets"></a>通过 Microsoft Azure PowerShell cmdlet 进行管理

Windows PowerShell 是一个功能强大的可扩展框架，在 Azure 中部署大量系统的客户已广泛使用了该框架。 在台式机、便携式计算机或专用管理工作站上使用证书安装 PowerShell cmdlet 后，可以远程运行 PowerShell cmdlet。

[此文][powershell-install-configure]介绍了在本地台式机/便携式计算机上启用 Azure PowerShell cmdlet 的过程，以及有关如何配置这些 cmdlet 以配合 Azure 订阅使用的信息。

有关如何安装、更新和配置 Azure PowerShell cmdlet 的详细步骤，还可参阅[部署指南的此章][deployment-guide-4.1]。

根据到目前为止的客户体验，PowerShell (PS) 绝对是用于部署 VM 以及在 VM 部署中创建自定义步骤的更强大工具。 在 Azure 中运行 SAP 实例的所有客户均使用 PS cmdlet 作为补充性手段来执行其在 Azure 门户中所能执行的管理任务，甚至专门使用 PS cmdlet 来管理 Azure 中的部署。 由于 Azure 专用的 cmdlet 与 2000 多个 Windows 相关 cmdlet 采用相同的命名约定，因此，Windows 管理员可以轻而易举地使用这些 cmdlet。

请查阅此处的示例：<https://blogs.technet.com/b/keithmayer/archive/2015/07/07/18-steps-for-end-to-end-iaas-provisioning-in-the-cloud-with-azure-resource-manager-arm-powershell-and-desired-state-configuration-dsc.aspx>


部署适用于 SAP 的 Azure 监视扩展（请参阅本文档中的[适用于 SAP 的 Azure 监视解决方案][planning-guide-9.1]一章）只能通过 PowerShell 或 CLI 完成。 因此，在 Azure 中部署或管理 SAP NetWeaver 系统时，必须安装并配置 PowerShell 或 CLI。  

随着 Azure 提供的功能越来越多，我们会添加新的 PS cmdlet，这就需要你更新 cmdlet。 因此，最好是每月至少查看 Azure 下载站点 <https://azure.microsoft.com/downloads/> 一次，获取新版 cmdlet。 安装新版本时会覆盖旧版本。

请查看此处 <https://docs.microsoft.com/powershell/azure/overview>，获取 Azure 相关的 PowerShell 命令的常规列表。

### <a name="management-via-microsoft-azure-cli-commands"></a>通过 Microsoft Azure CLI 命令进行管理

对于使用 Linux 并想要管理 Azure 资源的客户而言，可能不适合使用 Powershell。 Microsoft 提供了 Azure CLI 作为替代方案。
Azure CLI 提供了一组开源且跨平台的命令，这些命令可以用于 Azure 平台。 Azure CLI 提供许多与 Azure 门户相同的功能。

有关安装、配置和如何使用 CLI 命令完成 Azure 任务的信息，请参阅

* [安装 Azure 经典 CLI][xplat-cli]
* [使用 Azure 资源管理器模板与 Azure CLI 部署和管理虚拟机][../../linux/create-ssh-secured-vm-from-template.md]
* [将适用于 Mac、Linux 和 Windows 的 Azure 经典 CLI 与 Azure 资源管理器配合使用][xplat-cli-azure-resource-manager]

有关如何使用 Azure CLI 部署适用于 SAP 的 Azure 监视扩展，另请参阅[部署指南][planning-guide]中的[适用于 Linux VM 的 Azure CLI][deployment-guide-4.5.2] 一章。

## <a name="different-ways-to-deploy-vms-for-sap-in-azure"></a>在 Azure 中为 SAP 部署 VM 的不同方式

本章介绍在 Azure 中部署 VM 的不同方式。 本章还会介绍其他准备过程，以及如何在 Azure 中处理 VHD 和 VM。

### <a name="deployment-of-vms-for-sap"></a>为 SAP 部署 VM

Microsoft Azure 提供多种用于部署 VM 和相关磁盘的方法。 因此，请务必了解这些方法之间的差异，因为 VM 的准备工作根据部署方法的不同而异。 总体而言，我们将探讨以下方案：

#### <a name="4d175f1b-7353-4137-9d2f-817683c26e53"></a>使用非通用化磁盘将虚拟机从本地移至 Azure

打算将某个特定 SAP 系统从本地移至 Azure。 通过将包含 OS、SAP 二进制文件和 DBMS 二进制文件的 VHD，以及包含 DBMS 数据和日志文件的 VHD 上传到 Azure，可以实现此目的。 与[下面的方案 2][planning-guide-5.1.2] 相反，需要将 Azure VM 中的主机名、SAP SID 和 SAP 用户帐户保留为与本地环境中的配置相同。 因此，不需要将映像通用化。 有关本地准备步骤以及向 Azure 上传非通用化 VM 或 VHD 的信息，请参阅本文档的[准备使用非通用化磁盘将 VM 从本地移到 Azure][planning-guide-5.2.1] 一章。 请参阅[方案 3：使用非通用化 Azure VHD 和 SAP 从本地移动 VM][deployment-guide-3.4] 一章（[部署指南][deployment-guide]），了解有关在 Azure 中部署此类映像的详细步骤。

#### <a name="e18f7839-c0e2-4385-b1e6-4538453a285c"></a>使用特定于客户的映像部署虚拟机

由于 OS 或 DBMS 版本存在特定的修补程序要求，Azure 市场中提供的映像可能并不符合需要。 因此，可能需要使用自己的、以后可以多次部署的“专用”OS/DBMS VM 映像创建一个 VM。 若要准备这样一个可供复制的专用映像，必须考虑以下要素：

- - -
> ![Windows][Logo_Windows] Windows
>
> 有关详细信息，请参阅：<https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed>Windows 设置（例如 Windows SID 和主机名）必须通过 sysprep 命令在本地 VM 抽象化/通用化。
>
>
> ![Linux][Logo_Linux] Linux
>
> 请遵循这些 [SUSE][virtual-machines-linux-create-upload-vhd-suse]、[Red Hat][virtual-machines-linux-redhat-create-upload-vhd] 或 [Oracle Linux][virtual-machines-linux-create-upload-vhd-oracle] 文章中所述的步骤准备要上传到 Azure 的 VHD。
>
>

- - -
如果已将 SAP 内容安装在本地 VM 中（尤其是对于双层系统），则可以在部署 Azure VM 之后，通过 SAP Software Provisioning Manager 支持的实例重命名过程来修改 SAP 系统设置（SAP 说明 [1619720]）。 有关本地准备步骤以及如何将通用 VM 上传到 Azure 的信息，请参阅本文档的章节[准备使用特定于客户的映像为 SAP 部署 VM][planning-guide-5.2.2] 和[将 VHD 从本地上传到 Azure][planning-guide-5.3.2]。 请参阅[方案 2：使用自定义映像为 SAP 部署 VM][deployment-guide-3.3] 一章（[部署指南][deployment-guide]），了解有关在 Azure 中部署此类映像的详细步骤。

#### <a name="deploying-a-vm-out-of-the-azure-marketplace"></a>部署从 Azure 市场中取出的 VM

想要从 Azure 市场获取某个 Microsoft 或第三方 VM 映像来部署 VM。 将该 VM 部署到 Azure 中后，需要像在本地环境中一样，遵照相同的指导原则并使用相同的工具在 VM 内部安装 SAP 软件和/或 DBMS。 有关更详细的部署说明，请参阅[方案 1：为 SAP 部署来自 Azure 市场的 VM][deployment-guide-3.2]（[部署指南][deployment-guide]）。

### <a name="6ffb9f41-a292-40bf-9e70-8204448559e7"></a>为 Azure 准备包含 SAP 的虚拟机

将 VM 上传到 Azure 之前，需要确保这些 VM 和 VHD 符合特定的要求。 根据所用的部署方法，准备工作将有细微的差异。

#### <a name="1b287330-944b-495d-9ea7-94b83aff73ef"></a>准备使用非通用化磁盘将虚拟机从本地移到 Azure

一种常用的部署方法是将运行 SAP 系统的现有 VM 从本地移到 Azure。 该 VM 以及其中包含的 SAP 系统只应使用相同的主机名、并且很有可能使用相同的 SAP SID 在 Azure 中运行。 在此情况下，VM 的来宾 OS 不应该针对多个部署通用化。 如果本地网络已扩展到 Azure 中（请参阅本文档中的[跨界 - 将单个或多个 SAP VM 部署到 Azure 中并要求完全集成到本地网络][planning-guide-2.2]一章），则在该 VM 中使用的域帐户甚至都有可能与以前在本地使用的域帐户相同。

准备自己的 Azure VM 磁盘时要满足的要求如下：

* 包含操作系统的 VHD 的最大大小最初只能是 127GB。 此限制已在 2015 年 3 月底消除。 现在，包含操作系统的 VHD 可以有高达 1TB 的大小，Azure 存储托管任何其他 VHD 也是一样。
* 它需要采用固定的 VHD 格式。 Azure 尚不支持动态 VHD 或采用 VHDx 格式的 VHD。 当你使用 PowerShell cmdlet 或 CLI 上传 VHD 时，动态 VHD 将转换为静态 VHD
* 已装载到 VM、并且应该在 Azure 中再次装载到 VM 的 VHD 也需要采用固定的 VHD 格式。 有关数据磁盘的大小限制，请参阅[这篇文章 (Linux)](../../linux/managed-disks-overview.md) 和[这篇文章 (Windows)](../../windows/managed-disks-overview.md)。 当你使用 PowerShell cmdlet 或 CLI 上传 VHD 时，动态 VHD 将转换为静态 VHD
* 使用管理员特权添加另一个本地帐户，该帐户可供 Microsoft 支持人员使用，或者在部署 VM 之前分配为上下文以供服务和应用程序短暂访问，并可供其他适当的用户使用。
* 添加其他本地帐户，因为特定的部署方案可能需要这些帐户。

- - -
> ![Windows][Logo_Windows] Windows
>
> 在此方案中，无需通用化 (sysprep) VM 就能在 Azure 中上传和部署该 VM。
> 确保驱动器 D:\ 未被使用。
> 按照本文档的[为附加的磁盘设置自动装载][planning-guide-5.5.3]一章中所述为附加磁盘设置磁盘自动装载。
>
> ![Linux][Logo_Linux] Linux
>
> 在此方案中，无需通用化 ( waagent -deprovision) VM 就能在 Azure 中上传和部署该 VM。
> 请确保 /mnt/resource 未被使用，并已通过 uuid 装载所有磁盘。 对于 OS 磁盘，请确保引导加载程序条目也反映了基于 uuid 的装载。
>
>

- - -
#### <a name="57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3"></a>准备使用特定于客户的映像为 SAP 部署虚拟机

包含通用化 OS 的 VHD 文件存储在 Azure 存储帐户的容器中，或存储为托管磁盘映像。 可通过将部署模板文件中的 VHD 或托管磁盘映像引用为源，从此类映像部署新的 VM，如[方案 2：使用自定义映像为 SAP 部署 VM][deployment-guide-3.3] 一章（[部署指南][deployment-guide]）中所述。

准备自己的 Azure VM 映像时要满足的要求如下：

* 包含操作系统的 VHD 的最大大小最初只能是 127GB。 此限制已在 2015 年 3 月底消除。 现在，包含操作系统的 VHD 可以有高达 1TB 的大小，Azure 存储托管任何其他 VHD 也是一样。
* 它需要采用固定的 VHD 格式。 Azure 尚不支持动态 VHD 或采用 VHDx 格式的 VHD。 当你使用 PowerShell cmdlet 或 CLI 上传 VHD 时，动态 VHD 将转换为静态 VHD
* 已装载到 VM、并且应该在 Azure 中再次装载到 VM 的 VHD 也需要采用固定的 VHD 格式。 有关数据磁盘的大小限制，请参阅[此文 (Linux)](../../windows/managed-disks-overview.md) 和[此文 (Windows)](../../linux/managed-disks-overview.md)。 当你使用 PowerShell cmdlet 或 CLI 上传 VHD 时，动态 VHD 将转换为静态 VHD
* 添加其他本地帐户，因为特定的部署方案可能需要这些帐户。
* 如果映像包含 SAP NetWeaver 的安装，并且可能在部署 Azure 时重命名主机名的原始名称，则建议将最新版 SAP Software Provisioning Manager DVD 复制到模板。 这样，便可以轻松地使用 SAP 提供的重命名功能来修改已更改的主机名，和/或在启动新副本之后，更改已部署 VM 映像中 SAP 系统的 SID。

- - -
> ![Windows][Logo_Windows] Windows
>
> 确保驱动器 D:\ 未被使用。按照本文档的[为附加的磁盘设置自动装载][planning-guide-5.5.3]一章中所述为附加的磁盘设置磁盘自动装载。
>
> ![Linux][Logo_Linux] Linux
>
> 请确保 /mnt/resource 未被使用，并已通过 uuid 装载所有磁盘。 对于 OS 磁盘，请确保引导加载程序条目也反映了基于 uuid 的装载。
>
>

- - -
* SAP GUI（用于管理和设置目的）可能已预先安装在此类模板中。
* 可以在跨界方案中安装成功运行 VM 所需的其他软件，但前提是这些软件支持 VM 重命名。

如果 VM 已做好成为通用 VM 的充分准备，并且最终独立于目标 Azure 部署方案中未提供的帐户/用户，那么，便可以执行最后一个准备步骤，即通用化这种映像。

##### <a name="generalizing-a-vm"></a>通用化 VM
- - -
> ![Windows][Logo_Windows] Windows
>
> 最后一步是使用管理员帐户登录到 VM。 以管理员身份打开 Windows 命令窗口。 转到 %windir%\windows\system32\sysprep 并执行 sysprep.exe。
> 此时会出现一个小窗口中。 必须选中“通用化”选项（默认情况下未选中），并将“关机选项”从默认的“重新启动”更改为“关机”。 此过程假设 sysprep 进程在 VM 的来宾 OS 中本地运行。
> 如要针对已在 Azure 中运行的 VM 执行该过程，请遵循[此文](https://docs.microsoft.com/azure/virtual-machines/windows/capture-image-resource)中所述的步骤。
>
> ![Linux][Logo_Linux] Linux
>
> [如何捕获 Linux 虚拟机以用作 Resource Manager 模板][capture-image-linux-step-2-create-vm-image]
>
>

- - -
### <a name="transferring-vms-and-vhds-between-on-premises-to-azure"></a>在本地与 Azure 之间传输 VM 和 VHD
由于无法通过 Azure 门户将 VM 映像和磁盘上传到 Azure，因此需要使用 Azure PowerShell cmdlet 或 CLI。 另一种可行的方法是使用“AzCopy”工具。 该工具可在本地与 Azure 之间（双向）复制 VHD。 它还可以在 Azure 区域之间复制 VHD。 有关如何下载和使用 AzCopy，请参阅[此文档][storage-use-azcopy]。

第三种替代做法是使用面向 GUI 的各种第三方工具。 但是，请确保这些工具支持 Azure 页 Blob。 为了达到目的，需要使用 Azure 页 Blob 存储区（此处 <https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs> 描述了相关差异）。 另外，Azure 提供的工具能够非常高效地压缩需要上传的 VM 和 VHD。 这种优势非常重要，因为这种压缩效率可以减少上传时间（具体的上传时间根据从本地设施和目标 Azure 部署区域与 Internet 网络之间的上传链路的情况而有一定的差异）。 可以合理地推测出这样一个结论：将位于欧洲的某个 VM 或 VHD 上传到位于美国的 Azure 数据中心所需的时间，比将同一个 VM/VHD 上传到位于欧洲的 Azure 数据中心所需的时间要长。

#### <a name="a43e40e6-1acc-4633-9816-8f095d5a7b6a"></a>将 VHD 从本地上载到 Azure
若要从本地网络上传现有的 VM 或 VHD，这种 VM 或 VHD 需要满足本文档的[准备使用非通用化磁盘将 VM 从本地移到 Azure][planning-guide-5.2.1] 一章中列出的要求。

此类 VM 不需要通用化，在本地端将它关闭后，随时可以将它上传并保持它在关闭时所处的状态和形式。 对于不包含任何操作系统的其他 VHD，也同样如此。

##### <a name="uploading-a-vhd-and-making-it-an-azure-disk"></a>上传 VHD 并使其成为 Azure 磁盘
在本例中，我们将要上传一个包含或不包含 OS 的 VHD 并使其成为 Azure 中的磁盘，然后将它作为数据磁盘装载到 VM，或者将它用作 OS 磁盘。 此过程包含多个步骤

**Powershell**

* 登录到你的订阅与*Connect AzAccount*
* 设置订阅的上下文*集 AzContext*和参数 SubscriptionId 或 SubscriptionName-请参阅 <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>
* 上传 VHD*添加 AzVhd*到 Azure 存储帐户-请参阅 <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>
* （可选）从 VHD 创建托管磁盘*新建 AzDisk* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/new-Azdisk>
* 将新 VM 配置的 OS 磁盘设置为 VHD 或托管磁盘*集 AzVMOSDisk* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>
* 从 VM 配置创建新的 VM *New-azvm* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>
* 将数据磁盘添加到与新的 VM*添加 AzVMDataDisk* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/add-Azvmdatadisk>

**Azure CLI**

* 使用 *az login* 登录到订阅
* 使用 az account set --subscription `<subscription name or id`> 选择订阅
* 使用 az storage blob upload 上传 VHD - 请参阅 [结合使用 Azure CLI 与 Azure 存储][storage-azure-cli]
* （可选）使用 *az disk create* 从 VHD 创建托管磁盘 - 请参阅 https://docs.microsoft.com/cli/azure/disk
* 使用 az vm create 和参数 --attach-os-disk 创建新的 VM，并将上传的 VHD 或托管磁盘指定为 OS 磁盘
* 使用 az vm disk attach 和参数 --new 将数据磁盘添加到新 VM

**模板**

* 使用 Powershell 或 Azure CLI 上传 VHD
* （可选）使用 Powershell、Azure CLI 或 Azure 门户从 VHD 创建托管磁盘
* 使用引用 VHD 的 JSON 模板（如[此示例 JSON 模板](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json)中所示）或使用托管磁盘（如[此示例 JSON 模板](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json)中所示）部署 VM。

#### <a name="deployment-of-a-vm-image"></a>部署 VM 映像
若要从本地网络上传现有的 VM 或 VHD 以将其用作 Azure VM 映像，这种 VM 或 VHD 需要满足本文档的[准备使用特定于客户的映像为 SAP 部署 VM][planning-guide-5.2.2] 一章中列出的要求。

* 在 Windows 上使用sysprep  或者在 Linux 上使用 waagent -deprovision 将 VM 通用化 - 请参阅 [Sysprep Technical Reference](https://technet.microsoft.com/library/cc766049.aspx)（Sysprep 技术参考）（适用于 Windows）或 [如何捕获 Linux 虚拟机以用作 Resource Manager 模板][capture-image-linux-step-2-create-vm-image]（适用于 Linux）
* 登录到你的订阅与*Connect AzAccount*
* 设置订阅的上下文*集 AzContext*和参数 SubscriptionId 或 SubscriptionName-请参阅 <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>
* 上传 VHD*添加 AzVhd*到 Azure 存储帐户-请参阅 <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>
* （可选）从 VHD 创建托管磁盘映像*新建 AzImage* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/new-Azimage>
* 将新 VM 配置的 OS 磁盘设置为
  * 使用 VHD*集 AzVMOSDisk-SourceImageUri-CreateOption fromImage* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>
  * 托管磁盘映像*集 AzVMSourceImage* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmsourceimage>
* 从 VM 配置创建新的 VM *New-azvm* -请参阅 <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>

**Azure CLI**

* 在 Windows 上使用sysprep  或者在 Linux 上使用 waagent -deprovision 将 VM 通用化 - 请参阅 [Sysprep Technical Reference](https://technet.microsoft.com/library/cc766049.aspx)（Sysprep 技术参考）（适用于 Windows）或 [如何捕获 Linux 虚拟机以用作 Resource Manager 模板][capture-image-linux-step-2-create-vm-image]（适用于 Linux）
* 使用 *az login* 登录到订阅
* 使用 az account set --subscription `<subscription name or id`> 选择订阅
* 使用 az storage blob upload 上传 VHD - 请参阅 [结合使用 Azure CLI 与 Azure 存储][storage-azure-cli]
* （可选）使用 *az image create* 从 VHD 创建托管磁盘映像 - 请参阅 https://docs.microsoft.com/cli/azure/image
* 使用 az vm create 和参数 --image 创建新的 VM，并将上传的 VHD 或托管磁盘映像指定为 OS 磁盘

**模板**

* 在 Windows 上使用sysprep  或者在 Linux 上使用 waagent -deprovision 将 VM 通用化 - 请参阅 [Sysprep Technical Reference](https://technet.microsoft.com/library/cc766049.aspx)（Sysprep 技术参考）（适用于 Windows）或 [如何捕获 Linux 虚拟机以用作 Resource Manager 模板][capture-image-linux-step-2-create-vm-image]（适用于 Linux）
* 使用 Powershell 或 Azure CLI 上传 VHD
* （可选）使用 Powershell、Azure CLI 或 Azure 门户从 VHD 创建托管磁盘映像
* 使用引用映像 VHD 的 JSON 模板（如[此示例 JSON 模板](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json)中所示）或使用托管磁盘映像（如[此示例 JSON 模板](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json)中所示）部署 VM。

#### <a name="downloading-vhds-or-managed-disks-to-on-premises"></a>将 VHD 或托管磁盘下载到本地
Azure 基础结构即服务不仅能够上传 VHD 和 SAP 系统， 而且你还可以在其中将 SAP 系统从 Azure 移回到本地。

下载期间，VHD 或托管磁盘不能为活动状态。 即使在下载已装载到 VM 的磁盘时，也需关闭并解除分配 VM。 如果只想下载数据库内容，然后使用此内容在本地安装新系统，并且如果在下载和设置新系统期间，Azure 中的系统仍可以正常运行，那么，可以通过执行将压缩的数据库备份到磁盘，并只下载该磁盘而不是同时下载 OS 基本 VM，来避免停机时间过长。

#### <a name="powershell"></a>Powershell

* 下载托管磁盘  
  需要首先获取对托管磁盘基础 blob 的访问权限。 然后可将基础 blob 复制到新的存储帐户，并从该存储帐户下载 blob。

  ```powershell
  $access = Grant-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name> -Access Read -DurationInSecond 3600
  $key = (Get-AzStorageAccountKey -ResourceGroupName <resource group> -Name <storage account name>)[0].Value
  $destContext = (New-AzStorageContext -StorageAccountName <storage account name -StorageAccountKey $key)
  Start-AzStorageBlobCopy -AbsoluteUri $access.AccessSAS -DestContainer <container name> -DestBlob <blob name> -DestContext $destContext
  # Wait for blob copy to finish
  Get-AzStorageBlobCopyState -Container <container name> -Blob <blob name> -Context $destContext
  Save-AzVhd -SourceUri <blob in new storage account> -LocalFilePath <local file path> -StorageKey $key
  # Wait for download to finish
  Revoke-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name>
  ```

* 下载 VHD  
  停止 SAP 系统并关闭 VM 后，你可以在本地目标上使用 PowerShell cmdlet 保存 AzVhd VHD 磁盘下载回本地环境。 若要执行此操作，需要使用可在 Azure 门户的“存储”部分找到的 VHD URL（需要导航到存储帐户以及创建 VHD 时所在的存储容器），并且需要知道 VHD 应该复制到的位置。

  然后，可以定义参数 SourceUri 作为要下载的 VHD 的 URL，定义 LocalFilePath 作为 VHD 的物理位置（包括其名称），以利用该命令来实现此目的。 该命令如下所示：

  ```powerhell
  Save-AzVhd -ResourceGroupName <resource group name of storage account> -SourceUri http://<storage account name>.blob.core.windows.net/<container name>/sapidedata.vhd -LocalFilePath E:\Azure_downloads\sapidesdata.vhd
  ```

  有关保存 AzVhd cmdlet 的更多详细信息，查看以下网页<https://docs.microsoft.com/powershell/module/az.compute/save-Azvhd>。

#### <a name="azure-cli"></a>Azure CLI
* 下载托管磁盘  
  需要首先获取对托管磁盘基础 blob 的访问权限。 然后可将基础 blob 复制到新的存储帐户，并从该存储帐户下载 blob。
  ```
  az disk grant-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --duration-in-seconds 3600
  az storage blob download --sas-token "<sas token>" --account-name <account name> --container-name <container name> --name <blob name> --file <local file>
  az disk revoke-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>"
  ```

* 下载 VHD   
  停止 SAP 系统并关闭 VM 后，可以在本地目标上使用 Azure CLI 命令 azure storage blob download 将 VHD 磁盘下载回本地。 若要执行此操作，需要使用可在 Azure 门户的“存储”部分找到的 VHD 名称和容器（需要导航到存储帐户以及创建 VHD 时所在的存储容器），并且需要知道 VHD 应该复制到的位置。

  然后，可以定义要下载的 VHD 的参数 blob 和 container，并将目标定义为 VHD 的物理目标位置（包括其名称），以利用该命令来实现此目的。 该命令如下所示：

  ```
  az storage blob download --name <name of the VHD to download> --container-name <container of the VHD to download> --account-name <storage account name of the VHD to download> --account-key <storage account key> --file <destination of the VHD to download>
  ```

### <a name="transferring-vms-and-disks-within-azure"></a>在 Azure 内部传输 VM 和磁盘

#### <a name="copying-sap-systems-within-azure"></a>在 Azure 内部复制 SAP 系统

SAP 系统，甚至是支持 SAP 应用程序层的专用 DBMS 服务器，都有可能包括多个含有 OS 及二进制文件或者 SAP 数据库数据和日志文件的磁盘。 Azure 的复制磁盘功能以及 Azure 的保存磁盘至本地磁盘功能都没有同步机制用于以一致的方式创建多个磁盘的快照。 因此，即使已复制或已保存的磁盘是针对同一个 VM 装载的，它们的状态也会不同。 由于不同的磁盘包含不同的数据和日志文件这一铁定事实，最终的数据库也就无法保持一致。

**结论：结论：若要复制或保存属于 SAP 系统配置的磁盘，需要停止该 SAP 系统，同时还需要关闭部署的 VM。** 只有在执行此操作后，才可以复制或下载磁盘集，以便在 Azure 中或者在本地创建 SAP 系统的副本。

数据磁盘作为 VHD 文件存储在 Azure 存储帐户中，并可直接附加到虚拟机或用作映像。 在本例中，VHD 将复制到另一个位置，再附加到虚拟机。 Azure 中 VHD 文件的完整名称必须在 Azure 中唯一。 如前所述，此名称由三个部分组成，如下所示：

    http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>

数据磁盘也可以是托管磁盘。 在此情况下，先使用托管磁盘创建一个新的托管磁盘，再将其附加到虚拟机。 托管磁盘名称在资源组中必须是唯一的。

##### <a name="powershell"></a>Powershell

可按[此文][storage-powershell-guide-full-copy-vhd]中所述，使用 Azure PowerShell cmdlet 来复制 VHD。 若要创建新的托管磁盘，请使用新建 AzDiskConfig 和新建 AzDisk 如下面的示例中所示。

```powershell
$config = New-AzDiskConfig -CreateOption Copy -SourceUri "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" -Location <location>
New-AzDisk -ResourceGroupName <resource group name> -DiskName <disk name> -Disk $config
```

##### <a name="azure-cli"></a>Azure CLI

可按[此文][storage-azure-cli-copy-blobs]中所述，使用 Azure CLI 来复制 VHD。 若要创建新的托管磁盘，请使用 az disk create，如下例所示。

```
az disk create --source "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --name <disk name> --resource-group <resource group name> --location <location>
```

##### <a name="azure-storage-tools"></a>Azure 存储工具

* <https://storageexplorer.com/>

可在以下网页中找到专业版本的 Azure 存储资源管理器：

* <https://www.cerebrata.com/>
* <http://clumsyleaf.com/products/cloudxplorer>

存储帐户中的 VHD 复制过程本身需要花费几秒的时间（类似于 SAN 硬件使用迟缓复制和写入时复制功能创建快照）。 创建 VHD 文件副本后，可以将其附加到虚拟机，或用作映像以将 VHD 副本附加到虚拟机。

##### <a name="powershell"></a>Powershell

```powershell
# attach a vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -VhdUri <path to vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -ManagedDiskId <managed disk id> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a copy of the vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name <disk name> -VhdUri <new path of vhd> -SourceImageUri <path to image vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption fromImage
$vm | Update-AzVM

# attach a copy of the managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$diskConfig = New-AzDiskConfig -Location $vm.Location -CreateOption Copy -SourceUri <source managed disk id>
$disk = New-AzDisk -DiskName <disk name> -Disk $diskConfig -ResourceGroupName <resource group name>
$vm = Add-AzVMDataDisk -VM $vm -Caching <caching option> -Lun <lun, for example 0> -CreateOption attach -ManagedDiskId $disk.Id
$vm | Update-AzVM
```
##### <a name="azure-cli"></a>Azure CLI

```

# attach a vhd to a vm
az vm unmanaged-disk attach --resource-group <resource group name> --vm-name <vm name> --vhd-uri <path to vhd>

# attach a managed disk to a vm
az vm disk attach --resource-group <resource group name> --vm-name <vm name> --disk <managed disk id>

# attach a copy of the vhd to a vm
# this scenario is currently not possible with Azure CLI. A workaround is to manually copy the vhd to the destination.

# attach a copy of a managed disk to a vm
az disk create --name <new disk name> --resource-group <resource group name> --location <location of target virtual machine> --source <source managed disk id>
az vm disk attach --disk <new disk name or managed disk id> --resource-group <resource group name> --vm-name <vm name> --caching <caching option> --lun <lun, for example 0>
```

#### <a name="9789b076-2011-4afa-b2fe-b07a8aba58a1"></a>在 Azure 存储帐户之间复制磁盘
无法在 Azure 门户中执行此任务。 可以使用 Azure PowerShell cmdlet、Azure CLI 或第三方存储浏览器。 PowerShell cmdlet 或 CLI 命令可以创建和管理 Blob，包括跨 Azure 订阅中的存储帐户和区域异步复制 Blob。

##### <a name="powershell"></a>Powershell
还可在订阅之间复制 VHD。 有关详细信息，请阅读[此文][storage-powershell-guide-full-copy-vhd]。

PS cmdlet 逻辑的基本流程如下所示：

* 创建的存储帐户上下文**源**使用的存储帐户*新建 AzStorageContext* -请参阅 <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>
* 创建的存储帐户上下文**目标**使用的存储帐户*新建 AzStorageContext* -请参阅 <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>
* 开始复制

```powershell
Start-AzStorageBlobCopy -SrcBlob <source blob name> -SrcContainer <source container name> -SrcContext <variable containing context of source storage account> -DestBlob <target blob name> -DestContainer <target container name> -DestContext <variable containing context of target storage account>
```

* 检查副本在循环中的状态

```powershell
Get-AzStorageBlobCopyState -Blob <target blob name> -Container <target container name> -Context <variable containing context of target storage account>
```

* 如上所述将新 VHD 附加到虚拟机。

有关示例，请参阅[此文][storage-powershell-guide-full-copy-vhd]。

##### <a name="azure-cli"></a>Azure CLI
* 开始复制

```
az storage blob copy start --source-blob <source blob name> --source-container <source container name> --source-account-name <source storage account name> --source-account-key <source storage account key> --destination-container <target container name> --destination-blob <target blob name> --account-name <target storage account name> --account-key <target storage account name>
```

* 检查状态（如果副本仍在循环中）

```
az storage blob show --name <target blob name> --container <target container name> --account-name <target storage account name> --account-key <target storage account name>
```

* 如上所述将新 VHD 附加到虚拟机。

有关示例，请参阅[此文][storage-azure-cli-copy-blobs]。

### <a name="disk-handling"></a>磁盘处理

#### <a name="4efec401-91e0-40c0-8e64-f2dceadff646"></a>SAP 部署的虚拟机/磁盘结构

理想情况下，处理 VM 和关联磁盘的结构非常简单。 在本地安装中，客户已形成了多种用于结构化服务器安装的方式。

* 一个基础磁盘，其中包含 OS 以及 DBMS 和/或 SAP 的所有二进制文件。 从 2015 年 3 月开始，此磁盘可以有高达 1TB 的大小，而不像以前一样存在 127GB 上限。
* 一个或多个磁盘，其中包含 SAP 数据库的 DBMS 日志文件，以及 DBMS 临时存储区域的日志文件（如果 DBMS 支持此模式）。 如果数据库日志 IOPS 要求较高，则需要条带化多个磁盘，以达到所需的 IOPS 量。
* 多个磁盘，其中包含 SAP 数据库的一个或两个数据库文件，以及 DBMS 临时数据文件（如果 DBMS 支持此模式）。

![适用于 SAP 的 Azure IaaS VM 的参考配置][planning-guide-figure-1300]


- - -
> ![Windows][Logo_Windows] Windows
>
> 在许多客户那里，我们看到了类似于这样的配置：SAP 和 DBMS 二进制文件并未安装在 OS 所安装到的 C:\ 驱动器上。 采用这种配置的原因多种多样，但是，如果我们回过头来分析根本原因，通常会发现这些驱动器都很小，并且早在 10-15 年以前，就需要提供额外的空间来进行 OS 升级。 时至今日，这两种情况再也不那么常见了。 当今的 C:\ 驱动器可以映射到大型卷磁盘或 VM 上。 为了保持部署结构的简单性，建议为 Azure 中的 SAP NetWeaver 系统采用以下部署模式
>
> Windows 操作系统页面文件应该位于 D: 驱动器（非持久性磁盘）上
>
> ![Linux][Logo_Linux] Linux
>
> 将 Linux 交换文件放在 Linux 上的 /mnt /mnt/resource 下，如[此文][virtual-machines-linux-agent-user-guide]所述。 可以在 Linux 代理 /etc/waagent.conf 的配置文件中配置交换文件。 添加或更改以下设置：
>
>

```
ResourceDisk.EnableSwap=y
ResourceDisk.SwapSizeMB=30720
```

若要激活更改，需要使用以下命令重新启动 Linux 代理

```
sudo service waagent restart
```

有关建议的交换文件大小的详细信息，请参阅 SAP 说明 [1597355]

- - -
用于 DBMS 数据文件的磁盘数目以及托管这些磁盘的 Azure 存储类型应该根据 IOPS 要求和所需的延迟来确定。 [此文 (Linux)][virtual-machines-sizes-linux] 和[此文 (Windows)][virtual-machines-sizes-windows] 中介绍了确切的配额。

过去两年的 SAP 部署经验提供了一些教训，摘要如下：

* 不同数据文件的 IOPS 流量不一定相同，因为现有的客户系统可能有代表其 SAP 数据库的不同大小的数据文件。 因此，最好在多个磁盘上使用 RAID 配置，以放置从中分割出的数据文件 LUN。 有时（特别是针对 Azure 标准存储）IOPS 速率达到单个磁盘对 DBMS 事务日志的配额。 在此情况下，建议使用高级存储，或者使用软件条带聚合多个标准存储磁盘。

- - -
> ![Windows][Logo_Windows] Windows
>
> * [Azure 虚拟机中 SQL Server 的性能最佳实践][virtual-machines-sql-server-performance-best-practices]
>
> ![Linux][Logo_Linux] Linux
>
> * [在 Linux 上配置软件 RAID][virtual-machines-linux-configure-raid]
> * [在 Azure 中的 Linux VM 上配置 LVM][virtual-machines-linux-configure-lvm]
> * [Azure 存储机密和 Linux I/O 优化](https://blogs.msdn.com/b/igorpag/archive/2014/10/23/azure-storage-secrets-and-linux-i-o-optimizations.aspx)
>
>

- - -
* 事实表明，高级存储可大幅提高性能，尤其是对于关键的事务日志写入。 在必须提供产能（例如性能）的 SAP 方案中，强烈建议使用可利用 Azure 高级存储的 VM 系列。

请记住，包含 OS 和建议的 SAP 与数据库（基础 VM）二进制文件的磁盘已不再限制为 127GB。 它现在可以拥有高达 1TB 的大小。 此空间大小足以保存所有必要的文件（包括 SAP 批处理作业日志等）。

有关更多建议和详细信息（尤其是针对 DBMS VM），请参阅 [DBMS 部署指南][dbms-guide]

#### <a name="disk-handling"></a>磁盘处理

在大多数方案中，需要创建更多的磁盘，以便将 SAP 数据库部署到 VM 中。 我们已在本文档的 [SAP 部署的 VM/磁盘结构][planning-guide-5.5.1]一章中讨论了有关磁盘数量的注意事项。 部署基础 VM 后，可以在 Azure 门户中附加和分离磁盘。 当该 VM 已启动并运行或者处于停止状态时，便可以附加/分离磁盘。 在附加磁盘时，Azure 门户可让你附加空磁盘，或者当前尚未附加到其他 VM 的现有磁盘。

**注意**：在任意给定时间，磁盘只能附加到一个 VM。

![使用 Azure 标准存储附加/分离磁盘][planning-guide-figure-1400]

新虚拟机部署期间，可决定是使用托管磁盘还是将磁盘置于 Azure 存储帐户。 若要使用高级存储，建议使用托管磁盘。

然后，需要确定是创建新的空磁盘，还是选择以前上传的、现在应该附加到该 VM 的现有磁盘。

**重要说明**：对于 Azure 标准存储，请**不要**使用主机缓存。 因此，应该将“主机缓存首选项”保留为默认值“无”。 使用 Azure 高级存储时，如果 I/O 特征大多数读取为类似对数据库数据文件的一般 I/O 流量，则应该启用“读取缓存”。 在数据库事务日志文件中，不建议使用缓存。

- - -
> ![Windows][Logo_Windows] Windows
>
> [如何在 Azure 门户中附加数据磁盘][virtual-machines-linux-attach-disk-portal]
>
> 如果附加了磁盘，则需要登录到 VM 以打开 Windows 磁盘管理器。 如果未按照[为附加的磁盘设置自动装载][planning-guide-5.5.3]一章中的建议启用自动装载，则需要重新联机并初始化新附加的卷。
>
> ![Linux][Logo_Linux] Linux
>
> 如果已附加磁盘，则需要如[此文][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]中所述登录到 VM 并初始化磁盘
>
>

- - -
如果新磁盘是空磁盘，则还需要格式化该磁盘。 对于格式化，尤其是在 DBMS 数据和日志文件方面，针对 DBMS 裸机部署的建议在此处也同样适用。

如前面的 [Microsoft Azure 虚拟机的概念][planning-guide-3.2]一章中所述，Azure 存储帐户不会提供无限的 I/O 卷、IOPS 和数据卷资源。 通常，这种限制会极大地影响到 DBMS VM。 如果要部署的高 I/O 量 VM 数小于每个 Azure 订阅的 Azure 存储帐户数限制，则最佳做法可能是为每个 VM 单独使用一个存储帐户。 否则，需要考虑如何在不达到单个存储帐户的限制的情况下，在不同的存储帐户之间平衡这些 VM。 [DBMS Deployment Guide][dbms-guide]（DBMS 部署指南）中介绍了更多详细信息。 即使是对于单纯的 SAP 应用程序服务器 VM，或者最终需要更多 VHD 的其他 VM，也应该考虑到这些限制。 这些限制不适用于使用托管磁盘的情况。 若要使用高级存储，建议使用托管磁盘。

与存储帐户相关的另一个主题是，是否要对存储帐户中的 VHD 进行异地复制。 异地复制在存储帐户级别而不是在 VM 级别启用或禁用。 如果启用了异地复制，存储帐户中的 VHD 复制到同一区域中的其他 Azure 数据中心。 在决定此项设置之前，应考虑以下限制：

Azure 异地复制在 VM 中的每个 VHD 上本地执行，并且不会跨 VM 中的多个 VHD 按顺序复制 IO。 因此，代表基础 VM 的 VHD 以及附加到 VM 的任何其他 VHD 都会独立地进行复制。 这意味着，不同 VHD 中发生的更改不会同步。 不遵守 IO 写入顺序复制 IO 这一事实，意味着异地复制对于数据库分布在多个 VHD 之间的数据库服务器而言毫无价值。 除了 DBMS 以外，可能还有其他一些应用程序的进程需要在不同 VHD 中写入或处理数据，并且必须要保持更改的顺序。 如果存在这样一种要求，则不应启用 Azure 中的异地复制。 根据你是否需要或者想要为一组 VM 启用异地复制，但同时又要为另一组 VM 禁用异地复制，可以事先将 VM 及其相关 VHD 划归到已启用或已禁用异地复制的不同存储帐户中。

#### <a name="17e0d543-7e8c-4160-a7da-dd7117a1ad9d"></a>为附加的磁盘设置自动装载
- - -
> ![Windows][Logo_Windows] Windows
>
> 对于从自有的映像或磁盘创建的 VM，需要检查并最终设置自动装载参数。 通过设置此参数，VM 在重新启动或者重新部署到 Azure 中后，可以自动重新装载附加的/装载的驱动器。
> 该参数是针对 Microsoft 在 Azure 市场中提供的映像设置的。
>
> 若要设置自动装载，请在以下网页中查看命令行可执行文件 diskpart.exe 的文档：
>
> * [DiskPart 命令行选项](https://technet.microsoft.com/library/bb490893.aspx)
> * [自动装载](https://technet.microsoft.com/library/cc753703.aspx)
>
> 应该以管理员身份打开 Windows 命令行窗口。
>
> 如果附加了磁盘，则需要登录到 VM 以打开 Windows 磁盘管理器。 如果未按照[为附加的磁盘设置自动装载][planning-guide-5.5.3]一章中的建议启用自动装载，则需要重新联机并初始化新附加的卷。
>
> ![Linux][Logo_Linux] Linux
>
> 需要如[此文][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]中所述初始化新附加的空磁盘。
> 还需要将新磁盘添加到 /etc/fstab。
>
>

- - -
### <a name="final-deployment"></a>最终部署

有关最终部署和确切的步骤，尤其是在部署 SAP 扩展监视方面，请参阅[部署指南][deployment-guide]。

## <a name="accessing-sap-systems-running-within-azure-vms"></a>访问 Azure VM 中运行的 SAP 系统

对于要使用 SAP GUI 跨公共 Internet 连接到这些 SAP 系统的方案，需要应用以下过程。

本文稍后介绍其他主要方案，这些方案将连接到在本地系统与 Azure 系统之间建立了站点到站点连接（VPN 隧道）或 ExpressRoute 连接的跨界部署中的 SAP 系统。

### <a name="remote-access-to-sap-systems"></a>远程访问 SAP 系统

使用 Azure 资源管理器时，不再有类似于以前经典模型中的默认终结点。 只要符合以下情况，就可以打开 Azure 资源管理器 VM 的所有端口：

1. 未针对子网或网络接口定义网络安全组。 可通过所谓的“网络安全组”来保护发往 Azure VM 的网络流量。 有关详细信息，请参阅[什么是网络安全组 (NSG)？][virtual-networks-nsg]
2. 未针对网络接口定义 Azure 负载均衡器   

请参阅[此文][virtual-machines-azure-resource-manager-architecture]中所述的经典模型与 ARM 之间的体系结构差异。

#### <a name="configuration-of-the-sap-system-and-sap-gui-connectivity-over-the-internet"></a>Internet 上的 SAP 系统和 SAP GUI 连接的配置

请参阅此文章了解关于本主题的详细信息：<https://blogs.msdn.com/b/saponsqlserver/archive/2014/06/24/sap-gui-connection-closed-when-connecting-to-sap-system-in-azure.aspx>

#### <a name="changing-firewall-settings-within-vm"></a>更改 VM 中的防火墙设置

可能需要在虚拟机上配置防火墙，以允许 SAP 系统的入站流量。

- - -
> ![Windows][Logo_Windows] Windows
>
> 默认情况下，已打开 Azure 所部署的 VM 中的 Windows 防火墙。 现在，需要允许打开 SAP 端口，否则 SAP GUI 将无法连接。
> 为此，请按以下步骤操作：
>
> * 打开“控制面板”\“系统和安全性”\“Windows 防火墙”并转到“高级设置”。
> * 接下来，右键单击“入站规则”并选择“新建规则”。
> * 在随后出现的向导中，请选择创建新的“端口”规则。
> * 在执行向导的下一步时，请将设置保留为 TCP，并键入想要打开的端口号。 由于我们的 SAP 实例 ID 为 00，因此应该键入 3200。 如果实例具有不同的实例编号，则应打开你前面基于实例编号定义的端口。
> * 在向导的下一部分中，需要保留选中“允许连接”项。
> * 在执行向导的下一步时，需要定义是否要为域、专用网络和公共网络应用该规则。 请根据需要做出调整。 但是，要通过公共网络从外部连接到 SAP GUI，则肯定需要将该规则应用到公共网络。
> * 在向导最后一步中，对规则命名并按“完成”进行保存。
>
> 该规则将立即生效。
>
> ![端口规则定义][planning-guide-figure-1600]
>
> ![Linux][Logo_Linux] Linux
>
> 默认情况下，Azure 市场中的 Linux 映像未启用 iptables 防火墙，而 SAP 系统的连接应该运行正常。 如果启用 iptables 或其他防火墙，请参阅 iptables 或所用防火墙的文档，以允许发往端口 32xx 入站 TCP 流量（其中 xx 是 SAP 系统的系统编号）。
>
>

- - -
#### <a name="security-recommendations"></a>安全建议

SAP GUI 不会立即连接到运行中的任何 SAP 实例（端口 32xx），而是先通过打开的端口连接到 SAP 消息服务器进程（端口 36xx）。 以前，消息服务器使用完全相同的端口来与应用程序实例进行内部通信。 为了防止本地应用程序服务器意外地与 Azure 中的消息服务器通信，可以更改内部通信端口。 在从本地系统克隆的系统（例如，为了项目测试等目的而克隆的开发系统）上，强烈建议将 SAP 消息服务器与其应用程序实例之间的内部通信端口更改为一个不同的端口号。可以对默认的配置文件参数执行此更改：

> rdisp/msserv_internal
>
>

如中所述[SAP 消息服务器的安全设置](https://help.sap.com/saphelp_nwpi71/helpdata/en/47/c56a6938fb2d65e10000000a42189c/content.htm)


### <a name="3e9c3690-da67-421a-bc3f-12c520d99a30"></a>用于 SAP NetWeaver 演示/培训方案的单一虚拟机

![运行具有相同 VM 名称且在 Azure 云服务中隔离的单一 VM SAP 演示系统][planning-guide-figure-1700]

在此方案中，我们将要实施一个典型的培训/演示系统方案，其中的整个培训/演示方案包含在单个 VM 中。 我们假设部署是通过 VM 映像模板完成的。 另外，我们假设其中的多个演示/培训 VM 需要与同名的 VM 部署在一起。 整个训练系统未连接到本地资产，到混合部署相反。

假设已根据本文档的[为 Azure 准备包含 SAP 的 VM][planning-guide-5.2] 一章的某些部分中所述创建了一个 VM 映像。

用于实施该方案的事件顺序如下：

##### <a name="powershell"></a>Powershell

* 为每个培训/演示布局创建新资源组

```powershell
$rgName = "SAPERPDemo1"
New-AzResourceGroup -Name $rgName -Location "North Europe"
```
* 如不使用托管磁盘，请创建新的存储帐户

```powershell
$suffix = Get-Random -Minimum 100000 -Maximum 999999
$account = New-AzStorageAccount -ResourceGroupName $rgName -Name "saperpdemo$suffix" -SkuName Standard_LRS -Kind "Storage" -Location "North Europe"
```

* 为每个培训/演示布局创建新的虚拟网络，以允许使用相同的主机名和 IP 地址。 虚拟网络受网络安全组的保护，只允许发往端口 3389 的流量，以针对 SSH 启用远程桌面访问和端口 22。

```powershell
# Create a new Virtual Network
$rdpRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGRDP -Protocol * -SourcePortRange * -DestinationPortRange 3389 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 100
$sshRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGSSH -Protocol * -SourcePortRange * -DestinationPortRange 22 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 101
$nsg = New-AzNetworkSecurityGroup -Name SAPERPDemoNSG -ResourceGroupName $rgName -Location  "North Europe" -SecurityRules $rdpRule,$sshRule

$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name Subnet1 -AddressPrefix  10.0.1.0/24 -NetworkSecurityGroup $nsg
$vnet = New-AzVirtualNetwork -Name SAPERPDemoVNet -ResourceGroupName $rgName -Location "North Europe"  -AddressPrefix 10.0.1.0/24 -Subnet $subnetConfig
```

* 创建可用于从 Internet 访问虚拟机的新公共 IP 地址

```powershell
# Create a public IP address with a DNS name
$pip = New-AzPublicIpAddress -Name SAPERPDemoPIP -ResourceGroupName $rgName -Location "North Europe" -DomainNameLabel $rgName.ToLower() -AllocationMethod Dynamic
```

* 为虚拟机创建新网络接口

```powershell
# Create a new Network Interface
$nic = New-AzNetworkInterface -Name SAPERPDemoNIC -ResourceGroupName $rgName -Location "North Europe" -Subnet $vnet.Subnets[0] -PublicIpAddress $pip
```

* 创建虚拟机。 对于此方案，每个 VM 都具有相同的名称。 这些 VM 中的 SAP NetWeaver 实例的 SAP SID 也是相同的。 在 Azure 资源组中，VM 的名称需要唯一，但是，在不同的 Azure 资源组中，可以运行同名的 VM。 Windows 的默认“Administrator”帐户和 Linux 的“root”帐户无效。 因此，新的管理员用户名需要与密码一起定义。 此外，还需要定义 VM 的大小。

```powershell
#####
# Create a new virtual machine with an official image from the Azure Marketplace
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

# select image
$vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version "latest"
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "SUSE" -Offer "SLES-SAP" -Skus "12-SP1" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "RedHat" -Offer "RHEL" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "Oracle" -Offer "Oracle-Linux" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$diskName="osfromimage"
$osDiskUri=$account.PrimaryEndpoints.Blob.ToString() + "vhds/" + $diskName  + ".vhd"

$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Windows
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Linux
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a Managed Disk Image
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMSourceImage -VM $vmconfig -Id <Id of Managed Disk Image>
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

* （可选）添加更多磁盘，并还原所需的内容。 所有 Blob 名称（Blob 的 URL）在 Azure 中必须唯一。

```powershell
# Optional: Attach additional VHD data disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
$dataDiskUri = $account.PrimaryEndpoints.Blob.ToString() + "vhds/datadisk.vhd"
Add-AzVMDataDisk -VM $vm -Name datadisk -VhdUri $dataDiskUri -DiskSizeInGB 1023 -CreateOption empty | Update-AzVM

# Optional: Attach additional Managed Disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
Add-AzVMDataDisk -VM $vm -Name datadisk -DiskSizeInGB 1023 -CreateOption empty -Lun 0 | Update-AzVM
```

##### <a name="cli"></a>CLI

可以在 Linux 上使用以下示例代码。 对于 Windows，请使用上述 PowerShell，或改编示例以使用 %rgName% 而不是 $rgName，并使用 Windows 命令 *set* 来设置环境变量。

* 为每个培训/演示布局创建新资源组

```
rgName=SAPERPDemo1
rgNameLower=saperpdemo1
az group create --name $rgName --location "North Europe"
```

* 创建新的存储帐户

```
az storage account create --resource-group $rgName --location "North Europe" --kind Storage --sku Standard_LRS --name $rgNameLower
```

* 为每个培训/演示布局创建新的虚拟网络，以允许使用相同的主机名和 IP 地址。 虚拟网络受网络安全组的保护，只允许发往端口 3389 的流量，以针对 SSH 启用远程桌面访问和端口 22。

```
az network nsg create --resource-group $rgName --location "North Europe" --name SAPERPDemoNSG
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGRDP --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 3389 --access Allow --priority 100 --direction Inbound
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGSSH --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 22 --access Allow --priority 101 --direction Inbound

az network vnet create --resource-group $rgName --name SAPERPDemoVNet --location "North Europe" --address-prefixes 10.0.1.0/24
az network vnet subnet create --resource-group $rgName --vnet-name SAPERPDemoVNet --name Subnet1 --address-prefix 10.0.1.0/24 --network-security-group SAPERPDemoNSG
```

* 创建可用于从 Internet 访问虚拟机的新公共 IP 地址

```
az network public-ip create --resource-group $rgName --name SAPERPDemoPIP --location "North Europe" --dns-name $rgNameLower --allocation-method Dynamic
```

* 为虚拟机创建新网络接口

```
az network nic create --resource-group $rgName --location "North Europe" --name SAPERPDemoNIC --public-ip-address SAPERPDemoPIP --subnet Subnet1 --vnet-name SAPERPDemoVNet
```

* 创建虚拟机。 对于此方案，每个 VM 都具有相同的名称。 这些 VM 中的 SAP NetWeaver 实例的 SAP SID 也是相同的。 在 Azure 资源组中，VM 的名称需要唯一，但是，在不同的 Azure 资源组中，可以运行同名的 VM。 Windows 的默认“Administrator”帐户和 Linux 的“root”帐户无效。 因此，新的管理员用户名需要与密码一起定义。 此外，还需要定义 VM 的大小。

```
#####
# Create virtual machines using storage accounts
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password

#####
# Create virtual machines using Managed Disks
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
```

```
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Windows --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Linux --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd> --authentication-type password

#####
# Create a new virtual machine with a Managed Disk Image
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id> --authentication-type password
```

* （可选）添加更多磁盘，并还原所需的内容。 所有 Blob 名称（Blob 的 URL）在 Azure 中必须唯一。

```
# Optional: Attach additional VHD data disks
az vm unmanaged-disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --vhd-uri https://$rgNameLower.blob.core.windows.net/vhds/data.vhd  --new

# Optional: Attach additional Managed Disks
az vm disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --disk datadisk --new
```

##### <a name="template"></a>模板

可以在 GitHub 上的 azure 快速入门模板存储库上使用的示例模板。

* [简单的 Linux VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
* [简单的 Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
* [来自映像的 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

### <a name="implement-a-set-of-vms-that-communicate-within-azure"></a>实施一组需要在 Azure 中通信的 VM

这种非混合方案是用于实现培训和演示目的的典型方案，其中，表示演示/培训方案的软件分散在多个 VM 中。 不同 VM 中安装的不同组件需要相互通信。 同样，在此方案中，无需本地网络通信或跨界方案。

此方案是本文档的[用于 SAP NetWeaver 演示/培训的单一 VM 方案][planning-guide-7.1]一章中所述的安装的扩展。 在此情况下，要将更多的虚拟机添加到现有的资源组。 在以下示例中，训练布局包括一个 SAP ASCS/SCS VM、一个运行 DBMS 的 VM 和一个 SAP 应用程序服务器实例 VM。

在构建此方案之前，需要考虑前面的方案中已经执行的基本设置。

#### <a name="resource-group-and-virtual-machine-naming"></a>资源组和虚拟机命名

所有资源组名称必须唯一。 可以制定自己的资源命名模式，例如 `<rg-name`>-suffix。

虚拟机名称必须在资源组中必须唯一。

#### <a name="set-up-network-for-communication-between-the-different-vms"></a>设置用于在不同 VM 之间进行通信的网络

![Azure 虚拟网络中的 VM 集][planning-guide-figure-1900]

若要避免与相同培训/演示布局的副本发生命名冲突，需要为每个布局创建一个 Azure 虚拟网络。 DNS 名称解析由 Azure 提供，或者，可以在 Azure 外部配置自己的 DNS 服务器（本文档将不提供更深入的介绍）。 在此方案中，我们不需要配置自己的 DNS。 对于 Azure 虚拟网络内部的所有虚拟机，启用基于主机名的通信。

按虚拟网络而不仅仅是按资源组隔离培训或演示布局的原因可能是：

* 设置的 SAP 布局需要自身的 AD/OpenLDAP，并且域服务器需要属于每个布局。  
* 设置的 SAP 布局包含的组件需要使用固定 IP 地址。

可以在[此文][virtual-networks-create-vnet-arm-pportal]中找到有关 Azure 虚拟网络以及如何定义这种网络的详细信息。

## <a name="deploying-sap-vms-with-corporate-network-connectivity-cross-premises"></a>使用企业网络连接部署 SAP VM（跨界）

假设运行某个 SAP 布局，并想要在高端 DBMS 服务器的裸机、应用程序层的本地虚拟化环境与采用小型双层配置的 SAP 系统和 Azure IaaS 之间分割部署。 基本假设条件是一个 SAP 环境内的 SAP 系统需要彼此通信，并在公司内部部署其他许多软件组件，而不管其部署形式为何。 另外，对于使用 SAP GUI 或其他界面进行连接的最终用户而言，部署形式的差异不应造成任何体验差异。 仅当我们已通过站点到站点/多站点连接或类似于 Azure ExpressRoute 的专用连接将本地 Active Directory/OpenLDAP 和 DNS 服务扩展到了 Azure 系统时，才能满足这些条件。



### <a name="scenario-of-an-sap-landscape"></a>SAP 布局的方案

下图大致描述了跨界或混合方案：

![本地与 Azure 资产之间的站点到站点连接][planning-guide-figure-2100]

如上所示的方案描述的方案，在内部部署

最低的要求是在浏览器中使用 SSL/TLS 等安全通信协议或者在系统中使用基于 VPN 的连接来访问 Azure 服务。 假设条件是公司要以极其不同的方式处理其企业网络与 Azure 之间的 VPN 连接。 一些公司可能会完全打开所有的端口， 还有一些公司可能希望确切地知道需要打开哪些端口，等等。

下表中列出了典型的 SAP 通信端口。 基本上，打开 SAP 网关端口便已足够。

<!-- sapms is prefix of a SAP service name and not a spelling error -->

| 服务 | 端口名称 | 示例 `<nn`> = 01 | 默认范围（最小-最大） | 注释 |
| --- | --- | --- | --- | --- |
| 调度程序 |sapdp`<nn>` 请参阅 * |3201 |3200 - 3299 |SAP 调度程序，由适用于 Windows 和 Java 的 SAP GUI 使用 |
| 消息服务器 |sapms`<sid`> 请参阅 ** |3600 |free sapms`<anySID`> |sid = SAP-System-ID |
| 网关 |sapgw`<nn`> 请参阅 * |3301 |可用 |SAP 网关，用于 CPIC 和 RFC 通信 |
| SAP 路由器 |sapdp99 |3299 |免费 |仅限 CI（中心实例）。安装后，可在 /etc/services 中将服务名称重新指定为任意值。 |

*) nn = SAP 实例编号

**) sid = SAP-System-ID

可以在 <https://scn.sap.com/docs/DOC-17124> 中找到有关不同 SAP 产品或 SAP 产品的不同服务所需的端口的更多详细信息。
参考此文档应该可以在 VPN 设备中打开特定 SAP 产品和方案所需的专用端口。

在这种方案中部署 VM 时采取的其他安全措施可能包括创建一个[网络安全组][virtual-networks-nsg]来定义访问规则。

### <a name="dealing-with-different-virtual-machine-series"></a>处理不同的虚拟机系列

Microsoft 已添加更多 VM 类型，这些类型在 vCPU 数量、内存，更重要的是其运行所在的硬件方面各不相同。 SAP 不一定支持所有 VM（请在 SAP 说明 [1928533] 中查看支持的 VM 类型）。 其中一些 VM 在不同代次的主机硬件上运行。 这些代次的主机硬件以 Azure 缩放单位的数据粒度进行部署。 有时，选择的不同 VM 类型可能无法在相同的缩放单元上运行。 根据不同的硬件，可用性集跨缩放单位的能力会受到限制。  例如，如果你在位于可用性集中的 E64s_v3 VM 上以及 HA 配置中运行辅助 DBMS 实例的 VM 上运行 SAP DBMS 层，则无法方便地将辅助 VM 作为 M 系列 VM 来停止和重启，而最好是升级 VM。 原因在于，M 系列 VM 和 Ev3 系列 VM 在不同的硬件上运行，因此在不同的缩放单元中运行。 需要创建新的可用性集，删除辅助 Ev3 系列 VM（但不删除存储），并将 VM 作为 M 系列 VM 重新部署到新可用性集。

#### <a name="printing-on-a-local-network-printer-from-sap-instance-in-azure"></a>使用局域网打印机从 Azure 中的 SAP 实例打印

##### <a name="printing-over-tcpip-in-cross-premises-scenario"></a>通过跨界方案中的 TCP/IP 打印

在 Azure VM 中设置基于本地 TCP/IP 的打印机的操作总体上与在企业网络中的操作相同，但假设前提是确实已建立 VPN 站点到站点隧道或 ExpressRoute 连接。

- - -
> ![Windows][Logo_Windows] Windows
>
> 为此，请按以下步骤操作：
>
> * 某些网络打印机随附了一个配置向导，方便你在 Azure VM 中设置打印机。 如果未随打印机一起分发向导软件，手动设置打印机的方法是创建一个新的 TCP/IP 打印机端口。
> * 打开“控制面板”->“设备和打印机”->“添加打印机”
> * 选择“使用 TCP/IP 地址或主机名添加打印机”
> * 键入打印机的 IP 地址
> * 选择打印机端口标准 9100
> * 如果需要，请手动安装适当的打印机驱动程序。
>
> ![Linux][Logo_Linux] Linux
>
> * 与 Windows 一样，只需遵循标准过程来安装网络打印机
> * 只需遵循 [SUSE](https://www.suse.com/documentation/sles-12/book_sle_deployment/data/sec_y2_hw_print.html) 或 [Red Hat 和 Oracle Linux](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sec-Printer_Configuration.html) 的 Linux 公开指南，了解如何添加打印机。
>
>

- - -
![网络打印][planning-guide-figure-2200]

##### <a name="host-based-printer-over-smb-shared-printer-in-cross-premises-scenario"></a>跨界方案中使用 SMB 的基于主机的打印机（共享打印机）

基于主机的打印机在设计上与网络不兼容。 但是，只要将基于主机的打印机连接到已开机的计算机，便可以在网络中的计算机之间共享该打印机。 请通过站点到站点连接或 ExpressRoute 连接到企业网络，再共享本地打印机。 SMB 协议使用 NetBIOS 而不是 DNS 作为命名服务。 NetBIOS 主机名可以不同于 DNS 主机名。 一般情况下，NetBIOS 主机名与 DNS 主机名是相同的。 DNS 域在 NetBIOS 命名空间中没有意义。 因而，不能在 NetBIOS 命名空间中使用包括 DNS 主机名和 DNS 域的完全限定的 DNS 主机名。

打印机共享在网络中通过唯一的名称进行标识：

* SMB 主机的主机名（始终需要）。
* 共享名称（始终需要）。
* 域名（如果打印机共享与 SAP 系统不在同一个域中）。
* 此外，可能需要使用用户名和密码来访问打印机共享。

如何：

- - -
> ![Windows][Logo_Windows] Windows
>
> 共享本地打印机。
> 在 Azure VM 中打开 Windows 资源管理器，并键入打印机共享名称。
> 打印机安装向导将引导完成整个安装过程。
>
> ![Linux][Logo_Linux] Linux
>
> 下面是有关在 Linux 中配置网络打印机的一些文档示例，或者有关在 Linux 中进行打印的章节。 只要 VM 是 VPN 的一部分，打印机就能在 Azure Linux VM 中以相同的方式工作：
>
> * SLES <https://en.opensuse.org/SDB:Printing_via_SMB_(Samba)_Share_or_Windows_Share>
> * RHEL 或 Oracle Linux <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sec-Printer_Configuration.html#s1-printing-smb-printer>
>
>

- - -
##### <a name="usb-printer-printer-forwarding"></a>USB 打印机（打印机转发）

在 Azure 中，远程桌面服务功能可让用户在一个不可用的远程会话中访问其本地打印机设备。

- - -
> ![Windows][Logo_Windows] Windows
>
> 有关使用 Windows 打印的更多详细信息，请参阅此处：<https://technet.microsoft.com/library/jj590748.aspx>。
>
>

- - -
#### <a name="integration-of-sap-azure-systems-into-correction-and-transport-system-tms-in-cross-premises"></a>在跨界方案中将 SAP Azure 系统集成到纠正与传输系统 (TMS)

需要配置 SAP 更改与传输系统 (TMS)，以便在布局中的系统之间导出和导入传输请求。 假设某个 SAP 系统的开发实例 (DEV) 位于 Azure 中，而质量保证 (QA) 和生产系统 (PRD) 位于本地。 此外，假设存在一个中心传输目录。

##### <a name="configuring-the-transport-domain"></a>配置传输域

根据[配置传输域控制器](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm)中所述，在指定为传输域控制器的系统上配置传输域。 随后会创建系统用户 TMSADM，并生成所需的 RFC 目标。 可以在事务 SM59 中检查这些 RFC 连接。 必须在整个传输域中启用主机名解析。

如何：

* 在我们的方案中，我们决定使用本地 QAS 系统作为 CTS 域控制器。 调用事务 STMS。 此时会显示“TMS”对话框。 有时显示的是“配置传输域”对话框。 （仅当尚未配置传输域时，才显示此对话框。）
* 确保已经为自动创建的用户 TMSADM 授权（“SM59”->“ABAP 连接”->“TMSADM@E61.DOMAIN_E61”->“详细信息”->“实用程序(M)”->“授权测试”）。 事务 STMS 的初始屏幕应显示此 SAP 系统当前正在充当传输域的控制器，如下图所示：

![域控制器上的事务 STMS 的初始屏幕][planning-guide-figure-2300]

#### <a name="including-sap-systems-in-the-transport-domain"></a>在传输域中包含 SAP 系统

在传输域中包含 SAP 系统的操作顺序如下所述：

* 在 Azure 中的 DEV 系统上，转到传输系统（客户端 000）并调用事务 STMS。 从对话框中选择“其他配置”，并选择“将系统包含在域中”。 将域控制器指定为目标主机（[传输域中包含 SAP 系统](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0c17acc11d1899e0000e829fbbd/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)）。 现在，该系统正在等待包含到传输域中。
* 出于安全原因，现在必须返回到域控制器以确认请求。 选择“系统概览”，并选择“审批”以审批等待中的系统。 然后确认提示消息，随后将会分发该配置。

现在，此 SAP 系统已包含有关传输域中所有其他 SAP 系统的必要信息。 同时，新 SAP 系统的地址数据已发送到所有其他 SAP 系统，并且该 SAP 系统已输入到传输控制程序的传输配置文件中。 检查 RFC 是否正常工作，以及是否能够访问域的传输目录。

根据文档[更改与传输系统](https://help.sap.com/saphelp_nw70ehp3/helpdata/en/48/c4300fca5d581ce10000000a42189c/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)中所述，像平时一样继续配置传输系统。

如何：

* 确保已在本地正确配置 STMS。
* 确保 Azure 上的虚拟机可以解析传输域控制器的主机名，反之亦然。
* 调用事务 STMS ->“其他配置”->“将系统包含在域中”。
* 确认本地 TMS 系统上的连接。
* 像平时一样配置传输路由、组和层。

在采用站点到站点连接的跨界方案中，本地与 Azure 之间的延迟可能仍然很高。 如果在从开发、测试系统到生产的整个流程中都遵循传输对象的顺序，或者考虑向不同系统应用传输或支持包的话，会认识到，根据中心传输目录的位置，有一部分系统在中心传输目录中读取或写入数据时会遇到较高的延迟。 如果 SAP 布局配置中的不同系统分散在不同的数据中心，而这些数据中心之间的距离又非常远，那么，这种配置也会出现类似的情况。

要解决这种延迟，使系统能够快速地从传输目录中读取或者向其写入数据，可以设置两个 STMS 传输域（一个用于本地，一个用于 Azure 中的系统），并将其链接起来。 请查看此文档，了解 SAP TMS 中这一概念所依据的原则：<https://help.sap.com/saphelp_me60/helpdata/en/c4/6045377b52253de10000009b38f889/content.htm?frameset=/en/57/38dd924eb711d182bf0000e829fbfe/frameset.htm>。

如何：

* 使用事务 STMS 在每个位置（本地和 Azure）设置传输域 <https://help.sap.com/saphelp_nw70ehp3/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm>
* 使用域链接来链接这些域，并确认两个域之间的链接。
  <https://help.sap.com/saphelp_nw73ehp1/helpdata/en/a3/139838280c4f18e10000009b38f8cf/content.htm>
* 将配置分发到链接的系统。

#### <a name="rfc-traffic-between-sap-instances-located-in-azure-and-on-premises-cross-premises"></a>位于 Azure 与本地的 SAP 实例之间的 RFC 流量（跨界）

位于本地与 Azure 中的系统之间的 RFC 流量需要正常进行。 若要设置连接，请在需要与目标系统建立 RFC 连接的源系统中调用事务 SM59。 该配置类似于 RFC 连接的标准设置。

假设在跨界方案中，运行需要相互通信的 SAP 系统的 VM 位于同一个域中。 在这种情况下，SAP 系统之间的 RFC 连接设置，与在本地方案中使用的设置步骤和输入是相同的。

#### <a name="accessing-local-fileshares-from-sap-instances-located-in-azure-or-vice-versa"></a>从位于 Azure 中的 SAP 实例访问本地文件共享或进行相反的访问

位于 Azure 中的 SAP 实例需要访问企业内部的本地文件共享。 此外，本地 SAP 实例需要访问位于 Azure 中的文件共享。 若要启用文件共享，必须在本地系统上配置权限和共享选项。 确保打开 Azure 与数据中心之间的 VPN 或 ExpressRoute 连接上的端口。

## <a name="supportability"></a>可支持性

### <a name="6f0a47f3-a289-4090-a053-2521618a28c3"></a>适用于 SAP 的 Azure 监视解决方案

为了在 Azure 上启用对任务关键型 SAP 系统的监视，SAP 监视工具 SAPOSCOL 或 SAP 主机代理将通过适用于 SAP 的 Azure 监视扩展，从 Azure 虚拟机服务主机获取数据。 由于 SAP 的需求与 SAP 应用程序密切相关，Microsoft 决定不在 Azure 中整体实施所需的功能，而是让客户自己将所需的监视组件和配置部署到他们在 Azure 中运行的虚拟机上。 但是，监视组件的部署和生命周期管理主要由 Azure 自动进行。

#### <a name="solution-design"></a>解决方案设计

为启用 SAP 监视而开发的解决方案构建在 Azure VM 代理和扩展框架的体系结构之上。 Azure VM 代理和扩展框架的思路是允许在 VM 中安装 Azure VM 扩展库中提供的软件应用程序。 这种概念所依据的原则思想是允许（例如，在使用适用于 SAP 的 Azure 监视扩展时）将特殊的功能部署到 VM 中，并在部署时配置此类软件。

在 Azure 门户中创建 VM 时，用于处理 VM 中特定 Azure VM 扩展的“Azure VM 代理”默认注入到 Windows VM 中。 对于 SUSE、Red Hat 或 Oracle Linux，VM 代理已是 Azure 市场映像的一部分。 如果用户将 Linux VM 从本地上传到 Azure，则必须手动安装 VM 代理。

适用于 SAP 的 Azure 监视解决方案的基本构建块如下所示：

![Microsoft Azure 扩展组件][planning-guide-figure-2400]

如以上框图中所示，适用于 SAP 的监视解决方案的一个组成部分托管在 Azure VM 映像和 Azure 扩展库中，该库是全局复制的存储库，由 Azure 运营部门管理。 SAP/MS 联合团队将负责处理 SAP 的 Azure 实施，以配合 Azure 运营部门发布适用于 SAP 的新版 Azure 监视扩展。

部署新的 Windows VM 时，Azure VM 代理会自动添加到该 VM 中。 此代理的功能是协调 Azure 扩展的加载和配置，以监视 SAP NetWeaver 系统。 对于 Linux VM，Azure VM 代理已是 Azure 市场 OS 映像的一部分。

但是，客户仍需执行一个步骤， 那就是启用性能数据收集并对其进行配置。 使用 PowerShell 脚本或 CLI 命令可以自动完成有关配置的过程。 可以根据[部署指南][deployment-guide]中所述，在 Microsoft Azure 脚本中心下载该 PowerShell 脚本。

适用于 SAP 的 Azure 监视解决方案的整体体系结构如下所示：

![适用于 SAP NetWeaver 的 Azure 监视解决方案][planning-guide-figure-2500]

**有关具体的操作方法以及在部署期间使用这些 PowerShell cmdlet 或 CLI 命令的详细步骤，请遵循[部署指南][deployment-guide]中提供的说明。**

### <a name="integration-of-azure-located-sap-instance-into-saprouter"></a>将 Azure 中的 SAP 实例集成到 SAProuter

在 Azure 中运行的 SAP 实例也需要能够从 SAProuter 访问。

![SAP 路由器网络连接][planning-guide-figure-2600]

如果未建立直接 IP 连接，SAProuter 可以在参与系统之间实现 TCP/IP 通信。 它提供的优势是，无需在通信合作伙伴之间建立网络级的端到端连接。 默认情况下，SAProuter 会侦听端口 3299。
若要通过 SAProuter 连接 SAP 实例，每次尝试连接时，都需要提供 SAProuter 字符串和主机名。

## <a name="sap-netweaver-as-java"></a>SAP NetWeaver AS Java

到目前为止，本文档的内容侧重于 SAP NetWeaver 的概况或者 SAP NetWeaver ABAP 堆栈。 本精简部分列出了有关 SAP Java 堆栈的具体注意事项。 基于 SAP NetWeaver 的最重要应用程序之一是 SAP 企业门户，该应用程序完全基于 NetWeaver Java。 其他基于 SAP NetWeaver 的应用程序（例如 SAP PI 和 SAP Solution Manager）同时使用 SAP NetWeaver 的 ABAP 和 Java 堆栈。 因此，必定需要同时考虑到有关 SAP NetWeaver Java 堆栈的具体事项。

### <a name="sap-enterprise-portal"></a>SAP 企业门户

如果在跨界方案中进行部署，则在 Azure 虚拟机中设置 SAP 门户的操作与在本地安装中的操作是相同的。 由于 DNS 由本地系统执行，可以按本地的配置完成单个实例的端口设置。 一般来说，本文档中到目前为止所述的建议和限制也适用于 SAP 企业门户或 SAP NetWeaver Java 堆栈等应用程序。

![公开的 SAP 门户][planning-guide-figure-2700]

某些客户采用的一种特殊部署方案是向 Internet 直接公开 SAP 企业门户，同时通过站点到站点 VPN 隧道或 ExpressRoute 将虚拟机主机连接到公司网络。 对于这种方案，必须确保特定的端口已打开，而未被防火墙或网络安全组封锁。 

初始门户 URI 为 http （s）：`<Portalserver`>: 5XX00/irj sap 中所述的正确端口<https://help.sap.com/saphelp_nw70ehp1/helpdata/de/a2/f9d7fed2adc340ab462ae159d19509/frameset.htm>。

![终结点配置][planning-guide-figure-2800]

如果想要自定义 SAP 企业门户的 URL 和/或端口，请查看以下文档：

* [更改门户 URL](https://wiki.scn.sap.com/wiki/display/EP/Change+Portal+URL)
* [更改默认端口号和门户端口号](https://wiki.scn.sap.com/wiki/display/NWTech/Change+Default++port+numbers%2C+Portal+port+numbers)

## <a name="high-availability-ha-and-disaster-recovery-dr-for-sap-netweaver-running-on-azure-virtual-machines"></a>Azure 虚拟机上运行的 SAP NetWeaver 的高可用性 (HA) 和灾难恢复 (DR)

### <a name="definition-of-terminologies"></a>术语定义

术语“高可用性 (HA)”通常与一系列技术相关，这些技术通过相同数据中心内受冗余、容错或故障转移保护的组件，为 IT 服务提供业务连续性，以最大程度减少 IT 中断的情况。 在本例中，我们会在一个 Azure 区域中实现高可用性。

**灾难恢复 (DR)** 的目标也是最大程度地减少 IT 服务中断以及恢复服务，但此目标是通过**不同**的数据中心（通常相距数百公里）实现的。 在本例中，这通常发生同一地缘政治区域内的不同 Azure 区域或客户建立的区域之间。

### <a name="overview-of-high-availability"></a>高可用性概述

Azure 中的 SAP 高可用性的相关介绍可分为两个部分：

* Azure 基础结构高可用性，例如计算 (VM)、网络、存储等的 HA，以及提高 SAP 应用程序可用性的优点。
* SAP 应用程序高可用性，例如 SAP 软件组件的 HA：
  * SAP 应用程序服务器
  * SAP ASCS/SCS 实例
  * 数据库服务器

如何与 Azure 基础结构 HA 相结合。

Azure 中的 SAP 高可用性与本地物理或虚拟环境中的 SAP 高可用性在相比之下有一些差异。 以下 SAP 文档说明了 Windows 虚拟环境中的标准 SAP 高可用性配置：<https://scn.sap.com/docs/DOC-44415>。 不同于 Windows，Linux 没有集成 sapinst 的 SAP-HA 配置。 有关 Linux 的本地 SAP HA 的详细信息，请参阅此处：<https://scn.sap.com/docs/DOC-8541>。

### <a name="azure-infrastructure-high-availability"></a>Azure 基础结构高可用性

目前存在一个 99.9% 的单一 VM SLA。 若要大致了解单一 VM 可用性的情形，可以计算可用的不同 Azure SLA 的积：<https://azure.microsoft.com/support/legal/sla/>。

计算的基数是每月 30 天，即 43200 分钟。 因此，0.05% 停机时间就相当于 21.6 分钟。 像往常一样，不同服务的可用性按以下方式相乘：

(可用性服务 #1/100) * (可用性服务 #2/100) * (可用性服务 #3/100) 

例如：

(99.95/100) * (99.9/100) * (99.9/100) = 0.9975，即总体可用性为 99.75%。

#### <a name="virtual-machine-vm-high-availability"></a>虚拟机 (VM) 高可用性

有两种类型的 Azure 平台事件可能影响到虚拟机的可用性：计划内维护和计划外维护。

* 计划内维护事件是指由 Microsoft 对底层 Azure平台进行定期更新，以改进虚拟机运行时所在的平台基础结构的总体可靠性、性能和安全性。
* 计划外维护事件见于虚拟机所在硬件或物理基础结构出现某类故障的情况。 此类故障可能包括：本地网络故障、本地磁盘故障，或者其他机架级别的故障。 检测到此类故障时，Azure 平台会自动将虚拟机从所在的不健康物理服务器迁移到健康的物理服务器。 此类事件很少见，但也会导致虚拟机重启。

可以在此文档中找到更多详细信息：<https://azure.microsoft.com/documentation/articles/virtual-machines-manage-availability>

#### <a name="azure-storage-redundancy"></a>Azure 存储冗余

始终复制 Microsoft Azure 存储帐户中的数据以确保持久性和高可用性，并且即使在遇到临时硬件故障时也符合 Azure 存储 SLA 要求。

由于 Azure 存储默认保留 3 个数据映像，因此不需要部署跨多个 Azure 磁盘的 RAID5 或 RAID1。

可以在此文章中找到更多详细信息：<https://azure.microsoft.com/documentation/articles/storage-redundancy/>

#### <a name="utilizing-azure-infrastructure-vm-restart-to-achieve-higher-availability-of-sap-applications"></a>利用 Azure 基础结构 VM 重新启动来实现 SAP 应用程序的更高可用性

如果确定不使用 Windows Server 故障转移群集 (WSFC) 或 Linux 上的 Pacemaker（当前仅支持 SLES 12 及更高版本）等功能，可利用 Azure VM 重启来保护 SAP 系统，以防止 Azure 物理服务器基础结构和整体基础 Azure 平台发生计划内和计划外的停机。

> [!NOTE]
> 请注意，Azure VM 重新启动主要是保护 VM 而不是应用程序。 VM 重新启动并未提供 SAP 应用程序的高可用性，而是提供特定基础结构级别的可用性，因而间接实现 SAP 系统的更高可用性。 此外，在计划内或计划外的主机中断之后重新启动 VM 所需的时间也没有 SLA。 因此，此高可用性方法不适用于 SAP 系统的关键组件，例如 (A)SCS 或 DBMS。
>
>

高可用性的另一个重要基础结构要素是存储。 例如，Azure 存储 SLA 可用性为 99.9%。 如果用户将所有 VM 及其磁盘部署到单个 Azure 存储帐户，当 Azure 存储不可用时，将导致 Azure 存储帐户中的所有 VM 以及这些 VM 内运行的所有 SAP 组件不可用。  

也可以针对每个 VM 使用专用存储帐户，而不是将所有 VM 放入单个 Azure 存储帐户；这样，就可以通过使用多个独立的 Azure 存储帐户，来增加整体 VM 和 SAP 应用程序可用性。

Azure 托管磁盘自动放置在其附加到的虚拟机的容错域中。 如果在一个可用性集中放置两个虚拟机并使用托管磁盘，则平台还会负责将托管磁盘分发到不同的容错域中。 若要使用高级存储，强烈建议仍使用托管磁盘。

使用 Azure 基础结构 HA 和存储帐户的 SAP NetWeaver 系统的示例体系结构如下所示：

![利用 Azure 基础结构 HA 来实现 SAP 应用程序的更高可用性][planning-guide-figure-2900]

使用 Azure 基础结构 HA 和托管磁盘的 SAP NetWeaver 系统的示例体系结构如下所示：

![利用 Azure 基础结构 HA 来实现 SAP 应用程序的更高可用性][planning-guide-figure-2901]

针对关键 SAP 组件，我们目前为止已实现：

* SAP 应用程序服务器 (AS) 的高可用性

  SAP 应用程序服务器实例是冗余组件。 每个 SAP AS 实例都部署在自己的 VM 上，并在不同的 Azure 容错域和升级域中运行（请参阅[容错域][planning-guide-3.2.1]和[升级域][planning-guide-3.2.2]部分）。 使用 Azure 可用性集可确保实现此目的（请参阅 [Azure 可用性集][planning-guide-3.2.3]一章）。 当 Azure 容错或升级域可能因计划内或计划外而不可用时，会导致有限数目的 VM 及其 SAP AS 实例不可用。

  每个 SAP 实例都在自身的 Azure 存储帐户中 - 当一个 Azure 存储可能不可用时，只导致一个 VM 及其 SAP AS 实例不可用。 不过请注意，一个 Azure 订阅中的 Azure 存储帐户数目有限。 为了确保在 VM 重启后自动启动 (A)SCS 实例，请务必设置[对 SAP 实例使用 Autostart][planning-guide-11.5] 一章所述的 (A)SCS 实例启动配置文件中的 Autostart 参数。
  有关详细信息，另请参阅 [SAP 应用程序服务器的高可用性][planning-guide-11.4.1]一章。

  即使使用托管磁盘，这些磁盘也会存储在 Azure 存储帐户中，且在出现存储中断时不可用。

* SAP (A)SCS 实例的*更高*可用性

  我们在此处使用 Azure VM 重新启动，来保护已安装 SAP (A)SCS 实例的 VM。 如果 Azure 服务器发生计划内或计划外的停机，则在另一个可用的服务器上重新启动 VM。 如前所述，在此 (A)SCS 实例方案中，Azure VM 重新启动主要是保护 VM 而不是应用程序。 通过 VM 重启，可以间接实现 SAP (A)SCS 实例的更高可用性。 为了确保在 VM 重新启动后自动启动 (A)SCS 实例，请务必设置[对 SAP 实例使用 Autostart][planning-guide-11.5] 一章所述的 (A)SCS 实例启动配置文件中的 Autostart 参数。 这意味着，在单个 VM 上作为单一故障点 (SPOF) 运行的 (A)SCS 实例将是整个 SAP 布局可用性的决定因素。

* DBMS 服务器的*更高*可用性

  类似于 SAP (A)SCS 实例使用方案，我们在此使用 Azure VM 重新启动，来保护已安装 DBMS 软件的 VM，并通过 VM 重新启动实现 DBMS 软件的更高可用性。
  在单个 VM 中运行的 DBMS 也是 SPOF，它是整个 SAP 布局可用性的决定因素。

### <a name="sap-application-high-availability-on-azure-iaas"></a>Azure IaaS 上的 SAP 应用程序高可用性

为了实现整个 SAP 系统的高可用性，需要保护所有关键 SAP 系统组件（例如冗余的 SAP 应用程序服务器），以及 SAP (A)SCS 实例和 DBMS 等独特的组件（例如单一故障点）。

#### <a name="5d9d36f9-9058-435d-8367-5ad05f00de77"></a>SAP 应用程序服务器的高可用性

对于 SAP 应用程序服务器/对话实例，不必要考虑特定高可用性解决方案。 高可用性可以通过冗余来实现，因此必须在不同的虚拟机中有足够的冗余。 它们应该全部放同一个 Azure 可用性集中，以避免在计划内维护停机时间内同时更新 VM。 [升级域][planning-guide-3.2.2]一章已介绍了在 Azure 缩放单元内不同升级域和容错域上构建的基本功能。 本文档的 [Azure 可用性集][planning-guide-3.2.3]一章介绍了 Azure 可用性集。

Azure 缩放单元内的 Azure 可用性集可使用不限数目的容错和升级域。 这意味着，如果将一些 VM 放在一个可用性集中，迟早会有多个 VM 出现在同一个容错或升级域中。

将一些 SAP 应用程序服务器实例部署在其专用 VM 中，并假设有 5 个升级域，最终将出现下图所示的情况。 可用性集内的容错域和更新域的实际数目上限将来可能会变化：

![Azure 中 SAP 应用程序服务器的 HA][planning-guide-figure-3000]

可以在此文档中找到更多详细信息：<https://azure.microsoft.com/documentation/articles/virtual-machines-manage-availability>

#### <a name="high-availability-for-sap-central-services-on-azure"></a>SAP Central Services on Azure 的高可用性

有关 SAP Central Services on Azure 的高可用性体系结构，请查看 [SAP NetWeaver 的高可用性体系结构和方案](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios)一文中的入门信息。 该文提供了适用于特定操作系统的更多详细信息的链接。

#### <a name="high-availability-for-the-sap-database-instance"></a>SAP 数据库实例的高可用性

典型的 SAP DBMS HA 设置基于两个 DBMS VM，在这些 VM 中，我们可以使用 DBMS 高可用性功能将主动 DBMS 实例中的数据复制到第二个 VM 的被动 DBMS 实例中。

[DBMS 部署指南][dbms-guide]中介绍了一般 DBMS 及特定 DBMS 的高可用性和灾难恢复功能。

#### <a name="end-to-end-high-availability-for-the-complete-sap-system"></a>整个 SAP 系统的端到端高可用性

下面是 Azure 中整个 SAP NetWeaver HA 体系结构的两个示例 - 一个适用于 Windows，另一个适用于 Linux。

仅限非托管磁盘：当部署许多 SAP 系统，并且所部署的 VM 数目将超过每个订阅的存储帐户上限时，可能需要稍微折衷以下所述的概念。 在此情况下，VM 的 VHD 需要合并到一个存储帐户中。 通常可通过合并不同 SAP 系统的 SAP 应用程序层 VM 的 VHD 来实现此目的。  也可以将不同 SAP 系统的不同 DBMS VM 的不同 VHD 合并到一个 Azure 存储帐户中。 由此记住 Azure 存储帐户 的 IOPS 限制 (<https://azure.microsoft.com/documentation/articles/storage-scalability-targets>)


##### <a name="windowslogowindows-ha-on-windows"></a>![Windows][Logo_Windows] Windows 上的 HA

![Azure IaaS 中包含 SQL Server 的 SAP NetWeaver 应用程序 HA 体系结构][planning-guide-figure-3200]

以下 Azure 构造适用于 SAP NetWeaver 系统，可将基础结构问题和主机修补的影响降到最低：

* Azure 上已部署整个系统（必需 - DBMS 层、 (A)SCS 实例和完整应用程序层必须在相同位置运行）。
* 整个系统在一个 Azure 订阅内运行（必需）。
* 整个系统在一个 Azure 虚拟网络内运行（必需）。
* 可以将一个 SAP 系统的多个 VM 分成三个可用性集，即使所有 VM 都属于同一虚拟网络也是如此。
* 每个层（例如 DBMS、ASCS、应用程序服务器）必须使用专用的可用性集。
* 运行一个 SAP 系统的 DBMS 实例的所有 VM 都在一个可用性集中。 假设有多个 VM 针对每个系统运行 DBMS 实例，因为使用了本地 DBMS 高可用性功能，例如 SQL Server AlwaysOn 或 Oracle Data Guard。
* 所有运行 DBMS 实例的 VM 都使用自己的存储帐户。 DBMS 数据和日志使用同步数据的 DBMS 高可用性功能，从一个存储帐户复制到另一个存储帐户。 无法使用一个存储帐户导致无法使用一个 SQL Windows 群集节点，而不是整个 SQL Server 服务。
* 运行一个 SAP 系统之 (A)SCS 实例的所有 VM 都在一个可用性集中。 在这些 VM 中，配置 Windows Sever 故障转移群集 (WSFC) 来保护 (A)SCS 实例。
* 所有运行 (A)SCS 实例的 VM 都使用自己的存储帐户。 (A)SCS 实例文件和 SAP 通用文件夹使用 SIOS DataKeeper 复制，从一个存储帐户复制到另一个存储帐户。 无法使用一个存储帐户导致无法使用一个 (A)SCS Windows 群集节点，而不是整个 (A)SCS 服务。
* 所有代表 SAP 应用程序服务器层的 VM 都在第三个可用性集中。
* 所有运行 SAP 应用程序服务器的 VM 都使用自己的存储帐户。 无法使用一个存储帐户导致无法使用一部 SAP 应用程序服务器，而其他 SAP 应用程序服务器则可继续运行。

下图显示使用托管磁盘的相同布局。

![Azure IaaS 中包含 SQL Server 的 SAP NetWeaver 应用程序 HA 体系结构][planning-guide-figure-3201]

##### <a name="linuxlogolinux-ha-on-linux"></a>![Linux][Logo_Linux] Linux 上的 HA

Azure 上 Linux 的 SAP HA 体系结构基本上与上述 Windows 相同。 有关支持的高可用性解决方案的列表，请参阅 SAP 说明 [1928533]。

### <a name="4e165b58-74ca-474f-a7f4-5e695a93204f"></a>对 SAP 实例使用 Autostart

SAP 提供了相应的功能，用于在启动 VM 中的 OS 后立即启动 SAP 实例。 有关确切步骤，请参阅 SAP 知识库文章 [1909114]。 但是，SAP 不再建议使用此设置，因为假设多个 VM 已受到影响或每个 VM 已运行多个实例，将无法控制实例重新启动的顺序。 假设这是一个 VM 中有一个 SAP 应用程序服务器实例的典型 Azure 方案，并且这是最终重新启动单个 VM 的方案，则 Autostart 并不重要，可以通过将此参数添加到以下位置来启用它：

    Autostart = 1

运行 SAP ABAP 和/或 Java 实例的启动配置文件。

> [!NOTE]
> Autostart 参数可能还有一些缺点。 具体而言，此参数在启动实例的相关 Windows/Linux 服务时，会触发 SAP ABAP 或 Java 实例的启动。 操作系统启动时，肯定会出现这种情况。 但是，SAP 软件生命周期管理功能（例如汇总或其他更新或升级）也经常需要重新启动 SAP 服务。 这些功能完全不要求自动重新启动实例。 因此，在运行此类任务之前，应该禁用 Autostart 参数。 群集化的 SAP 实例（例如 ASCS/SCS/CI）也不应该使用 Autostart 参数。
>
>

有关自动启动 SAP 实例的其他信息，请参阅此处：

* [随 Unix 服务器启动/停止一起启动/停止 SAP](https://scn.sap.com/community/unix/blog/2012/08/07/startstop-sap-along-with-your-unix-server-startstop)
* [启动和停止 SAP NetWeaver 管理代理](https://help.sap.com/saphelp_nwpi711/helpdata/en/49/9a15525b20423ee10000000a421938/content.htm)
* [如何启用 HANA 数据库的自动启动](http://www.freehanatutorials.com/2012/10/how-to-enable-auto-start-of-hana.html)

### <a name="larger-3-tier-sap-systems"></a>更大的第 3 层 SAP 系统
上述部分已谈到第 3 层 SAP 配置的高可用性方面。 但如果 DBMS 服务器要求太高而无法位于 Azure，但可将 SAP 应用程序层部署到 Azure 的系统又该怎么办？

#### <a name="location-of-3-tier-sap-configurations"></a>第 3 层 SAP 配置的位置
不支持在本地与 Azure 之间分区应用程序层本身，或拆分应用程序和 DBMS 层。 SAP 系统可以完全在本地或在 Azure 中部署。 此外，不支持在本地运行一些应用程序服务器，并在 Azure 中运行其他一些应用程序服务器。 这是讨论的起点。 我们也不支持将 SAP 系统的 DBMS 组件和 SAP 应用程序服务器层部署在两个不同的 Azure 区域。 例如，DBMS 位于美国西部，SAP 应用程序层位于美国中部。 不支持此类配置是因为 SAP NetWeaver 体系结构的延迟敏感性。

但是，在去年一整年，数据中心合作伙伴已开发 Azure 区域的共置。 这些共置通常接近 Azure 区域内的物理 Azure 数据中心。 距离变短和通过 ExpressRoute 将共置资产连接到 Azure 可将延迟缩短为不到 2 毫秒。 在此情况下，可以将 DBMS 层（包括 SAN/NAS 存储）放在此类共置中，并将 SAP 应用程序层放在 Azure 中。 [HANA 大型实例](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)。 

### <a name="offline-backup-of-sap-systems"></a>脱机备份 SAP 系统
根据所选的 SAP 配置（第 2 层或第 3 层），可能需要执行备份。 备份包括 VM 自身的内容以及数据库。 应该使用数据库方法执行 DBMS 相关的备份。 可以在 [DBMS Guide][dbms-guide]（DBMS 指南）中找到针对不同数据库的详细说明。 另外，SAP 数据可以通过本部分所述的脱机方式（也包括数据库内容）或者下一部分所述的联机方式来备份。

简单而言，执行脱机备份要求通过 Azure 门户关闭 VM，并将基础 VM 磁盘以及所有附加磁盘复制到 VM。 这样就能保存 VM 及其关联磁盘的时间点映像。 建议将备份复制到一个不同的 Azure 存储帐户。 因此，可以应用本文档的[在 Azure 存储帐户之间复制磁盘][planning-guide-5.4.2]一章中所述的过程。
除了使用 Azure 门户关闭以外，还可以通过 PowerShell 或 CLI 执行此操作，如此处所述：<https://azure.microsoft.com/documentation/articles/virtual-machines-deploy-rmtemplates-powershell/>

还原该状态需要删除基础 VM 以及基础 VM 的原始磁盘和装载的磁盘，将保存的磁盘复制回原始存储帐户或托管磁盘的资源组，然后重新部署系统。
以下文章举例说明了如何使用 Powershell 编写此过程的脚本：<http://www.westerndevs.com/azure-snapshots/>

请确保安装新的 SAP 许可证，因为按如上所述还原 VM 备份会创建新的硬件密钥。

### <a name="online-backup-of-an-sap-system"></a>联机备份 SAP 系统

[DBMS 指南][dbms-guide]中介绍了如何使用特定于 DBMS 的方法执行 DBMS 备份。

可以使用 Azure 虚拟机备份功能来备份 SAP 系统内的其他 VM。 Azure 虚拟机备份是目前在 Azure 中备份完整 VM 的标准方法。 Azure 备份将备份存储在 Azure 中，并允许再次还原 VM。

> [!NOTE]
> 截至 2015 年 12 月，使用 VM 备份并不保留用于 SAP 许可的唯一 VM ID。 这意味着，从 VM 备份还原需要安装新的 SAP 许可证密钥，因为还原的 VM 被视为新的 VM，而不是替代以前保存的旧版。
>
> ![Windows][Logo_Windows] Windows
>
> 理论上，如果 DBMS 系统支持 Windows VSS（卷影复制服务 <https://msdn.microsoft.com/library/windows/desktop/bb968832(v=vs.85).aspx>），就能以一致的方式备份运行数据库的 VM（比如 SQL Server 就可以）。
> 不过请注意，无法根据 Azure VM 备份时间点来还原数据库。 因此，建议使用 DBMS 功能执行数据库备份，而不要依赖于 Azure VM 备份。
>
> 若要熟悉 Azure 虚拟机备份，请从此处开始：<https://docs.microsoft.com/azure/backup/backup-azure-vms>。
>
> 也可以结合使用安装在 Azure VM 中的 Microsoft Data Protection Manager 及 Azure 备份来备份/还原数据库。 可在此处找到更多信息：<https://docs.microsoft.com/azure/backup/backup-azure-dpm-introduction>。  
>
> ![Linux][Logo_Linux] Linux
>
> Windows VSS 在 Linux 中没有等效功能。 因此，只能执行文件一致性备份，而无法执行应用程序一致性备份。 SAP DBMS 备份应该使用 DBMS 功能来执行。 例如，可以使用 tar 来保存包含 SAP 相关数据的文件系统，如此处所述：<https://help.sap.com/saphelp_nw70ehp2/helpdata/en/d3/c0da3ccbb04d35b186041ba6ac301f/content.htm>
>
>

### <a name="azure-as-dr-site-for-production-sap-landscapes"></a>Azure 用作 SAP 生产布局的 DR 站点

从 2014 年中期开始，借助 Hyper-V、System Center 和 Azure 的各种组件扩展可以使用 Azure 作为在本地运行的、基于 Hyper-V 的 VM 的 DR 站点。

<https://blogs.msdn.com/b/saponsqlserver/archive/2014/11/19/protecting-sap-solutions-with-azure-site-recovery.aspx> 博客中详细介绍了如何部署此解决方案。

## <a name="summary"></a>摘要

Azure 中 SAP 系统的高可用性要点如下：

* 目前，无法以与本地完全相同的方式来保护 SAP 单一故障点。 这是因为在 Azure 中还无法在不使用第三方软件的情况下构建共享磁盘群集。
* 对于 DBMS 层，需要使用不依赖于共享磁盘群集技术的 DBMS 功能。 [DBMS Guide][dbms-guide]（DBMS 指南）中提供了详细信息。
* 要将 Azure 基础结构或主机维护中的容错域问题的影响降到最低，应该使用 Azure 可用性集：
  * 建议针对 SAP 应用程序层使用一个可用性集。
  * 建议针对 SAP DBMS 层使用另一个可用性集。
  * 不建议对不同 SAP 系统的 VM 应用相同的可用性集。
  * 建议使用高级托管磁盘。
* 若要备份 SAP DBMS 层，请参阅 [DBMS 指南][dbms-guide]。
* 备份 SAP 对话实例没有太大帮助，因为重新部署简单的对话实例通常更快。
* 备份包含 SAP 系统全局目录的 VM 和不同实例的所有配置文件很有帮助，应该通过 Windows 备份或者诸如 Linux 上的 tar 来执行此操作。 由于 Windows Server 2008 (R2) 与 Windows Server 2012 (R2) 之间存在差异，因此使用更新版 Windows Server 可以更轻松地进行备份，建议运行 Windows Server 2012 (R2) 作为 Windows 来宾操作系统。

## <a name="next-steps"></a>后续步骤
阅读以下文章：

- [SAP NetWeaver 的 Azure 虚拟机部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/deployment-guide)
- [部署适用于 SAP 工作负荷的 Azure 虚拟机 DBMS 的注意事项](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)
- [Azure 上的 SAP HANA 基础结构配置和操作](https://docs.microsoft.com/
- azure/virtual-machines/workloads/sap/hana-vm-operations)
