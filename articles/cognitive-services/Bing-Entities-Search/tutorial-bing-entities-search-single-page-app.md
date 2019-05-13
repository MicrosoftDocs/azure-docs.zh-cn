---
title: 教程：必应实体搜索单页 Web 应用
titlesuffix: Azure Cognitive Services
description: 介绍如何在单页 Web 应用程序中使用必应实体搜索 API。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: tutorial
ms.date: 02/01/2019
ms.author: aahi
ms.openlocfilehash: 1b8cf36c631755458bc0c531773a6b2aba7f1038
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406382"
---
# <a name="tutorial-single-page-web-app"></a>教程：单页 Web 应用

必应实体搜索 API 用于在 Web 中搜索实体和场所的信息。 可以在给定查询中请求一种或两种结果。 场所和实体的定义在下面提供。

|||
|-|-|
|实体|按名称查找的知名人物、场所和事物|
|场所|按名称或类型查找的餐馆、酒店和其他本地商业场所（例如，意大利餐馆）|

本教程将生成一个单页 Web 应用程序，该应用程序使用必应实体搜索 API 直接在页面中显示搜索结果。 该应用程序包含 HTML、CSS 和 JavaScript 组件。

此 API 允许按位置确定结果的优先级。 在移动应用中，可以要求设备提供其自己的位置。 在 Web 应用中，可以使用 `getPosition()` 函数。 但此调用只能在安全的上下文中使用，其提供的位置可能不精确。 另外，用户可能需要搜索自身位置之外的其他位置附近的实体。

因此，应用会调用必应地图服务，从用户输入的位置获取纬度和经度。 然后，用户可以输入地标的名称（“太空针塔”）或者完整或不完整的地址（“纽约市”），必应地图 API 就会提供坐标。

<!-- Remove until we can replace with a sanitized version.
![[Single-page Bing Entity Search app]](media/entity-search-spa-demo.png)
-->

> [!NOTE]
> 页面底部出现的 JSON 和 HTTP 标题揭示单击时的 JSON 响应和 HTTP 请求信息。 这些详细信息在浏览该服务时很有用。

教程应用演示了如何：

> [!div class="checklist"]
> * 在 JavaScript 中执行必应实体搜索 API 调用
> * 在 JavaScript 中执行必应地图 `locationQuery` API 调用
> * 将搜索选项传递到 API 调用
> * 显示搜索结果
> * 处理必应客户端 ID 和 API 订阅密钥
> * 处理可能出现的任何错误

教程页是完全独立的；它不使用任何外部框架、样式表，甚至也不使用外部图像文件， 而只使用获得广泛支持的 JavaScript 语言功能，适用于所有主要 Web 浏览器的最新版本。

在本教程中，我们只讨论源代码的选定部分。 完整的源代码[在单独页上](tutorial-bing-entities-search-single-page-app-source.md)提供。 请将此代码复制并粘贴到文本编辑器中，并将其另存为 `bing.html`。

> [!NOTE]
> 本教程大致类似于[单页必应 Web 搜索应用教程](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md)，但只涉及实体搜索结果。

## <a name="app-components"></a>应用组件

与任何单页 Web 应用一样，本教程应用程序包含以下三个部分：

> [!div class="checklist"]
> * HTML - 定义页面的结构和内容
> * CSS - 定义页面的外观
> * JavaScript - 定义页面的行为

本教程不详细介绍大多数 HTML 或 CSS，因为它们很简单。

HTML 包含搜索表单，用户可以在其中输入查询并选择搜索选项。 该表单连接到实际上会通过 `<form>` 标记的 `onsubmit` 属性执行搜索的 JavaScript：

```html
<form name="bing" onsubmit="return newBingEntitySearch(this)">
```

`onsubmit` 处理程序返回 `false`，这会阻止表单提交到服务器。 JavaScript 代码实际上执行从表单中收集所需的信息并执行搜索的工作。

