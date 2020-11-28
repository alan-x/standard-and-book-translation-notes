### Cross-Origin Resource Sharing

### 概述
这个稳定定义了一个可以让客户端发送跨域请求的机制。使用一个 API 去生成资源的跨域请求的规格可以使用定义在这个规格中的算法。如果有一个 API 使用在 http://example.org 资源上，在 http://hello-world.example 上的资源可以可选的使用这个规格描述的机制，让这个资源可以从 http://example.org 上被好跨域获取。

### 这个文档的状态


### 1 介绍
这个章节是非正式的。

用户代理通常使用同源限制网络请求。这个限制阻止一个域的客户端应用从另一个域获取数据，同时也限制不安全的 HTTP 请求，这些请求可以自动启动不同于正在运行的应用的域的目标。

遵循这个模式的用户代理，跨域请求通常包含用户认证，包括 HTTP 认证和 cookie 信息。

这个规格从多种方式扩展这个模型：
- 一个响应可以包含 Access-Control-Allow-Origin 头部，使用请求指来源的域作为值，用来允许资源内容访问。用户代理验证这个值和请求来源是否一致。
- 用户代理可以通过预检请求来检查一个跨域资源是否准备好接收请求，使用一个非简单方法，从一个指定域。这再一次被用户代理检验。
- 服务端应用通过 Origin 头部，可以查明一个 HTTP 请求被用户认定为跨域请求。这个扩展可以让服务端应用强制限制他们想要去服务的跨域请求（比如，什么也不返回）。

这个规范是其他规范的基础，所以叫做 CORS API 规格。定义了这个规格是怎么使用的。栗子是 Server-Sent Event 和 XMLHttpRequest。

CORS wiki 页面提供关于这个文档更多的背景信息。

如果资源作者有一个简单的文字资源部署在 http://example.com/hello，他包含字符串“Hello World！”，类似 http://hello-world.example 可以访问它，响应绑定了一个这个文档引入的头部，它看起来就像这样：
```
    Access-Control-Allow-Origin: http://hello-world.example
    
    Hello World
```
http://hello-world.example 上的客户端网页应用可以使用 XMLHttpRequest 可以像这样访问这个资源：
```
    var client = new XMLHttpRequest()
    client.open("GET", "http://example.com/hello")
    client.onreadystatechange = function() { /* do something */}
    client.send()
```

如果资源作者想要处理不使用简单方法的跨域请求，会稍微复杂一点。在这种场景下，作者需要回应一个使用 OPTIONS 方法的预检请求，然后需要去处理使用期望方法（在这个栗子中是 DELETE）的真正的请求，并给出一个适合的响应。给预检请求的响应可能会有以下的头部：
```
    Access-Control-Allow-Origin: http://hello-world.example
    Access-Control-Max-Age: 36288000
    Access-aontrol-Allow-Methods: PUT, DELETE
```
Access-Control-Max-Age 头部指示响应可以缓存多久，所以在指定的时间内，接下来的请求，不需要再发送预检请求。Access-Control-Allow-Method 头部指示可以使用在真实请求的方法。返回给真实请求的响应可以简单的包含这个头部：
```
    Access-Control-Allow-Origin: http://hello-world.example
```
发送复杂的额外预检请求是用户代理的任务。假设应用部署在 http://calendar.example/app，作者可以使用下面的 XMLHttpRequest ECMAScript 片段：
```
    function deleteItem(itemId, updateUI) {
        var client = new XMLHttpReqeust()
        client.open("DELETE", "http://calendar.example/app")
        client.onload = updateUI
        client.onerror = updateUI
        client.onabort = updateUI
        client.send("id=" + itemId)
    }
```

### 2 兼容性
这个规格是为资源作者和用户代理写的。它包含对定义 API 使用定义在这个规格的跨域请求算法的规格的建议—— CORS API 规格 ——和通用安全考虑章节包含一些对客户端 Web 应用作者的建议。

就像章节和附录标记为非规范，这个规格中所有的图片，栗子，和笔记都是非规范的。其他任何东西都是规范的。

在这个规格中，词必须和可能解释为 RFC 2119 中描述的那样。

未完待续


### 3 术语

