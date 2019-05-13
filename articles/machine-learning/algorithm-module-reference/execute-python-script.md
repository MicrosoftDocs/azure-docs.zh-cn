---
title: 执行 Python 脚本：模块参考
titleSuffix: Azure Machine Learning service
description: 了解如何使用 Azure 机器学习服务中执行 Python 脚本模块运行 Python 代码。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 05/02/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 6e61ed5a18e69b107b78bf2042de21d14cd6beb5
ms.sourcegitcommit: 4b9c06dad94dfb3a103feb2ee0da5a6202c910cc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65029125"
---
# <a name="execute-python-script-module"></a>执行 Python 脚本模块

本指南介绍了 Azure 机器学习服务的可视界面 （预览版） 的模块。

使用此模块来运行 Python 代码。 有关 Python 的体系结构和设计原则的详细信息，请参阅[以下文章。](https://docs.microsoft.com/azure/machine-learning/machine-learning-execute-python-scripts)

使用 Python，您可以执行目前不支持现有模块如的任务：

+ 可视化数据使用 `matplotlib`
+ 使用 Python 库枚举数据集和工作区中的模型
+ 读取、 加载和操作数据的源不受[导入数据](./import-data.md)模块
+ 运行深度学习代码 


Azure 机器学习使用 Anaconda Python 分发版，其中包括数据处理的多个常见的实用程序。 我们将自动更新 Anaconda 版本。 当前版本为：
 -  为 Python 3.6 anaconda 4.5 + 分发 

预安装的包包括：
-  asn1crypto==0.24.0
- attrs==19.1.0
- azure-common==1.1.18
- azure-storage-blob==1.5.0
- azure-storage-common==1.4.0
- certifi==2019.3.9
- cffi==1.12.2
- chardet==3.0.4
- cryptography==2.6.1
- distro==1.4.0
- idna==2.8
- jsonschema==3.0.1
- lightgbm==2.2.3
- more-itertools==6.0.0
- numpy==1.16.2
- pandas==0.24.2
- Pillow==6.0.0
- pip==19.0.3
- pyarrow==0.12.1
- pycparser==2.19
- pycryptodomex==3.7.3
- pyrsistent==0.14.11
- python-dateutil==2.8.0
- pytz==2018.9
- requests==2.21.0
- scikit-learn==0.20.3
- scipy==1.2.1
- setuptools==40.8.0
- six==1.12.0
- torch==1.0.1.post2
- torchvision==0.2.2.post3
- urllib3==1.24.1
- wheel==0.33.1 

 若要安装其他包不在预安装列表中，例如*scikit 杂项*，将以下代码添加到您的脚本： 

 ```python
import os
os.system(f"pip install scikit-misc")
```

## <a name="how-to-use"></a>如何使用

**执行 Python 脚本**模块包含示例 Python 代码，可以使用作为起始点。 若要配置**执行 Python 脚本**模块，则提供的输入和 Python 代码中执行一组**Python 脚本**文本框。

1. 添加**执行 Python 脚本**模块在试验。

2. 添加和连接上**Dataset1**从你想要使用的输入的接口的任何数据集。 引用 Python 脚本作为此数据集**DataFrame1**。

    如果想要生成使用 Python、 数据或使用 Python 代码来将数据直接导入该模块，是可选的使用的数据集。

    此模块上支持的第二个数据集添加**Dataset2**。 作为 DataFrame2 引用 Python 脚本中的第二个数据集。

    在 Azure 机器学习中存储的数据集自动转换为**pandas**数据框架使用此模块加载时。

    ![执行 Python 输入的映射](media/module/python-module.png)

4. 若要包含新的 Python 包或代码，添加对包含这些自定义资源的压缩的文件**脚本捆绑包**。 输入**脚本捆绑包**必须压缩的文件已上传到你的工作区。 

    试验执行期间，可以使用已上传压缩的存档中包含的任何文件。 如果存档都包括目录结构，保留了结构，但您必须在前面添加一个名为目录**src**到路径。

5. 在中**Python 脚本**文本框中，键入或粘贴有效的 Python 脚本。

    **Python 脚本**文本框中已经填充了一些注释和访问数据和输出的示例代码中的说明。 **您必须编辑或替换此代码。** 请务必遵循 Python 有关缩进和大小写约定。

    + 该脚本必须包含一个名为函数`azureml_main`作为此模块的入口点。
    + 入口点函数可以包含最多两个输入的参数：`Param<dataframe1>`和 `Param<dataframe2>`
    + 解压缩并存储在目录中，连接到第三个输入端口的压缩的文件`.\Script Bundle`，这还将添加到 Python `sys.path`。 

    因此，如果 zip 文件包含`mymodule.py`，使用导入`import mymodule`。

    + 两个数据集可以返回到接口，它必须是一系列类型`pandas.DataFrame`。 可以在 Python 代码中创建其他输出并将其写入到 Azure 存储空间直接。

6. 运行实验，或选择的模块，并单击**运行所选**运行只是 Python 脚本。

    所有数据和代码加载到虚拟机，并使用指定的 Python 环境运行。

## <a name="results"></a>结果

必须作为 pandas 提供的由嵌入式 Python 代码执行任何计算的结果。数据帧，自动转换为 Azure 机器学习数据集格式，以便您可以与其他模块在试验中使用结果。

该模块返回两个数据集：  
  
+ **结果数据集 1**、 定义的 Python 脚本中的第一个返回的 pandas 数据帧

+ **结果数据集 2**、 定义的 Python 脚本中的第二个返回的 pandas 数据帧


## <a name="next-steps"></a>后续步骤

请参阅[可用的模块集](module-reference.md)到 Azure 机器学习服务。 