搜索分两个阶段执行。 首先，如果用户输入了位置限制，则会通过必应地图查询将其转换为坐标。 该查询的回调然后会启动必应实体搜索查询。

HTML 还包含划分（HTML `<div>` 标记），可以在其中显示搜索结果。

## <a name="managing-subscription-keys"></a>管理订阅密钥

> [!NOTE]
> 不管是对必应搜索 API 还是对必应地图 API，此应用都要求使用订阅密钥。 可以使用[试用型必应搜索密钥](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api)和[基本型必应地图密钥](https://www.microsoft.com/maps/create-a-bing-maps-key)。

为了避免必须将必应搜索和必应地图 API 订阅密钥包含在代码中这种情形，请使用浏览器的持久性存储来存储它们。 如果任何一种密钥尚未存储，我们会提示你进行存储，以备后用。 如果该密钥随后被 API 拒绝，我们会使已存储的密钥失效，并会在用户下次进行搜索时要求其提供密钥。

我们定义使用 `localStorage` 对象（如果浏览器支持它）或 Cookie 的 `storeValue` 和 `retrieveValue` 函数。 `getSubscriptionKey()` 函数使用这些函数来存储和检索用户的密钥。

```javascript
// cookie names for data we store
SEARCH_API_KEY_COOKIE = "bing-search-api-key";
MAPS_API_KEY_COOKIE   = "bing-maps-api-key";
CLIENT_ID_COOKIE      = "bing-search-client-id";

// API endpoints
SEARCH_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/entities";
MAPS_ENDPOINT   = "https://dev.virtualearth.net/REST/v1/Locations";

// ... omitted definitions of storeValue() and retrieveValue()

// get stored API subscription key, or prompt if it's not found
function getSubscriptionKey(cookie_name, key_length, api_name) {
    var key = retrieveValue(cookie_name);
    while (key.length !== key_length) {
        key = prompt("Enter " + api_name + " API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(cookie_name, key);
    return key;
}

function getMapsSubscriptionKey() {
    return getSubscriptionKey(MAPS_API_KEY_COOKIE, 64, "Bing Maps");
}

function getSearchSubscriptionKey() {
    return getSubscriptionKey(SEARCH_API_KEY_COOKIE, 32, "Bing Search");
}
```

在页面完成加载后，HTML `<body>` 标记包括名为 `getSearchSubscriptionKey()` 和 `getMapsSubscriptionKey()` 的 `onload` 属性。 这些调用的作用是在用户未输入其密钥的情况下立即提示用户输入密钥。

```html
<body onload="document.forms.bing.query.focus(); getSearchSubscriptionKey(); getMapsSubscriptionKey();">
```

## <a name="selecting-search-options"></a>选择搜索选项

![[必应实体搜索表单]](media/entity-search-spa-form.png)

HTML 表单包括以下控件：

| | |
|-|-|
|`where`|用于选择市场（位置和语言）进行搜索的下拉菜单。|
|`query`|用于输入搜索词的文本字段。|
|`safe`|指示是否启用安全搜索（限制“成人”内容）的复选框|
|`what`|选择搜索实体和/或场所的菜单。|
|`mapquery`|文本字段，用户可以在其中输入完整的或不完整的地址、地标等，以便必应实体搜索返回更相关的结果。|

> [!NOTE]
> 场所结果目前仅在美国提供。 `where` 和 `what` 菜单有代码来强制实施此限制。 在 `what` 菜单中选择“场所”后，如果选择非美国的市场，则 `what` 会更改为“任何内容”。 在 `where` 菜单中选择非美国的市场后，如果选择“场所”，则 `where` 会更改为“美国”。

JavaScript 函数 `bingSearchOptions()` 会将这些字段转换为适用于必应搜索 API 的特定查询字符串。

```javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.what.selectedIndex) options.push("responseFilter=" + form.what.value);
    return options.join("&");
}
```

例如，安全搜索功能可以是 `strict`、`moderate` 或 `off`，`moderate` 为默认设置。 但是，我们的表单使用只有两种状态的复选框。 JavaScript 代码会将此设置转换为 `strict` 或 `off`（不使用 `moderate`）。

`mapquery` 字段不在 `bingSearchOptions()` 中处理，因为它适用于必应地图位置查询，不适用于必应实体搜索。

## <a name="obtaining-a-location"></a>获取位置

必应地图 API 提供 [`locationQuery` 方法](//msdn.microsoft.com/library/ff701711.aspx)，用于查找用户所输入位置的纬度和经度。 然后，这些坐标会和用户的请求一起传递给必应实体搜索 API。 搜索结果会对靠近指定位置的实体和场所进行优先级划分。

不能在 Web 应用中使用普通的 `XMLHttpRequest` 查询来访问必应地图 API，因为此服务不支持跨域查询。 好在它支持 JSONP（“P”代表“padded”，即“填充式”）。 JSONP 响应是包装在函数调用中的普通 JSON 响应。 进行请求的方式是将 `<script>` 标记插入到文档中。 （加载脚本不受浏览器安全策略约束。）

`bingMapsLocate()` 函数为查询创建并插入 `<script>` 标记。 查询字符串的 `jsonp=bingMapsCallback` 段指定要随响应一起调用的函数的名称。

```javascript
function bingMapsLocate(where) {

    where = where.trim();
    var url = MAPS_ENDPOINT + "?q=" + encodeURIComponent(where) + 
                "&jsonp=bingMapsCallback&maxResults=1&key=" + getMapsSubscriptionKey();

    var script = document.getElementById("bingMapsResult")
    if (script) script.parentElement.removeChild(script);

    // global variable holds reference to timer that will complete the search if the maps query fails
    timer = setTimeout(function() {
        timer = null;
        var form = document.forms.bing;
        bingEntitySearch(form.query.value, "", bingSearchOptions(form), getSearchSubscriptionKey());
    }, 5000);

    script = document.createElement("script");
    script.setAttribute("type", "text/javascript");
    script.setAttribute("id", "bingMapsResult");
    script.setAttribute("src", url);
    script.setAttribute("onerror", "BingMapsCallback(null)");
    document.body.appendChild(script);

    return false;
}
```

> [!NOTE]
> 如果必应地图 API 不响应，则不会调用 `bingMapsCallBack()` 函数。 通常情况下，这意味着不调用 `bingEntitySearch()`，因此实体搜索结果不会显示。 为了避免此情况，`bingMapsLocate()` 还设置了一个在五秒钟后调用 `bingEntitySearch()` 的计时器。 在回调函数中有避免执行实体搜索两次的逻辑。

查询完成后，会根据请求调用 `bingMapsCallback()` 函数。

```javascript
function bingMapsCallback(response) {

    if (timer) {    // we beat the timer; stop it from firing
        clearTimeout(timer);
        timer = null;
    } else {        // the timer beat us; don't do anything
        return; 
    }

    var location = "";
    var name = "";
    var radius = 1000;

    if (response) {
        try {
            if (response.statusCode === 401) {
                invalidateMapsKey();
            } else if (response.statusCode === 200) {
                var resource = response.resourceSets[0].resources[0];
                var coords   = resource.point.coordinates;
                name         = resource.name;

                // the radius is the largest of the distances between the location and the corners
                // of its bounding box (in case it's not in the center) with a minimum of 1 km
                try {
                    var bbox    = resource.bbox;
                    radius  = Math.max(haversineDistance(bbox[0], bbox[1], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[1]),
                                       haversineDistance(bbox[0], bbox[3], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[3]), 1000);
                } catch(e) {  }
                var location = "lat:" + coords[0] + ";long:" + coords[1] + ";re:" + Math.round(radius);
            }
        }
        catch (e) { }   // response is unexpected. this isn't fatal, so just don't provide location
    }

    var form = document.forms.bing;
    if (name) form.mapquery.value = name;
    bingEntitySearch(form.query.value, location, bingSearchOptions(form), getSearchSubscriptionKey());

}
```

除了纬度和经度，必应实体搜索查询还需要一个 *radius*（半径），用于指示位置信息的精确度。 我们使用在必应地图响应中提供的边界框来计算半径。 边界框是一个将整个位置包围住的矩形。 例如，如果用户输入 `NYC`，则结果大致包含纽约市的中心坐标以及一个围绕该市的边界框。 

首先使用函数 `haversineDistance()`（未显示）计算从主坐标分别到边界框的四个角的距离。 在这四个距离中，使用最大的一个作为半径。 最小半径为一公里。 如果响应中未提供边界框，则此值也会用作默认值。

获得坐标和半径以后，即可调用 `bingEntitySearch()` 执行实际搜索。

## <a name="performing-the-search"></a>执行搜索

有了查询、位置、选项字符串和 API 密钥以后，`BingEntitySearch()` 函数就会发出必应实体搜索请求。

```javascript
// perform a search given query, location, options string, and API keys
function bingEntitySearch(query, latlong, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "error");

    var request = new XMLHttpRequest();
    var queryurl = SEARCH_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // open the request
    try {
        request.open("GET", queryurl);
    } 
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // add request headers
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");

    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    if (latlong) request.setRequestHeader("X-Search-Location", latlong);

    // event handler for successful response
    request.addEventListener("load", handleBingResponse);
    
    // event handler for erorrs
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // event handler for aborted request
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // send the request
    request.send();
    return false;
}
```

HTTP 请求成功完成后，JavaScript 会调用 `load` 事件处理程序（即 `handleBingResponse()` 函数）来处理对 API 的成功 HTTP GET 请求。 

```javascript
// handle Bing search request results
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and HTTP request
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " + 
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "SearchResponse") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    if (divHidden("pole") && divHidden("mainline") && divHidden("sidebar")) 
        showDiv("noresults", "No results.<p><small>Looking for restaurants or other local businesses? Those currently areen't supported outside the US.</small>");
    }

    // Any other HTTP status is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSearchKey();

        // some error responses don't have a top-level errors object, so gin one up
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // display HTTP status code
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // add all fields from all error responses
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // also display Bing Trace ID if it isn't blocked by CORS
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // and display the error message
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> HTTP 请求成功不一定意味着搜索本身成功。 如果搜索操作中出现错误，必应实体搜索 API 将返回非 200 HTTP 状态代码并将错误信息包含在 JSON 响应中。 此外，如果请求速率受限制，该 API 还会返回空响应。

上面两个函数中的很多代码专用于错误处理。 以下阶段可能会出现错误：

|阶段|可能的错误|处理方式|
|-|-|-|
|生成 JavaScript 请求对象|无效的 URL|`try`/`catch` 块|
|发出请求|网络错误，已中止连接|`error` 和 `abort` 事件处理程序|
|执行搜索|无效的请求、无效的 JSON、速率限制|`load` 事件处理程序中的测试|

处理错误时，会调用 `renderErrorMessage()`，获取有关该错误的任何已知详细信息。 如果响应通过全部错误测试，则会调用 `renderSearchResults()`，在页面中显示搜索结果。

## <a name="displaying-search-results"></a>显示搜索结果

必应实体搜索 API [要求按指定顺序显示结果](use-display-requirements.md)。 由于 API 可能返回两种不同类型的响应，在 JSON 响应中循环访问顶级 `Entities` 或 `Places` 集合并显示这些结果会显得不够。 （如果只需一种类型的结果，请使用 `responseFilter` 查询参数。）

请在搜索结果中改用 `rankingResponse` 集合，对要显示的结果排序。 此对象是指 `Entitiess` 和/或 `Places` 集合中的项。

`rankingResponse` 最多可能包含三个搜索结果集合，分别称为 `pole`、`mainline` 和 `sidebar`。 

`pole`，如果存在，是最相关的搜索结果，应该突出显示。 `mainline` 是指主要搜索结果。 mainline 结果应该紧跟在 `pole` 后面显示（或者显示在最前面，如果 `pole` 不存在的话）。 

最后， `sidebar` 是指辅助搜索结果。 它们可以显示在实际的侧栏中，或者直接显示在 mainline 结果之后。 我们为教程应用选择了后者。

`rankingResponse` 集合中的每个项采用两种不同但却等效的方式来映射实际的搜索结果项。

| | |
|-|-|
|`id`|`id` 看起来像 URL，但不应将其用于链接。 排名结果的 `id` 类型与答案集合中某个搜索结果项的 `id` 匹配，或者与整个答案集合（例如 `Entities`）匹配。
|`answerType`<br>`resultIndex`|`answerType` 是指包含结果的顶级答案集合（例如 `Entities`）。 `resultIndex` 是指结果在该集合中的索引。 如果省略 `resultIndex`，则排名结果指整个集合。

> [!NOTE]
> 有关此部分搜索响应的详细信息，请参阅[排名结果](rank-results.md)。

可以根据应用程序的情况，使用最方便的方法来定位引用的搜索结果项。 在教程代码中，我们使用 `answerType` 和 `resultIndex` 来定位每个搜索结果。

最后，让我们看看函数 `renderSearchResults()`。 此函数循环访问三个 `rankingResponse` 集合，这些集合代表搜索结果的三个部分。 对于每个部分，我们都调用 `renderResultsItems()` 来呈现该部分的结果。

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // if spelling was corrected, update search field
    if (results.queryContext.alteredQuery) 
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // for each possible section, render the results from that section
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

## <a name="rendering-result-items"></a>呈现结果项

在 JavaScript 代码中有一个对象 `searchItemRenderers`，其中包含呈现器函数，为每种搜索结果生成 HTML。

```javascript
searchItemRenderers = { 
    entities: function(item) { ... },
    places: function(item) { ... }
}
```

呈现器函数可接受以下参数：

| | |
|-|-|
|`item`|包含项目属性（例如其 URL 和说明）的 JavaScript 对象。|
|`index`|结果项集合中的结果项的索引。|
|`count`|搜索结果项集合中的项数。|

`index` 和 `count` 参数可用于计算结果数、为集合的开头或末尾生成特殊的 HTML、在一定数量的项后插入换行符，等等。 呈现器在不需要此功能的情况下，不需接受这两个参数。 事实上，我们不在教程应用的呈现器中使用它们。

让我们更仔细地看看 `entities` 呈现器：

```javascript
    entities: function(item) {
        var html = [];
        html.push("<p class='entity'>");
        if (item.image) {
            var img = item.image;
            if (img.hostPageUrl) html.push("<a href='" + img.hostPageUrl + "'>");
            html.push("<img src='" + img.thumbnailUrl +  "' title='" + img.name + "' height=" + img.height + " width= " + img.width + ">");
            if (img.hostPageUrl) html.push("</a>");
            if (img.provider) {
                var provider = img.provider[0];
                html.push("<small>Image from ");
                if (provider.url) html.push("<a href='" + provider.url + "'>");
                html.push(provider.name ? provider.name : getHost(provider.url));
                if (provider.url) html.push("</a>");
                html.push("</small>");
            }
        }
        html.push("<p>");
        if (item.entityPresentationInfo) {
            var pi = item.entityPresentationInfo;
            if (pi.entityTypeHints || pi.entityTypeDisplayHint) {
                html.push("<i>");
                if (pi.entityTypeDisplayHint) html.push(pi.entityTypeDisplayHint);
                else if (pi.entityTypeHints) html.push(pi.entityTypeHints.join("/"));
                html.push("</i> - ");
            }
        }
        html.push(item.description);
        if (item.webSearchUrl) html.push("&nbsp;<a href='" + item.webSearchUrl + "'>More</a>")
        if (item.contractualRules) {
            html.push("<p><small>");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++) {
                var rule = item.contractualRules[i];
                var link = [];
                if (rule.license) rule = rule.license;
                if (rule.url) link.push("<a href='" + rule.url + "'>");
                link.push(rule.name || rule.text || rule.targetPropertyName + " source");
                if (rule.url) link.push("</a>");
                rules.push(link.join(""));
            }
            html.push("License: " + rules.join(" - "));
            html.push("</small>");
        }
        return html.join("");
    }, // places renderer omitted
```

实体呈现器函数：

> [!div class="checklist"]
> * 生成 HTML `<img>` 标记以显示图像缩略图（如果有）。 
> * 生成链接到图像所在页面的 HTML `<a>` 标记。
> * 生成可显示有关图像及其所在站点的信息的说明。
> * 使用显示提示（如果有）整合实体的分类。
> * 包括一个必应搜索链接，用于获取有关实体的详细信息。
> * 显示数据源要求的任何许可或归属信息。

## <a name="persisting-client-id"></a>保留客户端 ID

来自必应搜索 API 的响应可能包含应通过后续请求发送回 API 的 `X-MSEdge-ClientID` 标头。 如果正在使用多个必应搜索 API，应将相同客户端 ID 用于所有这些必应搜索 API（如有可能）。

提供 `X-MSEdge-ClientID` 标头可以让必应 API 关联用户的所有搜索，这有两个主要好处。

首先，它允许必应搜索引擎将过去的上下文应用于搜索来查找更好地满足用户的结果。 例如，如果用户以前搜索过与航海相关的词汇，则稍后搜索“码头”时，系统可能会优先返回可以让帆船停靠的码头的信息。

其次，在新功能广泛应用之前，必应可能会随机选择用户体验该功能。 为每个请求提供相同客户端 ID 可确保被允许看到某个功能的用户可以始终看到该功能。 如果没有客户端 ID，用户可能会看到功能在其搜索结果中随机出现和消失。

浏览器安全策略 (CORS) 可能会阻止将 `X-MSEdge-ClientID` 标头提供给 JavaScript。 当搜索响应的域不同于请求搜索的页面时，会出现此限制。 在生产环境中，应该托管一个服务器端脚本，以便在网页所在的域进行 API 调用，这样就可以解决此策略的问题。 由于脚本与网页的域相同，因此 `X-MSEdge-ClientID` 标头可供 JavaScript 使用。

> [!NOTE]
> 在生产型 Web 应用程序中，无论如何都应在服务器端执行请求。 否则就必须将必应搜索 API 密钥包含在网页中，这样查看源代码的任何人都可以获得它。 收费取决于 API 订阅密钥下的所有使用量（即使请求是由未经授权的用户发出的，也是如此），因此请确保不要公开你的密钥。

进行开发时，可以通过 CORS 代理发出必应 Web 搜索 API 请求。 来自此类代理的响应有一个 `Access-Control-Expose-Headers` 标头，此标头将响应头列入允许列表，并将它们提供给 JavaScript。

安装 CORS 代理很容易，教程应用可以用它来访问客户端 ID 标头。 首先，如果尚未安装 Node.js，请[安装它](https://nodejs.org/en/download/)。 然后，在命令窗口中发出以下命令：

    npm install -g cors-proxy-server

接下来，在 HTML 文件中将必应 Web 搜索终结点更改为：

    https://localhost:9090/httpss://api.cognitive.microsoft.com/bing/v7.0/search

最后，使用以下命令启动 CORS 代理：

    cors-proxy-server

使用教程应用期间，不要关闭命令窗口；关闭窗口会导致代理停止运行。 在搜索结果下的可展开 HTTP 响应头部分中，现在可以看到 `X-MSEdge-ClientID` 响应头（以及其他响应头），并验证此响应头是否对所有请求都相同。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [必应实体搜索 API 参考](//docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference)

> [!div class="nextstepaction"]
> [必应地图 API 文档](//msdn.microsoft.com/library/dd877180.aspx)