这个规格中的一些术语来自 The Web Origin Concept，HTML，HTTP 和 URI。

术语定义通常贯穿整个规格。然而，这里定义了少量并不是适用于任何地方的定义。

大小写敏感方式对比两个字符串意味着精确的一个码点一个码点的对比他们。

ASCII 大小写不敏感的对比两个字符串意味着精确的一个码点一个码点的对比，期望在 U+0041 LATIN CAPITAL LETTER A 到 U+005A LATIN CAPITAL LETTER Z 和在 U+0061 LATIN SMALL LETTER A 到 U+007A LATIN SMALL LETTER Z 对应的字符被认为是匹配的。

将一个字符串转化为 ASCII 小写意味着替换任何在 U+0041 LATIN CAPITAL LETTER A 到 U+005A LATIN CAPITAL LETTER Z 内的字符为 U+0061 LATIN SMALL LETTER A 到 U+007A LATIN SMALL LETTER Z 对应的字符。

术语用户凭证在这个规格中意味着 cookies，HTTP 认证，和客户端 SSL 证书将会基于用代理和域的先前的交互。具体来说他不涉及代理认证或者 Origin 头部。

术语跨域用来表示非同源

如果一个 method 大小写敏感的匹配下面的一个，则称为简单请求：
- GET
- HEAD
- POST

如果一个 header 的头部域名字是 ASCII 大小写不敏感的命中 Accept，Accept-Langue，或者 Content-Language 或者它是一个 ASCII 大小写不敏感命中 Content-Type，并且头部域值媒体类型（不包含参数）是 ASCII 大小写不敏感命中 application/x-www-form-urlencoded，multipart/form-dat，或者 text/plain，则称之为简单头部。

如果一个 header 的头域名字 ASCII 大小写敏感命中以下其中一个，则称之为简单响应头部：
- Cache-Control
- Content-Language
- Expires
- Last-Modified
- Pragma

当转化一个头部的时候，头部必须转化为语法章节对应的 ABNF“ 产生式。如果头部不匹配产生式，则说明头部转化失败。

### 4 安全考虑

### 5 语法

这个章节定义了这个规格引入的新头部的语法。同时也提供每一个头部功能简短的介绍。

资源处理模型章节详细说明了在响应中如何使用这些头部，用户代理处理模型章节详细说明了用户代理如何使用这些头部

这个章节使用的 ABNF 语法来自 HTTP/1.1。

> 注意：HTTP/1.1 作为 ABNF 的基础是为了确保这个规格的新头部有相同的转化规则
> 注意：HTTP/1.1 现在没有定义一个隐含的 OWS 在头部值的开头，但是这里假设这种格式存在。

### 5.1 Access-Control-Allow-Origin 响应头部

Access-Control-Allow-Origin 头部指示一个资源分享，通过在响应汇总返回请求的 Origin 头部返回的值，“*”，或者“null”。ABNF：
```
    Access-Control-Allow-Origin = "Access-Control-Allow-Origin" ":" " origin-list-or-null" | "*"
```
> 注意：在实践中，origin-list-or-null 产生式更加严格，不允许返回一个空格分隔的源列表，而是返回一个单独的源或者字符串“null”

### 5.2 Access-Contro-Allow-Credentials 响应头部

Access-Control-Allow-Credentials 头部指示请求的响应是否可以在缺省凭证标志未设置的时候暴露。当作为预检请求的响应的一部分的时候，他表示真实的请求可以包含用户凭证。ABNF：
```
Access-Control-Allow-Credentials = "Access-Control-Allow-Credentials" ":" true
                            true = %x74.72.75.65 ; "true", 大小写敏感
```
### 5.3 Access-Control-Expose-Header 响应头部

Access-Control-Expose-Header 响应头部指示哪一个头部可以安全的暴露给一个 CORS API 规格的 API。ABNF：
```
Access-Control-Expose-Headers = "Access-Control-Expose-Headers" ":" #field-name
```

### 5.4 Access-Control-Max-Age 响应头部
Access-Control-Max-Age 头部指示预检请求可以被缓存在预检结果缓存多久。ABNF：
```
Access-Control-Max-Age = "Access-Control-Max-Age" ":" delta-seconds
```

