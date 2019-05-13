---
title: 如何有效地使用 Azure Maps 搜索服务搜索 |Microsoft Docs
description: 了解如何使用 Azure Maps 搜索服务搜索使用最佳方案
ms.author: v-musehg
ms.date: 04/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 9cb0f89b4a48d7139adb35dcef48c0115b005c57
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65205607"
---
# <a name="best-practices-to-use-azure-maps-search-service"></a>使用 Azure 地图搜索服务的最佳做法

Azure Maps[搜索服务](https://docs.microsoft.com/rest/api/maps/search)包含使用各种功能，例如中的 Api 的搜索兴趣点 (POI) 数据以某一位置的地址搜索。 在本文中，我们将分享调用通过 Azure Maps 搜索服务的数据的最佳做法。 可以了解如何执行以下操作：

* 生成查询，以返回相关的匹配项
* 限制搜索结果
* 了解不同结果类型之间的差异
* 读取地址搜索响应结构


## <a name="prerequisites"></a>必备组件

若要调用任何 Maps 服务 API，需要具有 Maps 帐户和密钥。 有关创建帐户和检索密钥的信息，请参阅[如何管理 Azure Maps 帐户和密钥](how-to-manage-account-keys.md)。

> [!Tip]
> 若要查询的搜索服务，可以使用[Postman 应用](https://www.getpostman.com/apps)来生成 REST 调用，也可以使用任何您喜欢的 API 开发环境。


## <a name="best-practices-for-geocoding"></a>地理编码的最佳做法

在搜索使用 Azure Maps 搜索服务的完整或部分地址时，它采用搜索词，并返回地址的经度和纬度坐标。 此过程称为地理编码。 在国家/地区中进行地理编码的能力取决于道路数据覆盖范围和地理编码服务的地理编码精度。

请参阅[地理编码覆盖区域](https://docs.microsoft.com/azure/azure-maps/geocoding-coverage)按国家/地区中了解有关 Azure Maps 地理编码功能的详细信息。

### <a name="limit-search-results"></a>限制搜索结果

   在本部分中，将了解如何使用 Azure Maps 搜索 Api 限制搜索结果。 

   > [!Note]
   > 并非所有搜索 Api 完全都支持下面列出的参数

   **地理偏置搜索结果**

   地理偏置到您的结果与你的用户的相关区域您应始终添加详细的最大可能位置输入。 若要限制搜索结果，请考虑添加以下输入的类型：

   1. 设置`countrySet`参数，例如"美国，FR"。 默认搜索行为是搜索整个世界，可能会返回不必要的结果。 如果您的查询不包括`countrySet`参数，搜索可能返回不准确的结果。 例如，搜索名为的城市**贝尔维尤**将返回结果来自美国和法国，由于没有名为城市**贝尔维尤**法国和美国境内。

   2. 可以使用`btmRight`和`topleft`参数来设置边界框以将搜索限制在地图上的特定区域。

   3. 若要影响的区域关系的结果，可以定义`lat`并`lon`协调参数和设置的搜索区域使用 radius`radius`参数。


   **模糊搜索参数**

   1. `minFuzzyLevel`和`maxFuzzyLevel`，帮助返回相关的匹配项，即使查询参数并不完全对应于所需的信息。 大多数搜索查询默认`minFuzzyLevel=1`和`maxFuzzyLevel=2`以提高性能并减少不正常的结果。 需要"restrant"的搜索词的示例，它当匹配到"餐馆"`maxFuzzyLevel`设置为 2。 根据请求的需求，可以重写默认模糊级别。 

   2. 此外可以指定要使用返回的结果类型的确切一`idxSet`参数。 为此可以提交的索引以逗号分隔列表，项顺序并不重要。 以下是受支持的索引：

       * `Addr` - **地址范围**:对于某些街道有开头和结尾街道; 的内插的地址点这些点表示为地址范围。
       * `Geo` - **地理区域**:它是表示管理部门的土地的地图、 国家/地区、 状态、 城市的区域。
       * `PAD` - **点地址**:街道名称和编号与特定地址所在在索引中，例如，Soquel Dr 2501 在地图上的点。 它是准确性的最高级别的可用地址。  
       * `POI` - **兴趣点**:值得注意，可能感兴趣点，在地图上。  [获取搜索地址](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress)不会返回 Poi。  
       * `Str` - **街道**:在地图上的街道的表示形式。
       * `XStr` - **跨街道/交集**:表示形式的交接点;两个街道相交的位置。


       **用法示例**:

       * idxSet = POI （只搜索兴趣点） 

       * idxSet = 键盘，Addr (搜索仅，解决了板 = 点地址，Addr = 地址范围)

### <a name="reverse-geocode-and-geography-entity-type-filter"></a>反向地理编码和地理实体类型筛选器

执行与反向地理编码搜索时[搜索地址反向 API](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse)，该服务有能力，可以返回的管理区域的多边形。 只需提供参数`entityType`在请求中，您可以缩小搜索范围的指定的地理实体类型。 生成的响应将包含 geography ID，以及匹配的实体类型。 如果提供多个实体，终结点将返回**可用的最小实体**。 返回几何图形 ID 可用于获取通过该地理几何图形[获取多边形服务](https://docs.microsoft.com/rest/api/maps/search/getsearchpolygon)。

**示例请求：**

```HTTP
https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={subscription-key}&query=47.6394532,-122.1304551&language=en-US&entityType=Municipality
```

**响应：**

```JSON
{
    "summary": {
        "queryTime": 8,
        "numResults": 1
    },
    "addresses": [
        {
            "address": {
                "routeNumbers": [],
                "countryCode": "US",
                "countrySubdivision": "WA",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "municipality": "Redmond",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Redmond, WA",
                "boundingBox": {
                    "northEast": "47.717105,-122.034537",
                    "southWest": "47.627016,-122.164998",
                    "entity": "position"
                },
                "countrySubdivisionName": "Washington"
            },
            "position": "47.639454,-122.130455",
            "dataSources": {
                "geometry": {
                    "id": "00005557-4100-3c00-0000-0000596ae571"
                }
            },
            "entityType": "Municipality"
        }
    ]
}
```

### <a name="search-results-language"></a>搜索结果语言

`language`参数，可设置应在哪些语言搜索中返回结果。 如果未在请求中设置语言，搜索服务将自动默认为国家/地区中最常用的语言。 此外，当指定的语言中的数据不可用，则使用默认语言。 请参阅[支持的语言](https://docs.microsoft.com/azure/azure-maps/supported-languages)有关相对于按国家/地区的 Azure Maps 服务支持的语言的列表。


### <a name="predictive-mode-auto-suggest"></a>预测模式 （自动建议）

若要查找更多匹配项的部分查询`typeahead`参数应设置为 'true'。 该查询将被解释为部分输入和搜索将在进入预测的模式。 否则该服务将假定传入的所有相关信息。

在此示例下面的查询可以看到"Microso"中，查询搜索地址服务与`typeahead`参数设置为**true**。 如果发现响应，可以看到，搜索服务解释作为部分查询的查询和响应包含的自动建议查询的结果。

**示例查询：**

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1.0&typeahead=true&countrySet=US&lat=47.6370891183&lon=-122.123736172&query=Microsoft
```

**响应：**

```JSON
{
    "summary": {
        "query": "microsoft",
        "queryType": "NON_NEAR",
        "queryTime": 25,
        "numResults": 6,
        "offset": 0,
        "totalResults": 6,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6370891183,
            "lon": -122.123736172
        }
    },
    "results": [
        {
            "type": "Street",
            "id": "US/STR/p0/10294417",
            "score": 2.594,
            "dist": 327.546040632591,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "postalCode": "98052",
                "extendedPostalCode": "980526399,980528300",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Way, Redmond, WA 98052",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.63989,
                "lon": -122.12509
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63748,
                    "lon": -122.12309
                },
                "btmRightPoint": {
                    "lat": 47.64223,
                    "lon": -122.13061
                }
            }
        },
        ...,
        ...,
        ...,
        ...,
        {
            "type": "Street",
            "id": "US/STR/p0/9063400",
            "score": 2.075,
            "dist": 3655467.6406921702,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Yorkmount, Charlotte",
                "municipality": "Charlotte",
                "countrySecondarySubdivision": "Mecklenburg",
                "countryTertiarySubdivision": "Township 1 Charlotte",
                "countrySubdivision": "NC",
                "postalCode": "28217",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Way, Charlotte, NC 28217",
                "countrySubdivisionName": "North Carolina"
            },
            "position": {
                "lat": 35.14279,
                "lon": -80.91814
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14267,
                    "lon": -80.91814
                },
                "btmRightPoint": {
                    "lat": 35.14279,
                    "lon": -80.91824
                }
            }
        }
    ]
}
```


### <a name="uri-encoding-to-handle-special-characters"></a>URI 编码来处理特殊字符 

若要查找交叉街道地址，即"第一种途径 & 联合 Street，西雅图"、 特殊字符 & 发送请求之前要编码的需求。 我们建议编码的 URI 中的字符数据使用 %字符进行编码的所有字符的位置以及为其 utf-8 字符相对应的两个字符十六进制值。

**用法示例**:

获取搜索地址：

```
query=1st Avenue & E 111th St, New York
```

 应编码为：

```
query"=1st%20Avenue%20%26%20E%20111th%20St%2C%20New%20York
```


以下是要用于不同的语言的不同方法： 

JavaScript/TypeScript:
```Javascript
encodeURIComponent(query)
```

C#/ VB:
```C#
Uri.EscapeDataString(query)
```

Java:
```Java
URLEncoder.encode(query, "UTF-8") 
```

Python:
```Python
import urllib.parse 
urllib.parse.quote(query)
```

C++：
```C++
#include <curl/curl.h>
curl_easy_escape(query)
```

PHP:
```PHP
urlencode(query)
```

Ruby:
```Ruby
CGI::escape(query) 
```

Swift:
```Swift
query.stringByAddingPercentEncodingWithAllowedCharacters(.URLHostAllowedCharacterSet()) 
```

转到：
```Go
import ("net/url") 
url.QueryEscape(query)
```


## <a name="best-practices-for-poi-search"></a>POI 搜索最佳实践

感兴趣 (POI) 搜索的点，可按名称，例如，按名称搜索业务请求 POI 结果。 我们强烈建议你使用`countrySet`参数来指定国家/地区，应用程序需要覆盖，因为默认行为将搜索整个世界，可能会返回不必要的结果和/或导致更长的搜索时间。

### <a name="brand-search"></a>品牌搜索

若要提高相关性的结果以及在响应中的信息，感兴趣点 (POI) 搜索响应包括可用于进一步分析的响应的品牌信息。

让我们[POI 类别搜索](https://docs.microsoft.com/rest/api/maps/search/getsearchpoicategory)加油站附近 Microsoft 园区 （华盛顿州雷德蒙德） 的请求。 如果发现响应时，可以看到返回每个 POI 的品牌信息。

**示例查询：**

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=gas%20station&limit=3&lat=47.6413362&lon=-122.1327968
```

