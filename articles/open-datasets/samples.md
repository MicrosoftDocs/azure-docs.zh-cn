---
title: 使用 NOAA 开放数据集的示例 Jupyter 笔记本
titleSuffix: Azure Open Datasets
description: 通过使用 Azure 开放数据集的示例 Jupyter 笔记本，了解如何加载开放数据集，并使用它们来扩充演示数据。 所涉及的技术包括使用 Spark 和 Pandas 来处理数据。
ms.service: open-datasets
ms.topic: sample
author: cjgronlund
ms.author: cgronlun
ms.date: 05/02/2019
ms.openlocfilehash: b62a2690e5879e45a14d0b06a38e8c5171dda14e
ms.sourcegitcommit: 4891f404c1816ebd247467a12d7789b9a38cee7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65442335"
---
# <a name="example-jupyter-notebooks-show-how-to-enrich-data-with-open-datasets"></a>展示如何使用开放数据集来扩充数据的示例 Jupyter 笔记本 
使用 Azure 开放数据集的示例 Jupyter 笔记本展示了如何加载开放数据集，并使用它们来扩充演示数据。 所涉及的技术包括使用 Apache Spark 和 Pandas 来处理数据。

>[!IMPORTANT]
>如果是在非 Spark 环境中，开放数据集仅允许一次下载一个月的数据，并利用一些类来避免较大数据集出现 MemoryError。

## <a name="load-noaa-integrated-surface-database-isd-data"></a>加载 NOAA 集成地面数据库 (ISD) 数据 
|Notebook        | 说明                                    |
|----------------|------------------------------------------------|
|[将最近一个月的天气数据加载到 Pandas 数据帧](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-access/02-weather-to-pandas-dataframe.ipynb) | 了解如何将历史天气数据加载到最常用的 Pandas 数据帧。 |
|[将最近一个月的天气数据加载到 Spark 数据帧](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-access/01-weather-to-spark-dataframe.ipynb) | 了解如何将历史天气数据加载到最常用的 Spark 数据帧。  |

## <a name="join-demo-data-with-noaa-isd-data"></a>联接演示数据和 NOAA ISD 数据 
|Notebook        | 说明                                    |
|----------------|------------------------------------------------|
|[联接演示数据和天气数据 - Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/02-weather-join-in-pandas.ipynb) | 在 Pandas 数据帧中，联接 1 个月的传感器位置演示数据集与天气读数。  |
|[联接演示数据和天气数据 - Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/01-weather-join-in-spark.ipynb) | 在 Spark 数据帧中，联接传感器位置演示数据集与天气读数。 |

## <a name="join-nyc-taxi-data-with-noaa-isd-data"></a>联接纽约出租车数据与 NOAA ISD 数据 
|Notebook        | 说明                                    |
|----------------|------------------------------------------------|
|[扩充了天气数据的出租车车程数据 - Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/04-nyc-taxi-join-weather-in-pandas.ipynb) | 加载纽约绿色出租车数据（1 个月内），并在 Pandas 数据帧中使用天气数据扩充它。 此示例重写方法 `get_pandas_limit`，并平衡数据加载性能与数据量。|
|[扩充了天气数据的出租车车程数据 - Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/03-nyc-taxi-join-weather-in-spark.ipynb) | 加载纽约绿色出租车数据，并在 Spark 数据帧中使用天气数据扩充它。  |

## <a name="next-steps"></a>后续步骤

* [教程：使用自动化机器学习和开放数据集的回归建模](tutorial-opendatasets-automl.md)
* [用于开放数据集的 Python SDK](https://aka.ms/open-datasets-api)
* [Azure 开放数据集目录](https://azure.microsoft.com/services/open-datasets/catalog/)