### 5.5 Access-Control-Allow-Method 响应头部
Access-Control-Allow-Method 头部指定，作为预检请求响应的一部分，哪一个方法可以在真实请求中使用。
> 注意： `Allow`头部和 CORS 协议的目的无关。ABNF：
```
Access-Control-Allow-Methods: "Access-Control-Allow-Methods" ":" #Method
```

### 5.6 Access-Control-Allow-Headers 响应头部

Access-Control-Allow-Headers 头部指示，作为预检请求的响应的一部分，哪一个头部域名字可以被真实的请求使用。ABNF：
```
Access-Control-Allow-Headers = "Access-Control-Allow-Headers" ":" #field-name
```

### 5.7 Origin 请求头部
Origin 头部指示跨域请求或者预检请求的来源。

### 5.8 Access-Control-Request-Method 请求头
Access-Control-Request-Method 头部作为预检请求的一部分指示哪一个方法将被使用在真实请求。ABNF：
```
Access-Control-Request-Method = "Access-Control-Request-Method" ":" Mehod
```

### 5.9 Access-Contro-Request-Headers 请求头部
 Access-Control-Request-Headers 头部作为检请求的一部分指示哪一个头部将会在真实的请求中。ABNF：
 ```
 Access-Control-Request-Headers = "Access-Contro-Request-Heades" ":" #field-name
 ```

### 6 资源处理模型

这个章节描述了资源处理模型需要实现的东西。请求资源的每一种类型可能需要处理的都描述在它自己的子章节。

这个规格描述的资源共享策略都绑定到特定的资源。为了这个章节的目标，每一个资源都被如下绑定：
- 一个域列表由0个或者多个允许访问这个资源的域组成，
    > 注意：这可以包括资源它自己的域，尽管要注意请求跨域资源可以重定向回资源。
- 一个方法列表由0个或者多个资源支持的方法组成。
- 一个头部列表由0个或者多个资源支持的头部域名字组成。
- 一个暴露的头部由0个或者多个资源可能使用的并且可以暴露的不是简单响应头部的头部域名字组成。
- 一个支持的凭证标志指示资源请求中是否支持用户凭证。如果资源支持则为真没否则为假。

### 6.1 简单跨域请求，实际请求，和重定向
在简单跨域请求或者实际请求的响应中，资源指示是否分享响应。

如果资源已经搬迁了，他指示是否分享它的新 URL。

资源必须使用下面的步骤集合去决定哪一个额外的头部在响应中使用：
1. 如果 Origin 头部不存在，终止这些步骤。这个请求不再这个规格的范围内。
2. 如果 Origin 头部的值不是大小写敏感命中任何域列表中的值，不要设置任何额外的头部并终止这些步骤集合。

> 注意：总是命中也是可以接收的，因为域列表可以是无限的

3. 如果资源支持凭证，添加一个单独的 Access-Contro-Allow-Origin 头部，它的值是 Origin 头部的值，并添加单独的 Access--Control-Allow-Credentials 头部，它的值是大小写敏感的字符串“true”。

否则，添加一个单独的 Access-Control-Allow-Origin 头部，它的值是 Origin 头部的值或者字符串“*”。

4. 如果暴露头部列表不是空的，添加一个或者多个 Acces
-Control-Expose-Headers 头部，暴露头部列表给定的头部域名字作为值。
> 注意：通过添加不适当的头部，资源也可以清理预请求结果缓存的所有域是大小写敏感命中 Origin 头部并且 url 是大小写敏感命中资源 URL 的条目。


### 6.2 预请求

资源在预检请求的响应中指示那个方法和头部（不是简单方法和简单头部）他将会处理，和他是否支持凭证。