**响应：**

```JSON
{
    "summary": {
        "query": "gas station",
        "queryType": "NON_NEAR",
        "queryTime": 206,
        "numResults": 3,
        "offset": 0,
        "totalResults": 742169,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6413362,
            "lon": -122.1327968
        }
    },
    "results": [
        {
            "type": "POI",
            "id": "US/POI/p0/245813",
            "score": 5.663,
            "dist": 1037.0280221303253,
            "info": "search:ta:840531000004190-US",
            "poi": {
                "name": "Chevron",
                "phone": "+(1)-(425)-6532200",
                "brands": [
                    {
                        "name": "Chevron"
                    }
                ],
                "url": "www.chevron.com",
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2444",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Northeast Bellevue, Bellevue",
                "municipality": "Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "2444 Bel Red Rd, Bellevue, WA 98007",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.63201,
                "lon": -122.13281
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63291,
                    "lon": -122.13414
                },
                "btmRightPoint": {
                    "lat": 47.63111,
                    "lon": -122.13148
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63223,
                        "lon": -122.13311
                    }
                }
            ]
        },
        ...,
        {
            "type": "POI",
            "id": "US/POI/p0/7727106",
            "score": 5.662,
            "dist": 1458.645407416307,
            "info": "search:ta:840539000488527-US",
            "poi": {
                "name": "BROWN BEAR CAR WASH",
                "phone": "+(1)-(425)-6442868",
                "brands": [
                    {
                        "name": "Texaco"
                    }
                ],
                "url": "www.texaco.com/",
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "15248",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "postalCode": "98052",
                "extendedPostalCode": "980525511",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "15248 Bel Red Rd, Redmond, WA 98052",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.62843,
                "lon": -122.13628
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62933,
                    "lon": -122.13761
                },
                "btmRightPoint": {
                    "lat": 47.62753,
                    "lon": -122.13495
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.62826,
                        "lon": -122.13626
                    }
                }
            ]
        }
    ]
}
```


