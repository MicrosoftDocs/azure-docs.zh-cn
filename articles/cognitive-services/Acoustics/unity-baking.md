---
title: 项目音响效果 Unity 烘焙教程
titlesuffix: Azure Cognitive Services
description: 本教程介绍了在 Unity 中使用项目音响效果进行音效烘焙。
services: cognitive-services
author: kegodin
manager: nitinme
ms.service: cognitive-services
ms.subservice: acoustics
ms.topic: tutorial
ms.date: 03/20/2019
ms.author: kegodin
ms.openlocfilehash: 2f0fcdcdf781c86179b67eeef0223d46da0fc65b
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64917002"
---
# <a name="project-acoustics-unity-bake-tutorial"></a>项目音响效果 Unity 烘焙教程
本教程介绍了在 Unity 中使用项目音响效果进行音效烘焙。

所需软件：
* 用于 Windows 的 [Unity 2018.2 及更高版本](https://unity3d.com)
* [Unity 项目中集成的项目音响效果插件](unity-integration.md)或[项目音响效果 Unity 示例内容](unity-quickstart.md)
* 可选：[Azure Batch 帐户](create-azure-account.md)，它用于通过云计算加快烘焙速度

## <a name="open-the-project-acoustics-bake-window"></a>打开项目音响效果烘焙窗口
选择“Window”，再从 Unity 菜单中选择“音效”：

![Unity 编辑器的屏幕截图，突出显示了“音效”窗口菜单选项](media/window-acoustics.png)

## <a name="create-a-navigation-mesh"></a>创建导航网格
项目音响效果使用一个导航网格来放置用于模拟的侦听器探测点。 你可使用 Unity 的[导航网格工作流](https://docs.unity3d.com/Manual/nav-BuildingNavMesh.html)，或使用其他 3D 模型包来自行设计网格。 

## <a name="mark-acoustic-scene-objects"></a>标记音效场景对象
项目音响效果依靠两种类型的场景对象进行模拟：将在模拟中反射和屏蔽声音的对象，以及对模拟中的侦听器探测点进行约束的播放器导航网格。 这两种对象类型均使用“对象”选项卡进行标记。 

由于标记对象时仅向对象添加 AcousticsGeometry 或 AcousticsNavigation 组件，因此还可使用[标准Unity 组件工作流](https://docs.unity3d.com/Manual/UsingComponents.html)来标记或取消标记对象。 只能标记网格呈现器和地形。 系统会忽略其他所有对象类型。 复选框会标记或取消标记受影响的所有对象。

### <a name="mark-acoustic-occlusion-and-reflection-geometry"></a>标记声音屏蔽和反射几何体
打开“音效”窗口的“对象”选项卡。 如果对象应屏蔽、反射或吸收声音，则将此对象标记为“音效几何体”。 音效几何体可包含地面、墙壁、屋顶、窗户和窗玻璃、地毯以及大型家具等物体。 你可对这些对象使用任意复杂度级别。 由于场景已在模拟前像素化处理，因此与简单化的对象相比，详细网格（例如带有很多小树叶的树木）的成本不会增加太多。

不要包含不得对音效造成影响的物体，例如不可见碰撞体网格。

制作结果中修复了对象在（通过下面的“探测”选项卡）探测计算时的转换。 移动场景中任何标记的对象都需要重做探测计算并重做场景。

### <a name="mark-the-navigation-mesh"></a>标记导航网格
音效系统选取使用 Unity 的工作流创建的导航网格。 要使用自己的网格，请通过“对象”选项卡进行标记。

### <a name="for-reference-the-objects-tab-parts"></a>供参考：“对象”选项卡部件
选项卡页的控件部件有：

1. 选项卡选择按钮（已选择“对象”选项卡）。 使用这些按钮可从左到右演示进行音效制作的各个步骤。
2. 有关使用此页面需要执行的操作的简短说明。
3. 层次结构窗口的适用筛选器。 使用该筛选器可按指定类型的对象筛选层次结构窗口，以便更轻松标记这些窗口。 如果还没有为音效标记任何对象，选择最后两个选项不会显示任何内容。 但是如果进行了标记，则它们可用于查找标记的对象。
4. 如果未选择任何对象，本部分则显示场景中所有对象的状态：
    * 总计 - 场景中活动的非隐藏对象的总数。
    * 忽略 - 非 Mesh Renderer 或 Terrain 对象的数目。
    * 网格 - 场景中 Mesh Renderer 对象的数目
    * 地形 - 场景中 Terrain 对象的数目
    * 几何体 - 场景中标记为“声学几何体”的 Mesh 或 Terrain 对象的数目
    * 导航 - 场景中标记为“音效导航”的 Mesh 或 Terrain 对象的数目。 该数目不包括 Unity 的 NavMesh。
5. 显示场景中“可标记”对象的总数，该对象仅限 Mesh Renderer 和 Terrain。 显示可用于为音效将这些对象标记（向其添加相应的组件）为几何体或导航的复选框
6. 若未选择任何对象，在需要时，此说明可提醒选择对象进行标记。 此外，还可以选中一个或两个复选框以标记场景中的所有对象，无需选择所有对象。
7. 若已选择某些对象，本部分则仅显示场景中所选对象的状态。
8. 显示“可标记”选定对象的总数。 选中或取消选中复选框将仅标记或取消标记选定对象。

如果场景中未选择任何内容，则“对象”选项卡将如下图所示：

![“音效对象”选项卡的屏幕截图，没有选择任何项](media/objects-tab-no-selection-detail.png)

如果选择了场景或层次结构窗口中的某些对象，将如下图所示：

![“音效对象”选项卡的屏幕截图，显示了所选项](media/objects-tab-selection-detail.png)

如果部分对象已标记，部分未标记，相应的复选框则显示“混合”值：

![“音效对象”选项卡的屏幕截图，突出显示了混合选择图标](media/mixed-object-selection-detail.png)

单击复选框可强制标记所有对象，再次单击可取消标记所有对象。

可同时为几何体和导航标记对象。

## <a name="select-acoustic-materials"></a>选择音效材料
标记对象后，请单击“材料”按钮并分配音效材料。 项目音响效果材料系统是绑定到 Unity 视觉材料系统中的：两个具有不同音效材料的对象必须具有不同的视觉材料。

声学材料控制从每个表面反射回来的声音能量的大小。 默认音效材料具有与混凝土类似的吸收能力。 项目音响效果根据视觉材料的名称推荐材料。 你可向材料分配“自定义”音效材料，以启用吸收系数滑块。

房间中给定材料的混响时间与其吸收系数成反比，并且大多数材料的吸收系数值介于 0.01 到 0.20 之间。 如果材料的吸收系数超出此范围，则其吸收性很强。

![此图显示混响时间与吸收系数的负相关性](media/reverb-time-graph.png)

### <a name="for-reference-parts-of-the-materials-tab"></a>供参考：“材料”选项卡的控件部件
![Unity 中“音效材料”选项卡的屏幕截图](media/materials-tab-detail.png)

1. “材料”选项卡按钮，用于打开此页面。
2. 有关使用此页面需要执行的操作的简短说明。
3. 选中后，仅列出标记为“声学几何体”的对象所用的材料。 否则，将列出场景中使用的所有材料。
4. 使用以下选项可更改单击下方“音效”列 (#6) 中的下拉列表时所示下拉列表菜单的顺序。 “名称”按名称对音效材料进行排序。 “吸收能力”按吸收能力从低到高对材料进行排序。
5. 场景中所用的材料列表，按字母顺序排序。 如果已选中“仅显示已标记”复选框 (#3)，则仅显示标记为“声学几何体”的对象所用的材料。 此时单击材料可选择场景中使用该材料的所有对象。
6. 显示已分配给场景材料的声学材料。 单击下拉列表，向其他声学材料重新分配场景材料。 使用上方的“音效排序依据:”选项 (#4) 可更改单击此处的项目时所示的菜单排序顺序。
7. 显示上一列中所选材料的噪音吸收系数。 值为零表示完美反射（无吸收），而值为 1 表示完美吸收（无反射）。 吸收系数无法更改，除非所选材料为“自定义”。
8. 若材料已分配给“自定义”，则滑块不再禁用，可以使用滑块或键入值来选择吸收系数。

## <a name="calculate-and-review-listener-probe-locations"></a>计算并检查侦听器探测位置
分配材料后，切换到“探测”选项卡，再单击“计算”以放置侦听器探测点进行模拟。

### <a name="what-the-calculate-button-calculates"></a>“计算...”按钮计算的内容
“计算...”按钮使用所选的音效场景几何体来为模拟准备场景：

1. 它采用场景网格中的几何体并计算体素体。 体素体是一个三维体，包含整个场景，并由较小的立方“体素”组成。 体素的大小由模拟频率决定，而模拟频率通过“模拟分辨率”设置进行设定。 每个体素都被标记为“露天”或包含场景几何体。 如果体素包含几何体，则使用分配给该几何体的材料的吸收系数标记该体素。
2. 它使用导航网格来放置侦听器探测点。 该算法在空间覆盖面和模拟时间及文件大小等计算考虑因素之间达到平衡，同时确保走廊狭长和空间狭小时都始终能覆盖一定面积。 典型的探测点计数跨度很大，从针对小场景的 100 到针对大场景的数千个。

这些计算可能需要几分钟，具体取决于场景的大小和计算机的速度。

### <a name="review-voxel-and-probe-placement"></a>检查体素和探针布置情况
预览体素数据和探测点位置，确保已准备好烘焙场景。 如果导航网格不完整或者音效几何体缺乏或多余，则通常很快就能在预览中呈现。 可使用 Gizmos 菜单启用或禁用体素和探测定位：

![Unity 中 Gizmos 菜单的屏幕截图](media/gizmos-menu.png)

包含音效几何体的体素显示为绿色立方体。 请探索你的场景并验证应为几何体的所有内容是否都有体素。 场景照相机必须在对象 5 米范围内，以便体素显示。

如果比较所创建的体素在低分辨率和高分辨率下的效果，你将看到低分辨率的体素是高分辨率的两倍大。

![Unity 编辑器中低分辨率体素预览的屏幕截图](media/voxel-cubes-preview.png)

在运行时，模拟结果插入到侦听器探测点位置之间。 请检查播放器应在场景中遍历的任何位置附近是否有探测点。

![Unity 编辑器中探测预览的屏幕截图](media/probes-preview.png)

### <a name="take-care-with-scene-renames"></a>场景重命名时要小心
场景名称用于连接场景和存储探测点放置和体素化的文件。 如果在计算探测点后重命名场景，材料的分配和放置数据将丢失，应重新运行。

### <a name="for-reference-parts-of-the-probes-tab"></a>供参考：“探测”选项卡的控件部件
![Unity 中“音效探测”选项卡的屏幕截图](media/probes-tab-detail.png)

1. “探测”选项卡按钮，用于打开此页面
2. 有关需要使用此页面执行的操作的简短说明
3. 使用它们可选择粗糙或精细分辨率。 粗糙分辨率的速度更快，但需要进行某种权衡。 请参阅下面的[烘培分辨率](bake-resolution.md)了解详细信息。
4. 使用此字段，选择应该放置声学数据文件的位置。 单击具有“...”的按钮以使用文件夹选取器。 默认值为 Assets/AcousticsData。 此外，还将在此位置下创建“Editor”子文件夹。 要详细了解数据文件，请参阅下面的[数据文件](#Data-Files)。
5. 系统使用此处提供的前缀命名该场景的数据文件。 默认值为“Acoustics_[Scene Name]”。
6. 计算完探测后，将禁用上述控件。 单击“清除”按钮，清除计算并启用控件，以便可以使用新设置重新计算。
7. 单击“计算...”按钮可体素化场景并计算探测点位置。 此操作在计算机本地执行，并且必须在进行制作前完成。

在此版本的 Project Acoustics 中，不能手动放置探测，必须通过“探测”选项卡中提供的自动化进程进行放置。

请参阅[烘培分辨率](bake-resolution.md)了解有关粗糙分辨率与精细分辨率的更多详细信息。

## <a name="bake-your-scene-using-azure-batch"></a>使用 Azure Batch 烘焙场景
可借助 Azure Batch 服务使用云端计算群集来烘焙场景。 项目音响效果 Unity 插件会直接连接到 Azure Batch，为每个烘焙实例化、管理和关闭一个 Azure Batch 群集。 在“烘焙”选项卡上，输入你的 Azure 凭据，选择群集计算机类型和大小，然后单击“烘焙”。

### <a name="for-reference-parts-of-the-bake-tab"></a>供参考：“烘焙”选项卡的部件
![Unity 中“音效烘焙”选项卡的屏幕截图](media/bake-tab-details.png)

1. “制作”选项卡，按钮用于打开此页面。
2. 有关要在此页上所执行操作的简短说明。
3. 创建 Azure 帐户后，用于输入 Azure 凭据的字段。 有关详细信息，请参阅[创建 Azure Batch 帐户](create-azure-account.md)。
4. 音效工具集的 Docker 图像标记。
5. 启动 Azure 门户以管理你的订阅，监视使用情况和查看计费信息等。 
6. Azure Batch 计算用于计算的节点类型。 节点类型必须受 Azure 数据中心位置支持。 如果不确定，则保留 Standard_F8s_v2。
7. 要用于此计算的节点数。 此处输入的数字将影响完成制作的时间，并受 Azure Batch 核心分配限制。 默认分配仅允许使用两个 8 核节点或一个 16 核节点，但可以进行扩展。 有关核心分配约束的详细信息，请参阅[创建 Azure Batch 帐户](create-azure-account.md)。
8. 选择此复选框可将你的计算机池配置为使用[低优先级节点](https://docs.microsoft.com/azure/batch/batch-low-pri-vms)。 低优先级计算节点的成本较低，但它们可能并非一直可用或可能随时被抢占。
9. 场景的探测计数在“探测”选项卡上计算得出。探测数决定需要在云中运行的模拟次数。 不能指定多于探测数的节点数。
10. 作业在云端运行所预计耗费的运行时间。 这不包括节点启动时间。 作业开始运行后，这是返回结果前需要等待的时间。 请注意，这只是估计值。
11. 运行模拟所需的总计算时间。 这是 Azure 中将使用的总的节点计算时间。 有关如何使用该值的详细信息，请参阅[估计制作成本](#Estimating-bake-cost)。
12. 此消息告知了作业完成后制作结果保存的位置。
13. （仅限高级使用）如果出于某种原因需要强制 Unity 忘记已提交的制作（例如使用其他计算机下载结果），请单击“清除状态”按钮，以忘记已提交的作业。 请注意，这表示准备就绪后，将不下载结果文件，且这不同于取消作业。 如果运行，该作业将继续在云中运行。
14. 单击“烘焙”按钮，将烘焙提交到云端。 运行作业时，这将改为显示“取消作业”。
15. 准备在[电脑上处理音效模拟](#Local-bake)。
16. 此区域显示制作的状态。 完成后，它应显示“已下载”。

随时可在 [Azure 门户](https://portal.azure.com)获取关于有效作业、计算池和存储的完整信息。

作业运行时，“制作”按钮将变为“取消作业”。 使用此按钮可取消正在进行中的作业。 取消作业前需要先进行确认。 取消作业无法撤消，不会显示任何结果，但用户仍需要为使用的任何 Azure 计算时间付费。

开始制作后，即可以关闭 Unity。 云制作可能需要数个小时，具体取决于项目、节点类型和节点数。 重新加载该项目并打开“音效”窗口时，制作作业状态将更新。 如果作业已完成，将下载输出文件。

Azure 凭据安全地存储在本地计算机上，并与 Unity 编辑器相关联。 它们仅用于建立与 Azure 的安全连接。

### <a name="Estimating-bake-cost"></a> 估计 Azure 制作成本

要估计给定制作的成本，请采用“估计的计算成本”中显示的值（一段持续时间），再乘以所选 VM 节点类型的每小时费用（当地货币）。 结果不包括节点启动和运行所需的节点时间。 例如，如果为节点类型选择 Standard_F8s_v2，其费用为 $0.40/小时，而“估计的计算成本”为 3 小时 57 分钟，则运行作业的估计成本为 $0.40 * ~4 小时 = ~$1.60。 实际成本很可能略高，因为开始使用节点时会产生的额外时间。 在 [Azure Batch 定价](https://azure.microsoft.com/pricing/details/virtual-machines/linux)页可找到每小时的节点费用（对类别选择“优化计算”或“高性能计算”）。

## <a name="Local-bake"></a>在电脑上烘焙场景
可在自己的电脑上烘焙场景。 在创建 Azure Batch 帐户之前使用小场景进行音效试验时，这非常有用。 请注意，音效模拟可能耗时很长，具体取决于场景的大小。

### <a name="minimum-hardware-requirements"></a>最低硬件要求
* 至少 8 核且带 32 GB RAM 的 x86-64 处理器

例如，在具有 Intel Xeon E5-1660 @ 3 GHz 和 32 GB RAM 的 8 核计算机上测试
* 对于探测 100 次的小场景，低分辨率烘焙时约耗时 2 小时，高分辨率烘焙时约耗时 32 小时。
* 对于探测 1000 次的小场景，低分辨率烘焙时约耗时 20 小时，高分辨率烘焙时约耗时 21 天。

### <a name="setup-docker"></a>安装 Docker
在电脑上安装并配置 Docker 将处理模拟 -
1. 安装 [Docker 工具集](https://www.docker.com/products/docker-desktop)。
2. 启动 Docker 设置、导航到“高级”选项，再将资源配置为至少具有 8 GB RAM。 可向 Docker 分配的 CPU 越多，制作完成的速度就越快。 ![示例 Docker 设置的屏幕截图](media/docker-settings.png)
3. 导航到“共享驱动器”并启用用于处理的驱动器共享。![Docker 共享驱动器选项的屏幕截图](media/docker-shared-drives.png)

### <a name="run-local-bake"></a>运行本地制作
1. 单击“烘焙”选项卡上的“准备本地烘焙”按钮，再选择要保存输入文件和执行脚本的文件夹。 然后，通过将该文件夹复制到该计算机，即可在任何计算机上运行制作，只要该计算机满足最低硬件要求并安装了 Docker。
2. 使用“runlocalbake.bat”脚本启动模拟。 此脚本将提取 Project Acoustics Docker 映像（其中包含用于模拟处理的工具集）并启动模拟。 
3. 模拟完成后，将生成的 .ace 文件复制回你的 Unity 项目。 为确保 Unity 将此识别为二进制文件，请将“.bytes”追加到文件扩展名（例如，“Scene1.ace.bytes”）。 有关模拟的详细日志存储在“AcousticsLog.txt”中。 如果遇到任何问题，请共享此文件以协助进行诊断。

## <a name="Data-Files"></a>由烘焙过程添加的数据文件

有 4 个数据文件是在烘焙过程中创建的。 一个包含模拟结果并按你的标题进行传输。 其他文件存储与 Unity 编辑器相关的数据。

模拟结果：
* **Assets/AcousticsData/Acoustics\_[SceneName].ace.bytes**：这是运行时查找表，它包含模拟结果和体素化的音效场景元素。 使用“探测”选项卡上的字段可更改此文件的位置和名称。

请注意不要删除模拟结果文件。 该文件不可恢复，除非通过重新烘焙场景。

编辑器数据文件：
* **Assets/Editor/[SceneName]\_AcousticsParameters.asset**：此文件存储你在音效 UI 的字段中输入的数据。 不能更改此文件的位置和名称。
* **Assets/AcousticsData/Editor/Acoustics_[SceneName].vox**：该文件存储像素化的音效几何体以及通过“探测”选项卡上的“计算...”按钮计算得出的材料属性。使用“探测”选项卡上的字段可更改此文件的位置和名称。
* **Assets/AcousticsData/Editor/Acoustics\_[SceneName]\_config.xml**：该文件存储使用“探测”选项卡上的“计算...”按钮计算得出的模拟参数。使用“探测”选项卡上的字段可更改此文件的位置和名称。

## <a name="set-up-the-acoustics-lookup-table"></a>设置音效查找表
将项目音响效果预制件从项目面板拖放到场景中：

![Unity 中音效预设的屏幕截图](media/acoustics-prefab.png)

单击 ProjectAcoustics 游戏对象，然后转到其检查器面板。 通过将制做结果拖放到音效管理器脚本或单击文本框旁边的圆形按钮，指定制作结果的位置（Assets/AcousticsData 中的 .ACE 文件）。

![Unity 中音效管理器预设的屏幕截图](media/acoustics-manager.png)  

## <a name="next-steps"></a>后续步骤
* 了解 [Unity 设计控件](unity-workflow.md)
* 了解[项目音响效果设计概念](design-process.md)