资源必须使用下面的步骤集合去决定哪个额外的头部在响应中使用：
1. 如果 Origin 头部不存在，终止这些步骤。这个请求不再这个规格的范围内。
2. 如果 Origin 头部的值不是大小写敏感命中域列表的任何值，不要设置任何额外的头部，并终止这些步骤集合。
> 注意：总是命中也是可以接收的，因为域列表可以是无限的
> 注意：Origin 头部可以只包含一个单独的域，因为用户代理将不会遵循重定向。
3. 让 method 为转化 Access-Control-Request-Method 头部的结果的值。如果没有 Access-Control-Request-Method 头部或者抓花失败，不要设置额外的头部并终止这些步骤集合。这个请求在这个规格的范围之外。
4. 让 header field-names 为转化 Access-Control-Request-Headers 头部的结果的值。如果没有 Access-Control-Reqeust-Header 头部，让 header field-names 为空列表。
5. 如果 method 不是大小写敏感命中方法列表中的值，不要设置任何额外的头部，并终止这些步骤的集合。
> 注意：总是命中也是可以接收的，因为域列表可以是无限的
6. 如果 header field-name 不是 ASCII 大小写不敏感命中头部列表中的值，不要设置任何额外的头部，并终止这些步骤的集合。
> 注意：总是命中也是可以接收的，因为域列表可以是无限的
7. 如果资源支持凭证，添加一个单独的 Access-Contro-Allow-Origin 头部，它的值是 Origin 头部的值，并添加单独的 Access--Control-Allow-Credentials 头部，它的值是大小写敏感的字符串“true”。
> 注意：字符串“*”不能用在支持凭证的的资源。
8. 可选的添加一个单独的 Access-Control-Max-Age 头部，并使用允许用户代理缓存请求的结果的秒数作为值。
9. 如果 method 是一个简单方法，这些步骤可以跳过。添加一个或者多个由方法列表组成的 Access-Control-Allow-Methods 头部。
> 注意：如果方法是简单方法，他不需要被列出，但这不是被禁止的。
> 注意：因为方法列表是无限的，简单返回 Access-Control-Request-Method（如果支持） 指定的方法就足够了
10. 如果每一个 header field-names 是简单头部并且没有 Content-Type，这些步骤可以跳过。添加一个或者多个由头部列表（一个子集）组成的 Access-Control-Allow-Headers 头部。
> 注意：如果 header field-name 是一个简单头部，并且不是 Content-Type，他不需要被列出来。Content-Type 仅作为它的值的子集被列出，相当于简单头部。
> 注意：因为头部列表是无限的，简单返回 Access-Control-Allow-Headers 支持的头部就足够了。

否则，添加一个单独的 Access-Control-Allow-Origin 头部，它的值是 Origin 头部的值或者字符串“*”。

### 6.3 安全
这个章节是非规范的。

强烈推荐资源作者去保证请求使用安全方法，比如，GET 和 OPTIONS，没有副作用，因此潜在的攻击无法简单修改用户数据。如果资源被这样设置，攻击者必须在域列表上才能造成有效的伤害。

除了检车 Origin 头部，强烈鼓励资源作者去检查 Host 头部。也就是说，确保头部提供的主机名和资源部署的服务器的主机名一致。

### 6.4 实现考虑
这个章节是非规范的。

希望可以分享多个 Origin 但是不返回相同的“*”的资源实际上必须为每个他们想要允许的请求在响应中动态生成 Access-Control-Allow-Origin 头部。作为结果，这种资源的作者应该发送 vary：Origin HTTP 头部或者提供其他适合的控制指令去防止缓存这种响应，如果复用跨域可能不准确。

### 7 用户代理处理模型
这个章节描述了用户代理需要实现的处理模型。

这个章节的处理模型需要被定义算法什么时候被调用，返回值如何被处理的CORS API 规范引用。这个处理模型不适合单独使用。

### 7.1 跨域请求
- 请求 URL：获取的 URL
    > 注意：请求 URL 在重定向之前被修改
- 请求方法：请求的方法。如果没有明确设置，则是 GET。
- 作者请求头部：作者给请求设置的头部列表。除非明确设置，否则就是空的。
- 请求实体正文：请求的实体正文。除非明确设置，否则就是没有。
- 源域：请求的源
    > 因为某些 API 的特殊性，它无法被以常用的方式被定义，因此，它被作为参数提供。
- 引用源：一个 Document 或者 URL。用来确认 Referer 头部。

- 缺省凭证标志：当用户凭证在请求中被排除和当 cookie 在它的响应被忽略的时候设置。

- 强制预检标志：预检请求必须的时候设置。

跨域请求算法可以被希望允许定义的网络 API 跨域请求的CORS API 规格使用。
> 注意：CORS API 规格
可以自由的限制跨域请求的能力。比如，缺省凭证标志可以总是被设置。