### <a name="airport-search"></a>机场搜索

POI 搜索支持通过使用正式的机场代码搜索机场。 例如， **SEA** （西雅图塔科马国际机场）。 

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=SEA 
```

### <a name="nearby-search"></a>邻近搜索

若要检索仅 POI 结果以某一位置[附近的搜索 API](https://docs.microsoft.com/rest/api/maps/search/getsearchnearby)可能是正确的选择。 此终结点将仅返回 POI 结果，而不使用搜索查询参数中。 若要限制结果，建议将半径设置。

## <a name="understanding-the-responses"></a>了解响应

让我们将地址搜索请求向 Azure Maps [search 服务](https://docs.microsoft.com/rest/api/maps/search)西雅图中的地址。 如果您仔细查看下面的请求 URL 中，我们将设置`countrySet`参数**美国**搜索美国中的地址。

**示例查询：**

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1&query=400%20Broad%20Street%2C%20Seattle%2C%20WA&countrySet=US
```

进一步让我们看下面的响应结构。 在响应中的结果对象的结果类型是不同的。 如果仔细观察你可以看到我们有三种不同类型的结果对象，它是"点地址"、"Street"和"跨 Street"。 请注意，该地址搜索未返回 Poi。 `Score`为每个响应对象的参数指示相对的匹配分数对同一响应中其他对象的分数。 请参阅[获取搜索地址](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress)若要了解有关响应对象参数的详细信息。

