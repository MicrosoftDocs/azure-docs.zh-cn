---
title: 快速入门：获取受支持的语言的列表，Python - 文本翻译 API
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，请使用文本翻译 API 和 Python 获取翻译、音译和字典查找支持的语言列表以及示例。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: quickstart
ms.date: 02/21/2019
ms.author: erhopf
ms.openlocfilehash: 0b006f3860c1b00dc8ef0b0fa4d77e1fd9034a17
ms.sourcegitcommit: 2028fc790f1d265dc96cf12d1ee9f1437955ad87
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2019
ms.locfileid: "64922269"
---
# <a name="quickstart-use-the-translator-text-api-to-get-a-list-of-supported-languages-using-python"></a>快速入门：使用 Python 通过文本翻译 API 获取所支持语言的列表

本快速入门介绍如何使用 Python 和文本翻译 REST API 发出可以返回所支持语言的列表的 GET 请求。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* Python 2.7.x 或 3.x

## <a name="create-a-project-and-import-required-modules"></a>创建一个项目并导入必需的模块

使用最喜欢的 IDE 或编辑器创建新的 Python 项目。 然后，将此代码片段复制到项目的名为 `get-languages.py` 的文件中。

```python
# -*- coding: utf-8 -*-
import os, requests, uuid, json
```

> [!NOTE]
> 如果尚未使用这些模块，则需在运行程序之前安装它们。 若要安装这些包，请运行 `pip install requests uuid`。

第一个注释告知 Python 解释器使用 UTF-8 编码。 然后导入必需的模块，以便从环境变量读取订阅密钥、构造 HTTP 请求、创建唯一标识符，以及处理文本翻译 API 返回的 JSON 响应。

## <a name="set-the-base-url-and-path"></a>设置基 URL 和路径

文本翻译全局终结点设置为 `base_url`。 `path` 设置 `languages` 路由并确定我们需使用 API 的版本 3。

>[!NOTE]
> 有关终结点、路由和请求参数的详细信息，请参阅[文本翻译 API 3.0：语言](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-languages)。

```python
base_url = 'https://api.cognitive.microsofttranslator.com'
path = '/languages?api-version=3.0'
constructed_url = base_url + path
```

## <a name="add-headers"></a>添加标头

请求获取支持的语言不需身份验证。 将 `Content-type` 设置为 `application/json` 并添加 `X-ClientTraceId`，以便对请求进行唯一标识。

将以下代码片段复制到项目中。

```python
headers = {
    'Content-type': 'application/json',
    'X-ClientTraceId': str(uuid.uuid4())
}
```

## <a name="create-a-request-to-get-a-list-of-supported-languages"></a>创建一个请求，以获取支持的语言的列表

请使用 `requests` 模块创建 GET 请求。 它使用两个参数：串联的 URL 和请求标头：

```python
request = requests.get(constructed_url, headers=headers)
response = request.json()
```

## <a name="print-the-response"></a>输出响应

最后一步是输出结果。 以下代码片段通过将密钥排序、设置缩进以及声明项和密钥分隔符来美化结果。

```python
print(json.dumps(response, sort_keys=True, indent=4, ensure_ascii=False, separators=(',', ': ')))
```

## <a name="put-it-all-together"></a>将其放在一起

就是这样，你已构建了一个简单的程序。该程序可以调用文本翻译 API 并返回 JSON 响应。 现在，可以运行该程序了：

```console
python get-languages.py
```

如果希望将你的代码与我们的进行比较，请查看 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Python) 上提供的完整示例。

## <a name="sample-response"></a>示例响应

请在此[语言列表](https://docs.microsoft.com/azure/cognitive-services/translator/language-support)中查找国家/地区缩写。

此示例已截断，显示了结果的片段：

```json
{
    "translation": {
        "af": {
            "name": "Afrikaans",
            "nativeName": "Afrikaans",
            "dir": "ltr"
        },
        "ar": {
            "name": "Arabic",
            "nativeName": "العربية",
            "dir": "rtl"
        },
        ...
    },
    "transliteration": {
        "ar": {
            "name": "Arabic",
            "nativeName": "العربية",
            "scripts": [
                {
                    "code": "Arab",
                    "name": "Arabic",
                    "nativeName": "العربية",
                    "dir": "rtl",
                    "toScripts": [
                        {
                            "code": "Latn",
                            "name": "Latin",
                            "nativeName": "اللاتينية",
                            "dir": "ltr"
                        }
                    ]
                },
                {
                    "code": "Latn",
                    "name": "Latin",
                    "nativeName": "اللاتينية",
                    "dir": "ltr",
                    "toScripts": [
                        {
                            "code": "Arab",
                            "name": "Arabic",
                            "nativeName": "العربية",
                            "dir": "rtl"
                        }
                    ]
                }
            ]
        },
      ...
    },
    "dictionary": {
        "af": {
            "name": "Afrikaans",
            "nativeName": "Afrikaans",
            "dir": "ltr",
            "translations": [
                {
                    "name": "English",
                    "nativeName": "English",
                    "dir": "ltr",
                    "code": "en"
                }
            ]
        },
        "ar": {
            "name": "Arabic",
            "nativeName": "العربية",
            "dir": "rtl",
            "translations": [
                {
                    "name": "English",
                    "nativeName": "English",
                    "dir": "ltr",
                    "code": "en"
                }
            ]
        },
      ...
    }
}
```

## <a name="clean-up-resources"></a>清理资源

如果已将订阅密钥硬编码到程序中，请确保在完成本快速入门后删除该订阅密钥。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 Python 示例](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Python)

## <a name="see-also"></a>另请参阅

了解如何使用文本翻译 API 执行以下操作：

* [翻译文本](quickstart-python-translate.md)
* [直译文本](quickstart-python-transliterate.md)
* [按输入确定语言](quickstart-python-detect.md)
* [获取备用翻译](quickstart-python-dictionary.md)
* [根据输入确定句子长度](quickstart-python-sentences.md)