当开渔请求算法被调用，必须执行下面的步骤：
1. 如果因为一些原因，用户代理不想让请求终止这个算法，并设置跨域请求状态为网络错误。
> 注意：请求 URL 可能被用户以某种方式列入黑名单
2. 如果下面的条件为真，尊许简单跨域请求算法：
    - 请求方式是简单方法并且强制预请求标志未设置
    - 每一个作者请求头部是简单头部，或者作者请求头部是空。
3. 否则，遵循使用预检的跨域请求算法
> 注意：跨域请求使用简单方法，但是作者请求头部不是简单的，将会有一个预检请求去确保资源可以处理这些头部。（就像请求使用一个非简单请求方法。）

### 7.1.1 处理跨域请求响应
在暴露响应头部给 CORS API 规格定义的 API 之前，用户代理必须过滤所有的响应头部，粗了那些简单响应头部或者域名称 ASCII 大小写不敏感命中 Access-Control-Expose-Headers 头部中的任何值。

> 注意：XMLHttpReqeust 的 getResponseHeader() 方法将不会暴露任何前面没有指定的头部。

### 7.1.2 跨域请求状态
每一个跨域请求有关联的跨域请求状态，CORS API 规格启用一个 API 去创建一个跨域请求可以勾入。他在开渔请求期间最多可以有两个不同的值，值是：
- preflight complete：用户代理正要创建一个实际的请求。
- success：资源可以被分享
- abort error：用户放弃请求
- 网路错误：资源无法被分享。也用于 DNS 错误，TLS 协商失败，或者其他类型的网络错误发生。这没有包含指示错误的 HTTP 响应类型，比如 HTTP 状态码 410。

### 7.1.3 源域
源域是初始化域，用户代理必须使用 Origin 头部。他可以在重定向步骤被修改。

### 7.1.4 简单同源请求

下面描述的步骤是用户代理处理简单跨域请求要做的：
1. 应用创建一个请求步骤并在创建请求的额时候观察 request rules
    - 如果手动重定向标志未设置并且响应的 HTTP 状态码是 301，203，303，307，或者308。应用重定向步骤
    - 如果最终用户终止请求，应用放弃步骤。
    - 如果是一个网络错误，DNS错误，TLS 协商失败，或者其他类型网络错误的场景，应用网络错误步骤。不要请求任何类型的最终用户交互。
    > 注意：这没有包括指定一些类型错误的 HTTP 响应，比如 HTTP 状态码 404
    - 否则，执行资源分享检测。如果返回是啊比，应用网络错误步骤。否则，如果它返回通过，终止这个算法，并设置跨域请求状态为 success。不要真的终止请求。

### 7.1.5 带预检的跨域请求
为了保护在此规格存在之前的用户代理资源免受跨于请求，一个预检请求被创建，确保资源知道这个规格。这个请求的结果被缓存为预检结果缓存。

