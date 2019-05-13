---
title: 在搜索框中的 Azure 搜索中添加建议和记忆式键入功能
description: 通过创建建议器和系统地阐明在具有已完成的词或短语的搜索框中填充的请求启用 typeahead Azure 搜索中的查询操作。
manager: pablocas
author: mrcarter8
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: mcarter
ms.custom: seodec2018
ms.openlocfilehash: f3e7cc2ed71dda0eb34328432b26a4016b62be1c
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466423"
---
# <a name="add-suggestions-or-autocomplete-to-your-azure-search-application"></a>将建议或自动完成功能添加到 Azure 搜索应用程序

在本文中，了解如何使用[建议](https://docs.microsoft.com/rest/api/searchservice/suggestions)并[记忆式键入功能](https://docs.microsoft.com/rest/api/searchservice/autocomplete)生成支持搜索---键入时行为的功能强大的搜索框。

+ *建议*建议结果生成在你键入时，其中每条建议是从与已键入内容相匹配的索引的单个结果。 

+ *记忆式键入功能*"完成"的单词或短语当前键入用户。 而不是返回结果，它将完成的查询，然后您可以执行返回的结果。 使用建议，因为已完成的词或短语查询中的都基于索引中的匹配项。 服务不会提供在索引中返回零个结果的查询。

可以下载并运行中的示例代码**DotNetHowToAutocomplete**来评估这些功能。 示例代码以填充的预建的索引为目标[NYCJobs 演示数据](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs)。 NYCJobs 索引包含[建议器构造](index-add-suggesters.md)，这是对使用建议或自动完成功能的要求。 可以使用托管在沙盒服务中，已准备好的索引或[填充自己的索引](#configure-app)NYCJobs 示例解决方案中使用数据加载器。 

**DotNetHowToAutocomplete**示例演示了建议和记忆式键入功能，在这种C#和 JavaScript 语言版本。 C#开发人员可以单步执行使用的基于 ASP.NET MVC 的应用程序[Azure 搜索.NET SDK](https://aka.ms/search-sdk)。 在 HomeController.cs 文件可使记忆式键入功能和建议的查询调用的逻辑。 JavaScript 开发人员会发现在 IndexJavaScript.cshtml，其中包括直接调用等效的查询逻辑[Azure 搜索 REST API](https://docs.microsoft.com/rest/api/searchservice/)。 

对于这两个语言版本中，基于前端用户体验[jQuery UI](https://jqueryui.com/autocomplete/)并[XDSoft](https://xdsoft.net/jqplugins/autocomplete/)库。 我们使用这些库来生成搜索框中支持的建议和记忆式键入功能。 为 HomeController.cs 或 IndexJavaScript.cshtml 中定义的建议和自动完成操作，例如那些与配对收集在搜索框中输入。

本练习将指导你完成以下任务：

> [!div class="checklist"]
> * 实现一个搜索输入的框中 JavaScript 和发出请求的建议匹配项或自动完成条款
> * 在C#，在 HomeController.cs 中定义的建议和自动完成操作
> * 在 JavaScript 中，调用 REST Api 直接来提供相同的功能

## <a name="prerequisites"></a>必备组件

Azure 搜索服务是对于本练习可选的因为该解决方案使用一个实时沙盒服务托管的已准备好的 NYCJobs 演示索引。 如果你想要在搜索服务上运行此示例，请参阅[配置 NYC 作业索引](#configure-app)有关的说明。

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)（版本不限）。 示例代码和说明已在免费社区版上进行了测试。

* 下载[DotNetHowToAutoComplete 示例](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete)。

该示例是全面、 覆盖建议、 记忆式键入功能、 分面导航和客户端缓存。 你应查看自述文件和注释的示例提供的完整说明。

## <a name="run-the-sample"></a>运行示例

1. 打开**AutocompleteTutorial.sln** Visual Studio 中。 该解决方案包含 ASP.NET MVC 项目中使用 NYC 作业演示索引到的连接。

2. 按 F5 运行该项目，并在所选的浏览器中加载页面。

顶部显示了一个用于选择 C# 或 JavaScript 的选项。 C#选项从浏览器连接到了 HomeController 同时使用 Azure 搜索.NET SDK 来检索结果。 

JavaScript 选项直接从浏览器调用 Azure 搜索 REST API。 一般情况下，此选项的性能明显要好得多，因为它会使控制器与流保持隔离。 可以根据自己的需求和语言偏好选择适当的选项。 页面中提供了几个自动完成示例，为每个选项提供了一些指导。 每个示例包含了一些可供尝试的建议示例文本。  

请尝试在每个搜索框中键入几个字母，以查看结果。

## <a name="search-box"></a>搜索框

两个C#和 JavaScript 的版本中，搜索框实现是完全相同。 

打开**Index.cshtml**下文件夹 \Views\Home 若要查看的代码文件：

```html
<input class="searchBox" type="text" id="example1a" placeholder="search">
```

这是具有一个 ID 来引用的 JavaScript 和占位符文本的样式设置的类的简单输入的文本框。  奇妙的事情是嵌入式 JavaScript 中的。

C#语言示例使用 JavaScript 在 Index.cshtml 中利用[jQuery UI 自动填充库](https://jqueryui.com/autocomplete/)。 此库添加到搜索框自动完成体验，从而对 MVC 控制器的异步调用来检索建议。 IndexJavaScript.cshtml 中的 JavaScript 语言版本。 它包括搜索栏中，以及对 Azure 搜索 REST API 调用下面的脚本。

让我们看一下第一个示例中，后者调用 jQuery UI 自动完成函数，在请求中传递的建议的 JavaScript 代码：

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: "/home/suggest?highlights=false&fuzzy=false&",
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

在页面加载配置的 jQuery UI 自动填充"example1a"输入框，在浏览器中运行上面的代码。  `minLength: 3` 确保仅当搜索框中至少有三个字符时，才显示建议。  源值非常重要：

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

上述代码行通知 jQuery UI 自动填充函数获取要在搜索框下显示的项列表的位置。 由于这是 MVC 项目，它在 HomeController.cs 中，其中包含用于返回查询建议 （有关详细信息下一步部分中的建议） 的逻辑调用建议函数。 此函数还将几个参数传递给控件突出显示、 模糊匹配和术语。 自动完成 JavaScript API 会添加字词参数。

### <a name="extending-the-sample-to-support-fuzzy-matching"></a>扩展示例以支持模糊匹配

使用模糊搜索可以根据接近的匹配项获取结果，即使用户在搜索框中拼错了单词。 尽管不要求这样做，它可显著提高 typeahead 体验的可靠性。 让我们将源行更改为启用模糊匹配，以尝试此操作。

将以下行

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

更改为：

```javascript
source: "/home/suggest?highlights=false&fuzzy=true&",
```

按 F5 启动应用程序。

尝试键入类似于“execative”的字母，会发现返回了包含“executive”的结果，尽管此单词与键入的字母不完全匹配。

### <a name="jquery-autocomplete--backed-by-azure-search-autocomplete"></a>jQuery 记忆式键入功能由 Azure 搜索自动完成功能提供支持

到目前为止，建议了已如何搜索用户体验的代码。 下一步的代码块显示了 jQuery UI 自动完成函数 (行 91 在 index.cshtml)，为 Azure 搜索自动完成的请求中传递：

```javascript
$(function () {
    // using modified jQuery Autocomplete plugin v1.2.6 https://xdsoft.net/jqplugins/autocomplete/
    // $.autocomplete -> $.autocompleteInline
    $("#example2").autocompleteInline({
        appendMethod: "replace",
        source: [
            function (text, add) {
                if (!text) {
                    return;
                }

                $.getJSON("/home/autocomplete?term=" + text, function (data) {
                    if (data && data.length > 0) {
                        currentSuggestion2 = data[0];
                        add(data);
                    }
                });
            }
        ]
    });

    // complete on TAB and clear on ESC
    $("#example2").keydown(function (evt) {
        if (evt.keyCode === 9 /* TAB */ && currentSuggestion2) {
            $("#example2").val(currentSuggestion2);
            return false;
        } else if (evt.keyCode === 27 /* ESC */) {
            currentSuggestion2 = "";
            $("#example2").val("");
        }
    });
});
```

## <a name="c-example"></a>C# 示例

现在，我们已讨论了 web 页面的 JavaScript 代码，让我们看看C#实际检索的建议的匹配项，使用 Azure 搜索.NET SDK 的服务器端的控制器代码。

打开**HomeController.cs**控制器目录下的文件。 

您可能注意到的第一件事是一种方法在调用类的顶部`InitSearch`。 此方法在 Azure 搜索服务中创建经过身份验证的 HTTP 索引客户端。 有关详细信息，请参阅[如何使用.NET 应用程序中的 Azure 搜索](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk)。

在第 41 行，请注意建议函数。 它基于[DocumentsOperationsExtensions.Suggest 方法](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.suggest?view=azure-dotnet-preview)。

```csharp
public ActionResult Suggest(bool highlights, bool fuzzy, string term)
{
    InitSearch();

    // Call suggest API and return results
    SuggestParameters sp = new SuggestParameters()
    {
        UseFuzzyMatching = fuzzy,
        Top = 5
    };

    if (highlights)
    {
        sp.HighlightPreTag = "<b>";
        sp.HighlightPostTag = "</b>";
    }

    DocumentSuggestResult resp = _indexClient.Documents.Suggest(term, "sg", sp);

    // Convert the suggest query results to a list that can be displayed in the client.
    List<string> suggestions = resp.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = suggestions
    };
}
```

Suggest 函数采用两个参数，用于确定是要返回命中的突出显示内容，还是在返回搜索词输入的同时返回模糊匹配结果。 该方法将创建[SuggestParameters 对象](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.suggestparameters?view=azure-dotnet)，后者再传递到建议 API。 然后，结果将转换成 JSON，以便在客户端中显示。

行 69，请注意自动完成函数。 它基于[DocumentsOperationsExtensions.Autocomplete 方法](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.documentsoperationsextensions.autocomplete?view=azure-dotnet-preview)。

```csharp
public ActionResult AutoComplete(string term)
{
    InitSearch();
    //Call autocomplete API and return results
    AutocompleteParameters ap = new AutocompleteParameters()
    {
        AutocompleteMode = AutocompleteMode.OneTermWithContext,
        UseFuzzyMatching = false,
        Top = 5
    };
    AutocompleteResult autocompleteResult = _indexClient.Documents.Autocomplete(term, "sg", ap);

    // Conver the Suggest results to a list that can be displayed in the client.
    List<string> autocomplete = autocompleteResult.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = autocomplete
    };
}
```

记忆式键入功能函数采用搜索术语输入。 该方法将创建[AutoCompleteParameters 对象](https://docs.microsoft.com/rest/api/searchservice/autocomplete)。 然后，结果将转换成 JSON，以便在客户端中显示。

（可选）将断点添加到 Suggest 函数的开头，并逐步执行代码。 请注意，SDK，如何转换为从方法返回的结果返回的响应。

页上的其他示例，遵照相同的模式来添加搜索词突出显示并的面，从而支持客户端缓存的记忆式键入功能结果。 请查看其中的每个示例，以了解其工作原理，以及如何在搜索体验中利用这些功能。

## <a name="javascript-example"></a>JavaScript 示例

记忆式键入功能和建议的 Javascript 实现调用 REST API 中，作为源使用 URI 指定的索引和操作。 

若要查看的 JavaScript 实现，打开**IndexJavaScript.cshtml**。 请注意，jQuery UI 自动完成函数也可用于搜索框中，收集搜索术语输入，进行到 Azure 搜索的异步调用以检索建议的匹配项或完成条款。 

让我们看看第一个示例的 JavaScript 代码：

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: function (request, response) {
        $.ajax({
            type: "POST",
            url: suggestUri,
            dataType: "json",
            headers: {
                "api-key": searchServiceApiKey,
                "Content-Type": "application/json"
            },
            data: JSON.stringify({
                top: 5,
                fuzzy: false,
                suggesterName: "sg",
                search: request.term
            }),
                success: function (data) {
                    if (data.value && data.value.length > 0) {
                        response(data.value.map(x => x["@@search.text"]));
                    }
                }
            });
        },
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

如果将此示例与前面调用主控制器的示例进行比较，会发现两者存在一些类似之处。  `minLength` 和 `position` 的自动完成配置完全相同。 

此处的最明显差别在于源。 而不是在主控制器中调用的建议方法，REST 请求中的 JavaScript 函数创建和执行使用 Ajax。 然后，在“success”中处理响应，并将其用作源。

REST 调用使用 Uri 来指定是否[记忆式键入功能](https://docs.microsoft.com/rest/api/searchservice/autocomplete)或[建议](https://docs.microsoft.com/rest/api/searchservice/suggestions)进行 API 调用。 下面的 Uri 分别是 9 和 10，行上。

```javascript
var suggestUri = "https://" + searchServiceName + ".search.windows.net/indexes/" + indexName + "/docs/suggest?api-version=" + apiVersion;
var autocompleteUri = "https://" + searchServiceName + ".search.windows.net/indexes/" + indexName + "/docs/autocomplete?api-version=" + apiVersion;
```

在行 148，你可以找到脚本中调用`autocompleteUri`。 首次调用`suggestUri`39 行上。

> [!Note]
> 在 JavaScript 中进行 REST 调用的服务提供为的 REST API，方便演示，此处但不是应被解释为最佳做法或建议。 包含的 API 密钥和终结点在脚本中的将打开你最多的拒绝服务攻击到的任何用户可以读取这些值关闭脚本的服务。 它要出于学习目的使用 JavaScript 很安全，同时可能对免费服务上托管的索引建议使用 Java 或C#在生产代码中的索引和查询操作。 

<a name="configure-app"></a>

## <a name="configure-nycjobs-to-run-on-your-service"></a>配置你的服务上运行的 NYCJobs

到目前为止，你已使用托管的 NYCJobs 演示索引。 如果你想要完全的可见性的所有代码，包括索引，将按照这些说明来创建和加载搜索服务中的索引。

1. [创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可以使用此示例中的一项免费服务。 

   > [!Note]
   > 如果使用免费的 Azure 搜索服务，则仅限使用三个索引。 NYCJobs 数据加载器将创建两个索引。 请确保服务中有足够的空间用于接受新索引。

1. 下载[NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs)示例代码。

1. 在 NYCJobs 示例代码的 DataLoader 文件夹，打开**DataLoader.sln** Visual Studio 中。

1. 添加 Azure 搜索服务的连接信息。 打开 DataLoader 项目中的 App.config，更改 TargetSearchServiceName 和 TargetSearchServiceApiKey appSettings，以反映 Azure 搜索服务和 Azure 搜索服务 API 密钥。 可在 Azure 门户中找到这些信息。

1. 按 F5 启动应用程序，创建两个索引和导入 NYCJob 示例数据。

1. 打开**AutocompleteTutorial.sln**和编辑中的 Web.config **AutocompleteTutorial**项目。 将 SearchServiceName 和 SearchServiceApiKey 值更改为有效的搜索服务的值。

1. 按 F5 运行应用程序。 在默认浏览器中打开示例 web 应用。 体验与沙盒版本相同，在服务上托管的索引和数据。

## <a name="next-steps"></a>后续步骤

此示例演示用于生成支持自动完成功能和建议的搜索框中的基本步骤。 您看到了如何可以生成一个 ASP.NET MVC 应用程序和使用 Azure 搜索.NET SDK 或 REST API 来检索建议。

下一步，尝试将建议和记忆式键入功能集成到您的搜索体验。 以下参考文章应帮助。

> [!div class="nextstepaction"]
> [自动完成 REST API](https://docs.microsoft.com/rest/api/searchservice/autocomplete)
> [建议 REST API](https://docs.microsoft.com/rest/api/searchservice/suggestions)
> [“创建索引”REST API 中的分面索引属性](https://docs.microsoft.com/rest/api/searchservice/create-index)