**支持的结果类型：**

* **点地址：** 具有街道名称和编号与特定地址的映射的点。 可用地址的准确性最高级别。 

* **地址范围：** 对于某些街道有开头和结尾街道; 的内插的地址点这些点表示为地址范围。 

* **地理位置：** 它是表示管理部门的土地的地图、 国家/地区、 状态、 城市的区域。 

* **POI-（感兴趣的点）：** 值得注意，可能感兴趣点，在地图上。

* **街道：** 在地图上的街道的表示形式。 地址将解析为包含地址的街道名的纬度/经度坐标。 门牌号码可能不会处理。 

* **十字路口：** 交集。 交接点; 的表示形式两个街道相交的位置。

**响应：**

```JSON
{
    "summary": {
        "query": "400 broad street seattle wa",
        "queryType": "NON_NEAR",
        "queryTime": 129,
        "numResults": 6,
        "offset": 0,
        "totalResults": 6,
        "fuzzyLevel": 1
    },
    "results": [
        {
            "type": "Point Address",
            "id": "US/PAD/p0/43076024",
            "score": 9.894,
            "address": {
                "streetNumber": "400",
                "streetName": "Broad Street",
                "municipalitySubdivision": "Seattle, South Lake Union, Lower Queen Anne",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "postalCode": "98109",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "400 Broad Street, Seattle, WA 98109",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.62039,
                "lon": -122.34928
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62129,
                    "lon": -122.35061
                },
                "btmRightPoint": {
                    "lat": 47.61949,
                    "lon": -122.34795
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.61982,
                        "lon": -122.34886
                    }
                }
            ]
        },
        {
            "type": "Street",
            "id": "US/STR/p0/2440854",
            "score": 8.129,
            "address": {
                "streetName": "Broad Street",
                "municipalitySubdivision": "Seattle, Westlake, South Lake Union",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "postalCode": "98109",
                "extendedPostalCode": "981094347,981094700,981094701,981094702",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "Broad Street, Seattle, WA 98109",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.62553,
                "lon": -122.33936
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62545,
                    "lon": -122.33861
                },
                "btmRightPoint": {
                    "lat": 47.62574,
                    "lon": -122.33974
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/8450985",
            "score": 8.129,
            "address": {
                "streetName": "Broad Street",
                "municipalitySubdivision": "Seattle, Belltown",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "postalCode": "98109,98121",
                "extendedPostalCode": "981094991,981211117,981211237,981213206",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "Broad Street, Seattle, WA",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.61691,
                "lon": -122.35251
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61502,
                    "lon": -122.35041
                },
                "btmRightPoint": {
                    "lat": 47.61857,
                    "lon": -122.35484
                }
            }
        },
        ...,
        ...,
        {
            "type": "Cross Street",
            "id": "US/XSTR/p1/3816818",
            "score": 6.759,
            "address": {
                "streetName": "Broad Street & Valley Street",
                "municipalitySubdivision": "South Lake Union, Seattle",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "postalCode": "98109",
                "countryCode": "US",
                "country": "United States Of America",
                "countryCodeISO3": "USA",
                "freeformAddress": "Broad Street & Valley Street, Seattle, WA 98109",
                "countrySubdivisionName": "Washington"
            },
            "position": {
                "lat": 47.62574,
                "lon": -122.33861
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62664,
                    "lon": -122.33994
                },
                "btmRightPoint": {
                    "lat": 47.62484,
                    "lon": -122.33728
                }
            }
        }
    ]
}
```

### <a name="geometry"></a>几何图形

当响应类型是**几何图形**，它可以包括在返回的 geometry ID**数据源**"geometry"和"id"下的对象。 例如，[获取多边形服务](https://docs.microsoft.com/rest/api/maps/search/getsearchpolygon)可以请求以 GeoJSON 格式，如一组实体的城市或机场轮廓的几何图形数据。 可以使用此边界数据用于[地理围栏](https://docs.microsoft.com/azure/azure-maps/tutorial-geofence)或[的几何图形内搜索 Poi](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry)。


[搜索地址](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress)或[模糊搜索](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy)API 响应可以包括**geometry ID** "geometry"和"id"下的数据源对象中返回。


```JSON 
"dataSources": { 
        "geometry": { 
            "id": "00005557-4100-3c00-0000-000059690938" // The geometry ID is returned in the dataSources object under "geometry" and "id".
        }
} 
```

## <a name="next-steps"></a>后续步骤

* 了解[如何构建 Azure Maps 搜索服务请求](https://docs.microsoft.com/azure/azure-maps/how-to-search-for-address)。
* 探索 Azure Maps[搜索服务 API 文档](https://docs.microsoft.com/rest/api/maps/search)。 