### 7.1.6 预检请求缓存
下面的步骤描述了用户代理处理带预检的跨域请求必须要做的。请求一个非同源 URL 受限需要使用预检结果缓存条目或者一个预检请求。
1. 去下一步，如果下面的条件是真：
    - 请求方法是一个方法缓存命中或者它是简单方法并且强制预检标志未设置。
    - 用户请求头部的每一个头部的域名称是头部缓存命中或者是简单头部
   否则，创建一个预检请求。从 origin 的源域获取请求 URL，使用引用源作为 override referer source，和手动重定向标志混合阻塞 cookies 标志设置，泗洪方法 OPTIONS，和下面额外的约束：
   - 包含一个 Access-Control-Request-Method 头部，将请求方法作为头部域的值（就算它是一个简单方法）。
   - 如果用户请求头部不是空的，包含一个 Access-Control-Request-Headers 头部，将作者请求头部的头部域名称使用逗号分隔，按字典序排序作为值，每一个转化为 ASCII 小写（就算一个或者多个是简单头部）。
   - 排除作者请求头部
   - 排除用户凭证
   - 排除请求实体正文
   下面 request rules 在创建预检请求的时候被观测：
   - 如果终端用户取消请求：应用取消步骤
   - 如果响应的 HTTP 状态码不是 2xx 范围，应用网络错误步骤
    > 注意：缓存和网络错误步骤不再这里使用，因为这里正要有一个真实网络错误。
   - 如果有一个网络错误，DNS 错误，TLS 协商是啊比，或者其他类型的网络错误，应用网络错误步骤。不要请求任何类型的终端用户交互。
   > 注意：这不包含指示一些类型的错误的额 HTTP 响应，比如状态码为 410 的 HTTP 状态。
    > 注意：缓存和网络错误步骤不再这里使用，因为这里正要有一个真实网络错误。
   - 否则（HTTP 状态码在 2xx 区间内）
    1. 如果资源分享检测返回失败，应用缓存和网络错误步骤
    2. 让 method 为空列表
    3. 如果有一个或者多个 Access-Control-Allow-Method 头部，让 methods为转化头部的结果。如果转化失败，应用缓存和网络错误步骤。
    4. 如果 methods 还是空的列表并且强制预检标志被设置，拼接请求方法到 methods。
    > 注意：这确保预检请求能完整的发生，因为强制预检标志也被缓存了。
    5. 让 headers 为空的列表
    6. 如果有一个或者多个 Access-Control-Allow-Headers 头部，让 headers 为转化头部的结果的值。如果转化失败，应用缓存和网络错误步骤
    7. 如果请求方法不是大小写敏感命中 methods 中的任何方法，并且不是简单方法，应用缓存和网络错误步骤。
    8. 如果作者请求头部中的每一个头部不是 ASCII 大小写不敏感命中 headers 中的任何一个头部域名称，并且头部不是一个简单头部，应用缓存和网络错误步骤。
    9. 如果因为一些原因，用户代理无法提供预检结果缓存（比如，因为硬盘空间限制），去整个步骤集合的下一步（比如，真实请求）。
    10. 如果有一个单独的 Access-Control-Max-Age 头部，转化他并让 max-age 为结果值。如果没有这些头部，或者多余一个头部，或者转化失败，让 max-age 为用户代理决定的结果值（0也是允许的）。
    11. 使用 method 遍历 methods ，如果方法缓存命中，则设置命中条目的 max-age 为 max-age。使用 method 遍历 methods，如果方法缓存没有命中，在预检结果缓存创建一个新的条目，并使用瞎买呢的系列值：
    - origin：source origin
    - url：请求 URL
    - max-age： max-age
    - credentials：omit credentials flag 被设置则是 false，否则是 true。
    - method：给定的 method
    - header：空
    12. 使用 header 遍历 headers，如果头部缓存命中，设置命中缓存条目的 max-age 域值为 max-age。使用header 遍历 headers，如果头部缓存没有命中，则在预检结果缓存创建一个新的条目，并使用下列值：
    - origin：source origin
    - url：请求的 URL
    - max-age：max-age
    - credentials：omit credentials flag 被设置则是 false，否则是 true。
    - method：给定的 method
    - header：空
2. 设置跨域请求状态为 prefloght complete。
3. 这是真实请求。应用创建一个请求步骤并在创建请求的时候监控 request rule ：
- 如果响应有状态码 301，302，303，307，或者308，应用缓存和网络错误步骤
- 如果最终用户取消请求，应用放弃步骤
- 如果网路错误，DNS 错误，TLS协商失败，或者其他类型的网络错误，应用网络错误步骤。不要请求任何类型的最终用户交互。
> 注意：这不包括表示错误的 HTTP 响应类型，比如 HTTP 状态码 400。
- 否则，执行资源分享检测，如果返回失败，应用缓存和网络错误步骤。否则如果返回通过，终止这个算法，并设置跨域请求状态为 success。不要真的终止请求。

考虑下面的场景：
1. 用户代理从一个 API 获取请求，比如 XMLHttpRequest，从 source origin http://example.org 使用自定义 XMODIFY 方法执行一个跨域请求，到 http://blog.example/entries/hello-world。
2. 用户代理使用一个 OPTIONS 方法执行一个预检请求到 http://blog.example/entried/hello-world 并包含 Origin 和 Access-Control-Request-Method 头部和适当的值。
3. 请求的响应包含下面的头部：
```
Access-Control-Allow-Origin: http://example.org
Access-Control-Max-Age: 2520
Access-Control-Allow-Method: PUT, DELETE, XMODIFY
```
4. 用户代理 使用 XMODIFY 方法执行期待的请求到 http://blog.exmaple/entried/hello-word 因为资源是允许的。此外在接下来的42分钟，不再需要预检请求。

### 7.1.6 预检结果缓存

就像提到的，一个带预检的跨域请求使用预检结果缓存。这个缓存由一系列条目组成，每一个条目由下面域构成：
- origin：保存 source-origin
- url：保存请求 URL
- max-age：保存 Access-Control-Max-Age 头部值。
- credentials：如果 omit credentials flag 被设置，则是 false，否则就是 true
- method：如果 header 是空的，则是空的；否则是 Access-Control-Allow-Methods 头部值中的一个。
- header：如果方法是空的，否则就是 Access-Control-Allow-Headers 头部值中的一个。
> 注意：为了清晰。method 和 header 域都是唯一的。如果其中一个是空的，则另一个不是空的。
> 注意：条目的主键由除了 max-age 之外的域构成。

条目必须从存储条目开始在 max-age 域指定的时间过去以后移除。条目也可以被下面的每一个算法添加或者移除。通过这种方式添加或者移除他们在缓存中不会重复。

用户代理可能在 max-age 域指定的时间过去之前晴空缓存条。
> 注意：尽管这个效果让预先检结果缓存可选，强烈推荐用户代理去支持他。

### 7.1.7 通用跨域请求算法
通用步骤集合中使用的变量是调用这些步骤集合的算法的一部分。
每当创建请求步骤被应用的时候，从 origin 的 source origin 获取请求 URL，使用 referrer source 作为 override referrer source 并设置 manual redirect flag。使用请求方法作为方法，请求实体正文作为实体正文，包含作者请求头部，如果 omit credentials flag 未被设置，则包含用户凭证。

当重定向步骤被应用的时候，执行下面的步骤集合：
1. 让 original URL 为 请求 URL。
2. 让请求 URL 为 重定向响应头部中 Location 头部指定的 URL，
3. 如果请求 URL <schema> 不支持，无限循环预防被潍坊，或者用户代理因为一些原因不像去创建一个新的请求，应用一个网络错误步骤。
4. 如果请求 URL 包含 userinfo 产生式，则应用网络错误步骤
5. 如果当前资源的资源分享检测返回失败，应用网络错误步骤。
6. 如果请求 URL的域和 original URL origin 不是同源，设置 source origin 为全局唯一标识（传播的时候是“null”）
7. 遵循 request rules 的时候透明的遵循重定向。

当放弃步骤应用的时候，终止执行当前集合步骤的算法，并设置跨域请求状态为 abort error。

当放弃步骤应用的时候，终止执行当前集合步骤的算法，并设置跨域请求状态为 network error。
> 注意：这这对设置用户凭证没有任何作用给。比如，如果 block cookies flag 未设置，cookie 将会被响应设置。

当缓存和网络错误步骤被应用，执行下面的步骤：
1. 从预检结果缓存中删除条目，如果条目的 origin 域值大小写铭感的命中 source-origin 并且 url 域值大小写敏感命中请求 URL。
2. 应用网络错误步骤，就像算法调用缓存和网络错误步骤调用网络错误步骤。

如果下面有一个为真，则说明哟一个缓存条目在预检结果缓存中被命中：
- origin 域值是大小写敏感命中 source origin
- url 域值大小写敏感命中请求 URL
- credentials 域值是 true，并且 omit credentials flag 未设置，或者是 false 并且 omit credentials flag 被设置。

如果有一个缓存条目缓存命中并且 method 域值和给定方法大小写敏感命中，则称为方法缓存命中

如果有一个缓存条目缓存命中并且 header 域值和给定头部 ASCII 大小写敏感命中，则称为方法缓存命中

### 7.2 资源分享检测
面对给定资源的资源分享检测算法如下：

1. 如果响应包含0个或者多余一个 Access-Control-Allow-Origin 头部值，返回失败并且终止算法。
2. 如果 Access-Control-Allow-Origin 头部值是“*”字符并且 omit credentials flag 被设置，返回通过并终止这些算法。
3. 如果 Access-Control-Allow-Origin 的值不是大小写敏感命中 Origin 头部的值，就像这个规格定义，返回失败并终止这些算法。
4. 如果 omit credentials flag 未设置，并且 Access-Control-Allow-Credentials 头部值不是大小写命中“true”，返回失败并终止算法。
5. 如果 omit credentials flag 未设置并且 Access-Control-Allow-Credentials 头部值不是大小写敏感命中“true”，返回失败并且终止这个算法。
6. 返回通过。
> 注意：上面的算法在 origin 是字符串“null”的时候也生效。

### 7.3 安全
这个章节是非规范的。

在很多地方，用户代理允许采取额外的预防措施。比如，用户代理允许不去缓存条目，移除缓存条目在他们到达他们的 max-age 之前，和不去链接确定的 URL。

鼓励用户代理去对 max-age 做强制限制，所以条目不会在预检结果缓存中存在不合理的时间。

就像跨域请求算法的第一步表明的那先，在重定向步骤算法，用户代理允许去终止算法并不创建一个请求。这可以完成是因为：
- 资源的 origin 的黑名单
- 资源的 origin 被认为是互联网的一部分
- URL <schema> 不被支持
- htpps 到 http 是不允许的
- https 不允许应为证书错误

鼓励用户代理在普通级别去应用安全决策。而不指示资源分享策略。比如，如果用户代理不允许跨域请求请求从 https 到 http 方案，对 HTML img 元素也这么推荐。


### 8 CORS API 规格建议
整个章节是非规范的。

这个规格定义了一个资源分享策略，没有一个 API 利用他将无法实现。定义使用这个策略的规格是 CORS API 规格。

如果一个 CORS API 规范定义了多个使用这个策略的 API，则必须分贝为每个 API 考虑建议。

### 8.1 创建一个跨域请求
这个规格定义的资源共享策略支持的 API 够可以创建跨域请求，CORS API 规格需要引用跨域请求算法，并适当设置下面输入变量：请求 URL，请求方法，作者请求头部，请求实体正文，source origin，manual redirect flag，omit credentials flag，和 force preflight flag。

CORS API 规范允许让这些输入变量被 API 控制，但也可以设置固定的值
> 一个 CORS API 规范的 API 值允许使用 GET  方法可能设置方法为 GET，请求实体正文为空，source origin 为一些适当的值，让其他变量被 API 控制。

### 8.2 处理同源到跨域重定向

因为浏览器基于同源安全模型，并且这个规格的策略适用于浏览器的 API，可预见的，这些将会利用这个策略 API 将必须处理一个将会导致跨域重定向的同源请求，在某个特殊的方式。

对于透明处理重定向的 API 的 CORS API 规范，鼓励去处理这种场景透明的就像捕获重定向并且在重定向 URL 调用跨域请求算法（跨域）。
> 注意：XMLHttpRequest 规格就是这么做的。

### 8.3 处理跨域请求状态
当跨域请求处理他关联的跨域请求状态被更新。依赖于 API 在不同方式交互的跨域请求状态的值：
- preflight complete：只能在预检请求后才能安全暴露出来的安全特性现在可以启用了。
> 注意：比如，XMLHttpRequest 上传进度事件。
- success：响应的内容可以被 API 分享，包含没有被过滤的头部
> 注意：请求本身还是可以被处理。比如，跨域请求状态值并不指示请求已经完成了。
- abort error：处理类似的用户放弃请求的请求。和处理 network error 类似。确保不要暴露关于这个请求更多的信息
- network error：处理类似发生某种错误的请求。确保不要暴露关于这个请求更多的信息

### 8.4 安全

和同源请求类似，CORS API 正确的限制头部，方法，并且用户凭证可以被作者设置和获取
> 复习 XMLHttpRequest 规格提供了一个强制实施这类限制的好的开始。
CORA API 规格同样需要确保不要泄漏任何东西，直到跨域请求状态被设置为 preflight compelte 后者 success，去防止，比如，端口扫描。
> 注意：在 XMLHttpRequest 进度事件只在跨域请求状态被设置为成功以后才分发，上传进度事件只在跨域请求状态为 preflight complete 的时候分发。

