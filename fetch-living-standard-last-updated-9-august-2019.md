### 概述

Fetch 标准定义请求，响应，和绑定他们的过程：获取。

### 目标

为了跨越 web 平台统一获取，这个规格取代了大量的算法和规格

- HTML 标准的获取和潜在的启用 CORS 的获取算法。
- CORS
- HTTP `Origin`头部语意

统一的获取提供一致的处理：

- URL 方案
- 重定向
- 跨域语法
- CSP
- Service Worker
- Mixed Content
- `Referer`

### 1. 序言

从高层次来看，获取一个资源是一个非常简单的操作。一个请求出去，一个响应回来。然而这个操作的细节却非常复杂，过去并没有小心的写下来，导致每一个 API 都不一样。

大量的 API 提供了获取资源的能力，比如，HTML 的 img 和 script 元素，CSS 的 cursor 和 list-style-image，navigator.sendBeacon() 和 self.importScripts() JavaScript API。Fetch 标准为这些特性提供了一个统一的架构，让他们在涉及到获取的各个方面时，保持一致性，比如重定向和 CORS 协议。

Fetch 标准也定义了 fetch() JavaSript API，它在相当低底层的暴露了大量网络功能的概念。


### 2. 基础设施

这个规则依赖于 Infra 标准。

这个规格使用的术语来自 ABNF，Encoding，HTML，IDL，MIME sniffing，Streams，和 URL 标准。

ABNF 表示由 HTTP 和 RFC 7405 增强的 ABNF。

Credentials 是 HTTP的 cookies，TLS 客户端认证，和 authentication entried（HTTP 认证）

这个标准声明的任务队列如下：
- 处理请求体
- 处理请求体结束
- 处理响应
- 处理响应体结束
- 处理响应结束

在请求 request 上对获取任务排队以运行一个操作，需要执行以下步骤：

1. 如果 request 的 client 是 null，终止这些步骤。

2. 在 request 的 client 负责的事件队列入队一个任务去运行一个操作使用网络任务源。

在给定的 request 上，将一个获取任务入队，处理请求体结束，获取请求完成任务入队。

为了序列化一个数字，使用最短的十进制数字字符串来表示。

> 这将会被 Infra 中描述更加具体的算法替代。

### 2.1 URL

如果一个方案是“about”，“blob”，或者”data“，那么这个方案是本地方案。

如果一个 URL 的方案是一个本地方案，那么它是本地的。

> 注意：这个定义也被用于 Referer 策略。

如果一个方案是“http”或者“https”，则它是 HTTP(S) 方案。

如果一个方案是“ftp”或者是 HTTP(S) 方案，则它是网络方案。

如果一个方案是“about”，“blob”，“data”，“file”，或者一个网络方案，则它是 获取方案。

> HTTP(S) 方案，网络方案，和获取方案都被使用于 HTML。

响应 URL 是不需要存储片段的 URL，就像他从来没有存在。当序列化的时候，设置排除片段标志，这意味着实现可以存储片段。


### 2.2 HTTP

尽管获取不止包含 HTTP，它从 HTTP 借用了许多的概念，并将这些引用语其他资源（比如，data URL）。

HTTP 制表符或者空格是 U+0009 TAB 或者 U+0020 SPACE。

HTTP 空白是 U+000A LF，U+000D CR，或者一个 HTTP 制表符或者空白。

> HTTP 空白只在 HTTP 头部之外特定的结构有用（比如，MIME 类型）。对于 HTTP 头部值，使用 HTTP 制表符和空白是优先的，在这上下文之外，ASCII 空白 是优先的。不像 ASCII 空白，它包含 U+000CFF

HTTP 新行字节是 Ox0A(LF) 或者 0x0D(CR)。

HTTP 制表符或者空格字节是 0x09(HT) 或者 0x020(SP)。

HTTP 空白字节是一个 HTTP 新行字节或者 HTTP 制表符或者空格字节。

HTTPS 状态值是“none”，“deprecated”，或者“modern”。

> 注意：通过 HTTPS 传递的响应总是有“modern”的 HTTPS 状态。用户代理可以在过渡期使用“deprecated”。比如，当移除对哈希函数，弱加密套件，“内部名称”验证，或者过长有效事件认证的支持。至于用户代理具体可以如何使用“deprecated”并没有在这个规格内定义。一个环境设置对象的 HTTPS 状态通常来一个响应。

从一个字符串 input 收集 HTTP 引用字符串，给定定位变量 position，和可选的 extract-value flag，运行如下步骤：

1. 让 positionStart 为 position。
2. 让 value 为空字符串。
3. 断言：input 的 position 位置的码点为 U+0022(")。
4. position 自增 1。
5. while true：
    1. 给定 position，从 input 收集一些列非 U+0022(") 的码点，拼接结果到 value。
    2. 如果 position 是 input 的结束，则 break。
    3. 让 quoteOrBackslash 为 input 内 position 的码点。
    4. position 自增 1。
    5. 如果 quoteOrbackslash 是 U+005C(\)，则：
        1. 如果 position 是 input 的结束，则拼接 U+005C(\) 到 value，然后 break。
        2. 拼接 input 的 position 处的码点到 value。
        3. position 自增 1。
    6. 否则
        1. 断言：quoteOrBackslas 是 U+0022(")。
        2. break。
6. 如果 extract-value flag 设置，则返回 value。
7. 返回 input 从 positionStart 到 positon 的码点，闭区间。

extract-value flag 参数让这个算法更加适用于获取，解码，和分割和转化 MIME 类型，同样其他头部也可能需要这个。

### 2.2.1. 方法

方法是匹配方法令牌产生式的字节序列。

 CROS-安全方法是`GET`、`HEAD`、`POST`方法。

禁止方法是字节大小写不敏感匹配`CONNECT`、`TRACE`、`TRACK`的方法。

规范化一个方法是将字节大小写不敏感匹配`DELETE`，`GET`，`HEAD`，`OPTIONS`，`POST`，`PUT`方法转化成比特大写。

> 注意：规范化其实是为了向后兼容性和跨 API 一致性，因为方法其实是“大小写敏感的”。

> 栗子：使用`patch`很可能导致`405 Method Not Allowed`，`PATCH`更有可能成功。

> 注意：方法没有限制。`CHICKEN`完美可接收（不是`CHECKIN`的错误拼写）。除了标准化的那些之外，也没有大小写的限制。`Egg`或`eGg`也可以，尽管鼓励一致大写。

### 2.2.2 头部

头部列表是一个或多个头部的列表。它初始化为一个空的列表。

> 注意：头部列表本质上是一个专用的多映射。一个有序的键值对列表，包含潜在的重复的 key。

头部列表 list 包含一个名字 name，如果 list 包含一个头部，它的名字是字节不敏感的匹配 name。

从一个头部列表 list 获取一个名字 name，执行以下步骤：

1. 如果 list 不包含 name，则返回 null。
2. 有序返回 list 中所有名字大小写不敏感匹配 name 的头部的值，使用 0x2C，0x20分割。

从头部列表 list 获取，解码，和分割一个名字 name，执行下面步骤：

1. 让 initialValue 为从 list 获取 name 的结果。
2. 如果 initialValue 是 null，就返回 null。
3. 让 input 为 initialValue 的同构解码结果。
4. 让 position 为 input 的索引值，初始化为 input 的开始。
5. 让 values 为一个字符串列表，初始化为空。
6. 让 value 为空字符串。
7. 当 position 不是 input 的结尾：
    1. 根据给定 position 从 input 收集一系列非 U+0022(") 或者 U+002C(,) 的码点，将之拼接到 value。
    > 结果可能是空字符串。
    2. 如果 position 不是 input 的结尾，那么：
        1. 如果 input position 位置的码点是 U+0022(")，则：
            1. 根据给定 position，从 input 中收集 HTTP 引用字符串，将结果拼接到 value。
            2. 如果 position 不是 input 的结尾，则 continue。
        2. 否则：
            1. 断言：input position 处的码点是 U+002C(,)。
            2. position 自增 1。
    3. 移除 value 中所有的 HTTP 制表符和空白。
    4. 拼接 value 到 values。
    5. 设置 value 为空字符串。
8. 返回 values。

栗子：这是获取，解码，和分割函数在`A`作为 name 参数的时候的实践：

| 头部（就像在网络中） | 输出 |
| - | - |
| A: nosniff, | "nosnoff","" |
| A: nosniff\n B: sniff\n A: |  |
| A: text/html;",x/x | "text/html;",x/x" |
| A: test/html;"\n A: x/x |  |
| A: x/x;test="hi",y/y | "x/x;test="hi"","y/y" |
| A: x/x;test="hi"\n C: \*\*bingo\*\*\n A:y/y |  |
| A: x / x,,,l | "x / x","","","l" |
| A: x / x\n A: ,\n A:l |  |
| A: "1,2", 3 | "1,2", "3" |
| A: "1,2"\n, D: 4\n A: 3 |  |

拼接一个键值对 name/value 到头部列表 list，执行下面的步骤：
1. 如果 list 包含了 name，则设置 name 为第一个这个头部的名字。
> 这重新使用 list 中已经存在的头部的 name 的大小写，如果错在多个相同的头部，则这些头部的名字都一样。
2. 拼接名字为 name，value 为 value 的新头部到 list。

从一个头部列表 list 删除一个名字 name，移除 list 中所有名字字节大小写不敏感匹配 name 的头部。

在头部列表 list 中设置一个键值对 name/value，执行以下步骤：
1. 如果 list 包含 name，则设置第一个这个头部的值为 value，并移除其他的，
2. 否则，拼接一个 name 为 name，value 为 value 的新头部到 list。

在头部列表 list 中绑定一个键值对 name/value，执行以下步骤：
1. 如果 list 包含 name，则设置第一个该头部的值为它的值，接着是 0x2C 0x20，接着是 value。
2. 否则，拼接一个 name 为 name，value 为 value 的新头部到 list。

绑定用在 XMLHttpRequest 和 WebSocket 协议握手。

给定一个列表名字是 headerNames，将头部名字转化有有序小写集合，执行以下步骤：
1. 让 headerNamesSet 为一个新的有序集合。
2. 以 name 遍历 headerNames，拼接 name 的字节小写转化结果到 headerNamesSet。
3. 将 headerNamesSet 按字节升序排序返回。

排序并绑定头部列表 list，执行以下步骤：
1. 让 headers 为空的键值对 list，让 key 为 name，value 为 value。
2. 将 list 中的所有头部的名字转化为小写有序集合，并放到 names。
3. 以 name 遍历 names：
    1. 从 list 中获取 name，并放到 value 中。
    2. 断言：value 不是空的。
    3. 拼接 name-value 到 headers。
4. 返回 headers。

一个头部由 name 和 value 构成。

一个 name 是一个字节序列，它匹配域名token生产式。

一个 value 是一个字节序列，它满足下列条件：

- 不以 HTTP 制表符和空白字节开始或结尾。
- 不包含 0x00(NUL) 或者 HTTP 新行字节。

> 注意：value 的定义不是根据 HTTP 的token生产式来定义的，因为他已经被打破了。

规范化一个 potentialValue，移除 potentialValue 开始或者结尾任何 HTTP 空白字节。


确定一个头部 header 是 CROS-安全请求头部，执行以下步骤：
1. 让 value 为 header 的值。
2. 将 header 的名字转化为字节小写，根据结果分别执行：
    - `accept`：如果 value 包含 CORS 不安全请求头部字节，则返回 false。
    - `accept-language`
    - `ocntent-language`：如果 value 包含不在 0x30(0)-0x39(9)  闭区间内，不在 0x41(A)-0x5A(Z)闭区间内，不在 0x61(a)-0x7A(z) 区间内，不是 0x20(Sp)，0x2A(*)，0x2C(,)，0x2D(-)，0x2E(.)，0x2E(,)，0x3B(;)，或者 0X3D(=)，则返回 false。
    - `content-type`：
        1. 如果 value 包含 CORS 不安全请求头部字节，则返回 false。
        2. 让 mimeType 为 value 的转化结果。
        3. 如果 mimeType 失败，则返回 false。
        4. 如果 mimeType 不是“application/x-www-form-urlencodede”，“multipart/form-data”，“text/plain”，则返回 false。
        > 警告：这里特意不使用解析 MIME 类型，因为这个算法相当宽容，并且不期望服务器去实现它。
        > 栗子：如果加息 MIME 类型被用在瞎买呢的请求，将会导致一个 CROS 预请求，并且幼稚的服务器解析将会吧请求体当作 JSON
        ```
        fetch("https://victim.example/naïve-endpoint", {
            method: "POST",
            headers: [
                ["Content-Type", "application/json"],
                ["Content-Type", "text/plain"]
            ],
            credentials: "include",
            body: JSON.stringify(exerciseForTheReader)
        });
        ```
    - 否则：返回 false。
3. 如果 value 的长度大于 128，则返回 false。
4. 返回 true。

> 注意：在`Content-Type`头部安全列表有几个异常，记录在文档 CROS 协议异常。

一个 CORS 不安全请求头部字节 byte 满足下面的一个要求：
- byte 小于 0x20 并且不是 0x09 HT。
- byte 是 0x22(")，0x28(左括弧)，0x29(右括弧)，0x3A(:)，0x3C(<)，0x3E(>)，0x3F(?)，0x40(@)，0x5B(\[)，0x5C(\\)，0x5D(\])，0x7B({)，0x7D(})，或者 0x7F DEL。

给定头部列表 headers，确定它是 CROS 不安全请求头部名，执行以下请求：
1. 让 unsafeNames 为一个新的列表。
2. 让 potentiallyUnsafeNames 为一个新的 列表。
3. 让 safelistValueSize 为 0。
4. 使用 header 遍历 headers：
    1. 如果 header 不是一个 CORS 安全请求头部，则拼接 header 的名字到 unsafeNames。
    2. 否则，拼接 header 名字到 potentiallyUnsafeNames 并增加 safelistValueSize 
5. 如果 safelistValueSize 大于 1024，则遍历 potentiallyUnsafeNames 的 name，拼接 name 到 unsafeNames。
6. 将 unsafeNames 转化为有序的小写集合，并返回。

一个 CROS 非通配符请求头部是小写不敏感的`Authorization`。

一个 特权的非 CORS 请求头部名是下面的其中一个(大小写不敏感)：
- `Range`。

> 注意：这些头部可以被授权 API 设置，并且如果请求被复制则会被保留，如果被未授权的 API 设置，则将会被移除。
> `Range`头部被用来下载和媒体查询，但是目前并没有被指定。html/2914 目的就是解决这个。
> 一个助手被用来为特殊的请求添加区间头部。

给定一个 CORS 暴露的头部名列表 list，如果是 CORS 安全响应头部名，字节大小写不敏感的匹配以下一个：
- `Cache-Control`
- `Content-Language`
- `Content-Length`
- `Content-Type`
- `Expires`
- `Last-Modified`
- `Pragma`
- 列表中的任意一个值不是禁止的响应名。

一个非 CORS 安全请求头部名是下面中的一个（大小写不敏感）：
- `Accept`
- `Accept-Language`
- `Content-Language`
- `Content-Type`

判断一个头部 header 是一个 非 CORS 安全请求头，执行以下步骤：
1. 如果 header 的名字不是一个 非 CORS 安全请求头部名字，则返回 false。
2. 返回 header 是否是一个 CORS 安全请求头

一个禁止头部名称是下面其中一个（大小写不敏感）：
- `Accept-Charset`
- `Accept-Encoding`
- `Access-Control-Request-Headers`
- `Access-Control-Request-Method`
- `Connection`
- `Content-Length`
- `Cookie`
- `Cookie2`
- `Date`
- `DNT`
- `Expect`
- `Host`
- `Keep-Alive`
- `Origin`
- `Referer`
- `TE`
- `Trailer`
- `Transfer-Encoding`
- `Upgrade`
- `Via`

或者头部名称用`Proxy-`或者`Sec-`开头（大小写不敏感）（包括直接匹配`Proxy-`或者`Sec-`）。
> 这些是禁止的，所以用户和代理仍然可以完全控制他们，`Sec-`开头的名称是保留的，为了让 fetch 安全的创建新的头部，允许开发这使用 API 控制这些头部，比如 XMLHttpRequest。

一个禁止的响应头部名称是下面的其中一个（大小写不敏感）：
- `Set-Cookie`
- `Set-Cookie2`

给定一个名字 name 和一个头部列表 list，解析头部列表的值，执行以下步骤：
1. 如果 list 不博阿含 name，则返回 null。
2. 如果 name 的ABNF 允许单独的头部，并且 列表包含多个，则返回失败。
> 如果需要不同的错误处理，先解析头部。
3. 让 values 为一个空的列表。
4. 遍历 列表中名字为 name 的头部：
    1. 让 extract 为从 header 解析头部值的结果。
    2. 如果 extract 是失败，则返回失败。
    3. 按顺序拼接 extract 中的每一个值到 values
5. 返回 values。

一个默认的`User-Aget`值是一个用户代理为`User-Agent`头部定义的值。

### 2.2.3 状态

一个状态是一个码

一个没有体的状态是 101，204，205，304。

一个 ok 状态码是 200-299 闭区间中的一个。

一个重定向状态码是 301，302，303，307，308。

### 2.2.4 正文
一个正文由以下组成：
- 一个流（null 或者 ReadableStream 对象）。
- 一个传输的比特（一个整数），初始值为 0。
- 一个总比特（一个整数），初始化为 0。
- 一个源，初始化为 null。

一个正文 body 完成了指的是 body 是空的，或者 body 的流被关闭或者错误。

等待一个正文 body，指的是等待 body 完成。

克隆一个正文 body，执行以下步骤：
- 让 <out1, out2> 为 body 流的结果。
- 设置 body 的流到 out1。
- 返回一个正文，它的流是 out2，其他的成员则是从 body 中复制出来的。

给定 codings 和 bytes，处理内容编码执行以下步骤：
- 如果 codings 不支持，返回 bytes。
- 如果解码没有导致错误，则返回 HTTP 中解释的那样使用 codings 解码 bytes 的结果，否则返回失败。

### 2.2.5 请求

获取的输入是请求

一个请求有一个关联的方法（一个方法）。除非标志否则是`GET`。

> 注意：这可以在重定向中更新为`GET`，就像在 HTTP fetch 中米描述的。

一个请求有一个关联的 URL（一个 URL）。

> 注意：鼓励实现让这个指针为请求的 URL 列表的的第一个 URL。它提供嘟噜的域，让其他标准方面的挂钩到 Fetch。

一个请求有一个关联的本地 URL 专用标志。除非标志否则就是未设置。

一个请求有一个关联的头部列表（一个头部列表）。除非标志，否则就是空的。

一个请求有一个关联的不安全请求标志，除非标志否则就是未设置
> 注意：不安全请求标志被像 fetch() 和 XMLHttpRequest 设置，为了确保一个基于支持的方法和头部列表的 CORS 预请求已经完成了。没有一个自由的 API 释放禁止的方法和禁止的头部名称。

一个请求有一个关联的正文（null 或者一个正文）。除非标志，否则就是 null。
> 这可以在重定向中更新为 null，就像 HTTP fetch 中描述的那样。

一个请求有一个关联的客户端（null 或者一个环境设置对象）。

一个请求有一个关联的保留客户（null，一个环境，或者一个环境设置对象）。除非标志否则就是 null。
> 注意：这只用于导航请求和 worker 请求，但不用于 service worker 请求。它为一个导航请求引用一个环境，为一个 worker 请求引用一个环境设置对象。

一个请求有一个关联的替换客户端 id（一个字符串）。除非标志，否则它是一个空的字符串。
> 这只用于导航请求，它是目标浏览撒谎给你心爱文的活跃文档的环境设置对象的 id。

一个请求有一个关联的 window（“no-window”，"client"，或者一个全局对象是 Window 对象的环境设置对象）。除非设置否则就是“client”。

> 注意：在获取的时候，“client”只改变为“no-window”或者请求的 client。它为标准提供一个方便的方法不必去精确的设置请求的 window。

一个请求有一个关联的 keepalive 标志。除非标志否则是未设置。

> 注意：这可以用来让请求活得比环境设置对象长，比如 navigator.sendBeacon 和 HTML 的 img 元素设置这个标志。设置这个标志的请求需要接收其他的处理需求。

一个请求有一个关联的 service-worker 模式，是“all”或者“none”。除非标志，否则就是“all”。
> 注意：这决定哪个 service worker 将会接收到这个 fetch 的 fetch 事件
- “all”：相关的 service worker 都会接收到这个 fetch 的 fetch 事件。
- “none”：没有 service worker 将会接收到这个 fetch 的 fetch 事件。

一个请求有一个关联的初始标志，它是一个空字符串，“download”，“imageset”，“manifest”，“prefetch”，“prerender”，或者“xslt”。除非标志，否则就是空字符串。

> 注意：一个请求的初始化指示器不是专门用来

### 2.2.6 响应
获取的结果是响应。一个响应总是变化。这意味着，不是所有的域可以直接获取。

一个响应有一个关联的类型，它是“basic”，“cors”，“default”，“error”，“opaque”，或者”opaqueredirect“，除非标志，否者它是”default“。

一个响应有一个关联的放弃标志，它初始值未设定。
> 这意味着请求想要被开发这或者终端用户终止。

一个响应有一个关联的 URL。它指向响应 URL 列表的最后的响应 URL，如果响应的 URL 列表不是空列表。

一个响应有一个关联的 URL 列表（一个包含0个或者多个的响应 URL 列表）。除非标志，否则他就是一个空的列表。

> 注意：除了最后一个响应 URL，任何一个响应 URL 列表可以被暴露给脚本。这将会违反自动 HTTP 重定向处理。

一个响应有一个关联的状态，它是一个状态除非标志，否则就是 200。

注意：HTTP/2 连接的响应将会有一个始终为空比特序列的作为状态信息，因为 hTTP/2 不支持他们。

一个响应有一个关联的头部列表（一个头部列表），除非标志否则就是空的。

一个响应有一个关联的正文（null 或者正文）。除非标志，否者就是null。

一个响应有一个关联的 trailer（一个头部列表）。除非标志否者就是空的。

一个响应有一个关联的 trailer filed flag，它初始化为未设置。

一个响应有一个关联的缓存状态（空字符串或者“local”）。除非标志，否者它是一个空字符串。


> 注意：这个是专门给 service worker 用的。

一个响应有一个关联的 HTTPS 状态（一个 HTTP 状态值）。除非标志，否则它是“none”。

一个响应有一个关联的 CSP 列表。它是一个响应的内容安全策略对象。除非指定，否则这个列表是一个空的。

一个响应有一个关联的 CORS-暴露头部名称列表（一个0个或者多个头部名称列表）。除非指定，否者它是一个空的列表。

> 注意：一个响应通常通过从`Access-Control-Expose-Headers`头部解析头部值来获取它的暴露的 CORS 头部名称列表。

一个响应有一个关联的范围请求标志，它初始化为未设置。

> 注意：这用来确保防止从一个更早的范围请求的一个部分响应，提供给一个无法发送范围请求的 API。查看标志的使用攻击的详细描述。

一个响应可以有一个关联的定位 URL（null，failure，或者 URL）。除非指定，否则，响应没有定位 URL。

这个概念用在 FETCH 和 HTML 导航算法的重定向处理。它确保`Location`有它解析的值并且只有一次。

一个类型是“error”的响应并且放弃标志被设置，它是一个放弃的网络错误。

一个类型是“error”是一个网络错误。

一个网络错误是一个响应，它的状态总是 0，状态信息总是空的字节流，头部列表是一个空的，正文总是 null，并且 trailer 总是空的。

一个过滤的响应是一个响应上的受限视图，它不是网路错误，这个响应作为过滤响应的关联的内部响应的引用。

> 注意：获取算法返回这么一个视图为了保证 API 不会意外泄露信息。如果信息因为历史原因需要被暴露，比如，条虫图片数据去解码，关联的内部响应可以使用，只有“accessible”内部规格算法并且它永远不会是一个过滤的响应。

一个基本的过滤响应是一个过滤响应，它的类型是“basic”，并且头部列表不包括任何内部响应的头部列表的名称在禁止响应头部名称的头部。

一个 CORS 过滤响应是一个过滤响应，它的类型是“cors”，头部列表不包含任何内部响应的头部列表的名称不再 CORS 安全响应头部名的头部，给定内部响应的 CORS 暴露的头部名字列表，它的 triler 是空的。


一个透明的过滤响应是一个过滤响应，它的类型是“opaque”，URL 列表是一个空的列表，状态是0，状态信息是空的字节序列，头部列表是空的，正文是 null，并且 trailer 是空的。

一个透明的重定向过滤响应是一个过滤响应，它的类型是“opaqueredirect”，状态是0，状态信息是空字节序列，头部列表是空的，正文是 null，并且 trailer 是空的。

> 注意：暴露一个 URL 列表给透明的重定向过滤响应是无害的，因为没有重定向将会被执行。
> 换句话说，一个透明的过滤的响应和一个透明的重定向过滤响应和网络错误是难以区分的。当引入一个新的 API，不要在内部规格算法使用内部响应，因为它将会泄露信息。
> 这也意味着 JavaScript 的 API，比如 response.ok，将会返回无用的结果。

克隆一个响应 response，执行下面的步骤：
1. 如果 respnnse 是一个过滤的响应，则返回一个新的相同的响应，并且它的内部响应是克隆 response 的内部响应。
2. 让 newResponse 为 response 的克隆，除了它的正文。
3. 如果 response 的正文不是 null，则设置 newResponse 的正文为克隆 response 正文的结果，
4. 返回 newResponse。

一个新鲜的响应是当前年龄在新鲜生命时间内的响应

一个过期需要重新验证的响应不是一个新鲜的响应并且当前年龄在过期需要验证生命时间内的响应。

一个过期的响应不是一个新鲜的响应或者过期需要重新校验的响应。

### 2.2.7 


### 2.3 验证入口

### 2.4 获取组

### 2.5 连接

### 2.6 端口堵塞

### 2.7 请求的响应会因为 MIME 类型堵塞吗？

### 2.8 流
> 注意：这个章节可能嵌入其他标准，比如 IDL

### 2.8.1 ReadableStream

一个 ReadableStream 对象是一个数据流的表示。在这个章节，我们定义 ReadableStream 对象的基本操作。

将 chunk 入队到一个 ReadableStream 对象 stream，执行这些步骤：
1. 调用 ReadableStreamDefaultControllerEnqueue(stream,[[readableStreamController]], chunk)。

关闭一个 ReadableStream 对象 stream，执行这些步骤：
1. 调用 ReadableStreamDefaultControllerClose(stream, [[readableStreamController]])。

使用给定 readon，让一个 ReadableStream 对象 stream 报错，执行以下步骤：
1. 调用 ReadableStreamDefaultControllerError(stream,[[readableStreamController]],reason)。

使用可选的 highWaterMark，sizeAlgorithm 算法，pull 动作，和 cancel 动作，执行这些步骤：

> 注意：这个算法曾经使用 strategy 参数，highWaterMark 和 sizeAlgorithm 成员就是从中提取，现在分离的参数。如果其他规则还使用 strategy 参数，请更新它。

1. 让 startAlgorithm 为返回 undefined 的算法。
2. 如果 pull 缺省，则这是它为返回 undefined 的动作。
3. 让 pullAlgorithm 为返回 promise 调用 pull() 结果的算法。
4. 如果 cancel 缺省，则设置它为返回 undefined 的动作。
5. 让 cancelAlgorithm 为接收 reason，返回 promise 调用 cancel(reason)) 的结果的算法。
6. 如果 highWaterMark 缺省，则设置它为1。
7. 如果 sizeAlgoorithm 缺省，则设置它为返回1的算法。
8. 返回 CreateReadableStream(startAlgorithm, pullAlgorithm, cacelAlgorithm,highWaterMark,sizeAlgorithm)。

使用 chunks 创建一个固定的 ReadableStrem 对象，执行以下步骤：
1. 让 stream 为构造一个 ReadableStream 对象的结果。
2. 使用 chunk 遍历 chunks，将 chunk 入队到 stream。
3. 关闭 stream。
4. 返回 stream。

从一个 ReadableStream 对象 stream 获取一个 reader，执行以下步骤：
1. 让 reader 为调用 AcquireReadableStreamDefaultReader(stream) 的结果。
2. 返回 reader。

使用 reader 从一个 ReadableStream 对象读取一个 chunk，返回调用 ReadableStreamDefaultReaderRead(reader) 的结果。

使用 reader 从一个 ReadableStream 对象读取所有的字节，执行以下步骤：
1. 让 promsie 为一个新的 promise。
2. 让 bytes 为空的字节序列。
3. 让 read 为调用 ReadableStreamDefaultReaderRead(reader) 的结果。
    - 如果 read 是 fullfilled，并且它的 done 属性是 false，它的 value 属性是 Uinit8Array 对象，拼接 value 属性 到 bytes 并再次执行上一个步骤。
    - 如果 read 是 fullfilled，并且它的 done 属性是 true，让 promsie 使用bytes resolve。
    - 如果 read 是 fullfilled，并且它的 value 和上面都不匹配，则让 promise 使用一个 TypeError reject。
    - 如果 read 是被一个 error rejected，则使用这个 error 让 prommise reject。
4. 返回 promise。

使用 reason 取消一个 ReadableStream 对象 stream，返回调用 ReadableStreamCancel(stream, reason) 的结果。

读取一个 ReadableStream 对象 stream，执行下面步骤：
1. 返回调用 ReableStreamTee(stream, true) 的结果。

使用空的列表构建一个固定的 ReadableStream 对象的结果是一个空的 ReadableStream 对象。

> 构造一个空的 ReadableStream 对象不会抛出一个异常。

一个 ReadableStream 对象 stream 是可读的说明 stream.[[state]] 是 “readable”。

一个 ReadableStream 对象 stream 是关闭的说明 stream.[[state]] 是"closed"。

一个 ReadableStream 对象 stream 是错误的说明 stream.[[state]] 是"errored"

一个 ReadableStream 对象 stream 是锁住的说明调用 isReadableStreamLocked(stream) 的结果是 true。

一个 ReadableStream 对象 stream 需要更多的数据说明以下的条件被命中了：
- stream 是可读的。
- 调用 ReadableStreamDefaultControllerGetDisiredSize(stream.[[readabaleStreamController]]) 的结果是正的。

一个 ReadableStream 对象 stream 是乱的，说明调用 isReadableStreamDisturbed(stream) 返回为 true。



### 3. HTTP 扩展

### 3.1 `Origin`头部

`Origin`请求头标志获取的来源。

> 注意：`Origin`头部是`Referer`[sic]头部的一个版本，它不披露路径。它用在所有 CORS 标志被设置并且请求的方法既不是`GET`也不是`HEAD`的HTTP 获取。因为兼容约束，它没有包含在所有的请求中。

它的值的 ANBF：
```
    Origin          = origin-or-null
    origin-or-null  = origin / %s"null" ; case-sensitive
    origin          = scheme "://" host [ ":" port ]
```

> 注意：这取代了`Origin`头部。

给定一个请求 request，和一个可选的 CORS flag，执行以下步骤。
1. 让 serializedOrigin 为为 request 序列化一个请求源的结果。
2. 如果 CORS flag 被设置，或者请求的模式是“websocket”，则拼接`Origin`/serializedOrigin 到请求的头部列表。
3. 否则，如果 request 的方法不是`GET`，也不是`HEAD`，则：
    1. 根据 request 引用策略执行：
        - “no-referer”：设置 serializedOrigin 为`null`。
        - “no-referer-when-downgrade”
        - "strict-origin"
        - "strict-origin-when-cross-origin"：如果 request 的源是一个源元组，它的 scheme 是“https”，并且 request 当前的 URL schema 不是“https”，则设置 serializedOrigin 为`null`
        - "same-origin"：如果请求的源不是同源的，并且 request 的当前 URL 是源，则设置 serializedOrigin 为`null`
        - 否则：啥也不做
    2. 拼接 `Origin`/serializedOrigin 到 request 的头部列表。

> 注意：一个请求的引用策略被所有并没有被明确指出和服务端分享他们的源的请求考虑。比如，通过使用 CORS 协议。

### 3.2 CORS 协议
为了允许响应跨域并允许 HTML 的 from 元素有更多可能的获取，CROS 协议存在。它在 HTTP 的上层，允许响应声明他们可以分享给其他域。

> 注意：这需要有一种选择机制，防止泄露防火墙（内网）中的响应信息。此外，包含认证的请求需要选择去防止泄露潜在的敏感信息。

本章节解释了 CORS 协议，因为它和服务端开发者有关。用户代理相关的是部分获取算法，还有新的 HTTP 头部语法。

### 3.2.1 概述
CORS 协议由一系列头部组成，他们指出一个响应如何跨域分享。

对于比 HTML form 元素更复杂的请求，一个 CORS 预请求将会发送，为了保证当前的 URL 支持 CORS 协议。

### 3.2.2 HTTP 请求

一个 CORS 请求是一个 HTTP 请求，它包含`Origin`头部。不能用它来可靠的识别参与了 CORS 协议，因为`Origin`头部也包含在所有方法不是`GET`也不是`HEAD`的请求中。

一个 CORS 预请求是一个 CORS 请求，用来检查 CORS 协议是否被理解。它使用`OPTIONS`作为方法，包含这些头部：

- `Access-Control-Request-Method`：标志那些方法在接下来的 CORS 请求可能会在相同的资源中使用。
- `Access-Control-Request-Headers`：标志那些头部将会在接下来的 CORS 请求可能会在相同的资源中使用。

### 3.2.3 HTTP 响应

一个 CORS 请求的 HTTP 响应可以包括以下头部：
- `Access-Control-Allow-Origin`：标志那些响应是否可以被分享，通过返回`Origin`的直接值，请求的头部（可能是`null`）或者`*`。
- `Access-Control-Allow-Credentials`：标志当请求的认证模式是“include”的时候，响应是否可以被分享。
> 注意：对于一个 CORS 预请求，请求的认证模式总是“omit”，但是对于任何接下来的 CORS 请求可能都不是。因此，支持需要指示作为 CORS 预请求的响应的一部分。
- `Access-Control-Allow-Methods`：指示 response 的 URL 支持哪些方法用于 CORS 协议。
> 注意：`Allow`头部和 CORS 协议目的无关。
- `Access-Control-Allow-Headers`：指示 response 的 URL 支持哪些头部用于 CORS 协议。
- `Access-Control-Max-Age`：指示 `Access-Control-Allow-Methods` 和 `Access-Control-Allow-Headers`头部提供的信息可以被缓存多久。

一个 CORS 请求 但是不是 CORS 预请求的 HTTP 响应也可以包含下面的头部：

- `Access-Control-Expose-Headers`：标志那些头部可以暴露作为响应的部分，通过列出他们的名字。

如果服务端不希望参加 CORS 协议，它对于 CORS 或者 CORS 预请求的响应必须不包含上面的头部。鼓励在这些 HTTP 响应中使用 403 状态码。

### 3.2.4 HTTP 新头部语法
CORS 协议中使用的头部的值的 ABNF：
```
    Access-Control-Request-Method       = method
    Access-Control-Request-Headers      = 1#field-name
    
    wildcard                            = "*"
    Access-Control-Allow-Origin         = origin-or-null / wildcard
    Access-Control-Allow-Credentials    = %s"true" ; case-sensitive
    Access-Control-Expose-Headers       = #field-name
    Access-Control-Max-Age              = delta-seconds
    Access-Control-Allow-Methods        = #method
    Access-Control-Allow-Headers        = #field-name
```
> 注意：`Access-Control-Expose-Headers`、`ccess-Control-Allow-Methods`、`Access-Control-Allow-Headers`响应头部的值是`*`则认为是没有认证的请求的通配符。对于这些请求，没有办法单独匹配头部名称或者方法名称是“*”的。

### 3.2.5 CORS 协议和认证

当请求的认证模式是“include”的时候，除了在获取中包含认证，它还对 CORS 协议有所影响。

> 栗子：在过去的日子里，XMLHttpRequest 可以被用来设置请求的认证模式为“include”：
```
var client = new XMLHttpRequest()
client.open("GET", "./")
client.withCredentials = true
/* … */
```
现在，`fetch("./", { credentials:"include" }).then(/* … */)`足够了。

### 3.2.6 栗子
> 栗子1

一个 https://foo.invalid/ 的脚本想要从 https://foo.invalid/ 上获取数据。（没有认证也没有访问响应头部很重要）。
```
var url = "https://bar.invalid/api?key=730d67a37d7f3d802e96396d00280768773813fbe726d116944d814422fc1a45&data=about:unicorn";
fetch(url).then(success, failure)
```
这将会使用 CORS 协议，尽管这对于 foo.invalid 的开发者来说是完全透明的。作为 CORS 协议的一部分，用户代理将会在请求包含`Origin`协议头部：
```
Origin: https://foo.invalid
```
一旦从 bar.invalid 接收到响应，用户代理将会验证`Access-Control-Allow-Origin`响应头部。如果他的值是`https://foo.invalid`或者`*`，用户代理将会调用 success 回调，如果是其他值或者没有值，用户代理将会调用 failure 回调

> 栗子2
foo.invalid 的开发者又回来了，现在想要从 bar.invalid 获取一些数据，同时访问响应头部。
```
fetch(url).then(response => {
  var hsts = response.headers.get("strict-transport-security"),
      csp = response.headers.get("content-security-policy")
  log(hsts, csp)
})
```
bar.invalid 为前面的每一个栗子正确提供了`Access-Control-Allow=Origin`响应头部。hsts 和 csp 的值取决于`Access-Control-Expose-Headers`响应头部。比如，如果响应包含下面的头部：
```
Content-Security-Policy: default-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubdomains; preload
Access-Control-Expose-Headers: Content-Security-Policy
```
那么  hsts 将会是 null，并且 csp 将会是 “default-src 'self'”，尽管响应的确包含了两个头部。这是因为 bar.invalid 必须明确的在`Access-Control-Expose-Headers`响应头部中列出他们想要分享的头部名。

可选的，如果 bar.invalid 想要给请求分享所有除了认证之外的所有的头部，可以使用`*`作为`Access-Control-Expose-Headers`的值。乳沟请求包含了认证，则响应头部名必须明确的列出，而不能使用`*`。

> 栗子3
foo.invalid 的开发者回来了。现在要从 bar.invalid 获取一些数据，并且包含认证。这一次 CORS 协议对于开发者不再是透明的了，因为认证需要一个明确的选项。
```
fetch(url, { credentials:"include" }).then(success, failure)
```
这同时让 bar.invalid 任何`Set-Cookie`响应头部包含所有的功能（否则他们将会被忽略）。

用户代理将会确保在请求中包含所有相关的认证。他也会在响应上做严格要求。不仅仅是 bar.invalid 需要将`https://foo.invalid`作为`Access-Control-Allow-Origin`头部(`*`在有认证的情况下不允许)， `Access-Control-Allow-Credentials`头部必须如下：
```
Access-Control-Allow-Origin: https://foo.invalid
Access-Control-Allow-Credentials: true
```
如果响应没有包含这两个头部，和这两个值， failure 响应将会调用。然而，任何`Set-Cookie`响应头部将会被尊重。

### 3.2.7 CORS 异常

### 3.3 `Content-Type`头部
`Content-Type`头部大部分定义在 HTTP。它的处理模型定义在这里是因为定义在 HTTP 的 ABNF 和网页内容不兼容。

从一个头部列表 headers 解析一个 MIME 类型，执行以下步骤：
1. 让 charset 为 null
2. 让 essence 为 null
3. 让 mimeType 为 null
4. 让 values 为从 headers 获取，解码，和分割`Content-Type`的结果
5. 如果 values 是 null， 则返回 failure
6. 使用 value 遍历 values：
    1. 让 temporaryMimeType 为转化 value 的结果
    2. 如果 temporaryMimeType 是失败或者 essence 是“*/*”，则 continue。
    3. 设置 mimeType 为 temporaryMimeType
    4. 如果 mimeType 的 essence 不是 essence，则
        1. 设置 charset 为 null
        2. 如果 mimeType 的参数['charset']存在，则设置 charset 为 mimeType 的参数['charset']。
        3. 设置 essence 为 mimeType 的 essence。
    5. 否则，如果 mime 的参数['charset']存在，并且 charset 不是 null， 则设置 charset 为 mimeType 的参数['charset']。
7. 如果 mimeType 是 null，则返回 失败
8. 返回 mimeType

> 警告：当解析 MIME 类型返回错误或者 essence 不是给定格式，对待它要像一个毁灭性错误。已存在的 web 平台特性没有遵循这种模式，这也是这些年一个易受攻击的安全来源。虽然有差别，但是 MIME 类型的参数可以被简单的安全忽略

栗子

| 头部（就像在网络中） | 输出（序列化后） |
| - | - |
| Content-Type: text/plain;charset=gbk,text/html | text/html |
| Content-Type: text/html;charset=gbk;a=b,text/html;x=y | text/html;x=y;charset=gbk |
| Content-Type: text/html;charset=gbk;a=b Content-Type: text/html;x=y |  |
| Content-Type: text/html;charset=gbk Content-Type: x/x Content-Type: text/html;x=y | text/html;x=y |
| Content-Type: text/html Content-Type: cannot-parse
 | text/html |
| Content-Type: text/html Content-Type: */* |  |
| Content-Type: text/html Content-Type: |  |

### 3.4 `X-Content-Type-Options`头部
`X-Content-Type-Options`响应头部可以用来要求检测响应的`Content-Type`头部和请求的目标头部。

为了确定 nosniff，给定头部列表 list，执行下面的步骤：
1. 让 values 为从 list 中获取，解码，分割`X-Content-Type-Options`的结果。
2. 如果 values 是 null，则返回 false。
3. 如果 values[0]是null，则返回 false。
4. 返回 false。

网页开发者和兼容检查者必须对`X-Content-Type-Options`使用下面的值的 ABNF :
```
X-Content-Type-Options           = "nosniff" ; case-insensitive
```
### 3.4.1 请求的响应是否应该被 nosniff 阻塞？

执行以下步骤：
1. 如果响应的头部列表的确定 nosniff 是 false，则返回 allowd。
2. 让 mimeType 为从响应的头部列表解析 MIME 类型的结果。
3. 让 destination 为请求的 destination。
4. 如果 destination 在脚本上类似，并且 mimeType 是失败的或者不是一个 JavaScript MIME 类型，则返回 blocked。
5. 如果 destination 是 style，并且 mimeType 是失败并且它的 essence 不是“text/css”，则返回 blocked。
6. 返回 allowd。

> 注意：只有请求的 destination 是脚本类似，或者“style”被考虑为？？？？？？

### 3.5 CORB
跨域读阻塞，CORB 更广为人知。是一个算法，辨识存疑的跨域资源获取（比如，获取将会失败比如从一个 img 元素渲染 JSON）并且阻塞他们在他们到达网页之前。CORB 让产生泄露敏感数据的风险远离跨域网页。

一个 CORB 保护的 MIME 类型是一个 HTML MIME 类型，一个 JSON MIME 类型，或者一个 XML MIME 类型，除了 image/svg+xml。

注意：就算没有 CORB，CORB保护的 MIME 类型访问跨域资源内容也需要被 CORS 协议管理（比如，XMLHttpRequest），不可观察的（比如，ping 和忽略响应的 CSP），或者导致失败（比如，当解码一个 HTML文档嵌入一个 img 元素作为一个图片的时候失败）。这意味着 CORB 可以阻塞 CORB 保护的 MIME 类型资源，避免引起网页的混乱。

为了执行一个 CORB 检测，给定你一个一个 requst 和 response，执行下面的步骤：
1. 如果 request 的初始化是“download”，则返回 allowed。
> 如果将下载作为导航，这个步骤将会被移除。
2. 如果 request 的当前 URL 的方案不是一个 HTTP(S) 方案，则返回 allowed。
3. 让 mimeType 为从 response 的头部列表解析 MIME 类型的结果。
4. 如果 mimeType 失败，则返回 allowed。
5. 如果响应的状态是 206，并且 mimeType 是一个 CORB 保护的 MIME 类型，则返回 blocked。
6. 如果确定 response 的头部列表的 nosniff 是 true 并且 mimeType 是一个 CORB 保护的 MIME type 或者 它的 essence 是“text/plain”， 则返回 blocked。
> 注意：CORB 只能保护 text/plain 响应有`X-Content-Type-options: nosniff`头部。不幸的是，保护这样的响应没有这个头部，当他们的状态是 206 的时候，将会大波大量已经存在的有 text/plain 的 MIME 类型的视频响应。
7. 返回 allowed。

### 3.6 `Cross-Origin-Resource-Policy`头部
`Cross-Origin-Resource-Policy`响应头部可以用来要求检测一个请求当前 URL 的域和请求的域，当请求的模式是“no-cors”。
它的值的 ABNF：
```
Cross-Origin-Resource-Policy     = %s"same-origin" / %s"same-site" ; case-sensitive
```
执行跨域资源策略检测，给定 request 和 response，执行下面的步骤：
1. 如果 请求的模型不是“no-cors”，则返回 allowed。
2. 如果请求的域和请求当前 URL 的域是同源的。则返回 allowed。
> 注意：当重定向携带一个`Cross-Origin-Resource-Policy`头部被检测，重兴乡没有这个头部将会导致响应没有被这个算法贡献。比如，request 的 tranted origin flag 是 not checked。
3. 让 policy 为从响应头部列表获取`Cross-Origin-Resource-Policy`的结果。
> 注意：这意味着`Cross-Origin-Resource-Policy: same-site, same-origin`以 allowed 结尾，因为它不会命中任何东西。两个或者更多`Cross-Origin-Resource-Policy`头部有相同的作用。
4. 如果 policy 是`same-policy`，则返回 blocked。
5. 如果下面返回 true
    - request 的源的主机和 request 当前的 URL 的主机相同
    - request 的源的方案是“https”或者响应的 HTTPS 状态是“none”
    则返回 allowed。
6. 如果 policy 是`same-site`，则返回 blocked
7. 返回 allowed

### 4. 获取
下面定义了 fetching 的算法。总而言之，它接收一个 request，输出一个 response。

也就是说，它要么设置 synchronous 标志，返回一个 response，要们入栈一个任务，声明为 process response，process response end-of-body，和 process response done

使用 request 执行一个 fetch，执行下面的步骤。一个进行中的 fetch 可以被终止通过 aborted 标志，除非指定，否则它是未设置。

用户代理可能会被要求将正在进行的 fetch 悬挂。用户代理可以接收或者忽略挂起的请求。被挂起的请求可以被恢复。用户代理应该忽略挂起的请求，如果进行的获取在 HTTP 缓存中更新了响应。

> 注意：用户代理不更新一个 request 的 HTTP 缓存的实体，如果 request 的缓存模式是“no-store”或者一个`Cache-Control: no-store`头部出现在响应中。

1. 执行这些步骤，但是在进行中的获取被终止的时候放弃：
    1. 如果 request 的 window 是“client”，这只 request 的window 为 request 的 client，如果 request 的 client 的全局对象是 Window 对象，否则，设置为“no-window”
    2. 如果 request origin 是“client”，设置 request 的 origin 为 request client 的 origin
    3. 如果 request 的头部列表不包含  Accept，则
        1. 让 value 为 `*/*`
        2. 如果 request 是 navigation request，用户代理应该设置 value 为`text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`。
        3. 否则，用户代理需要设置 value 为第一个命中的状态，根据 request 的 destination 分别执行：
        - “image”：`image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5`
        - “style”：`text/css,*/*;q=0.1`
        4. 拼接`Accept`/value 到 request 的头部列表
    4. 如果 request 的头部列表不包含`Accept-Language`，用户代理应该拼接`Accept-Language`/一个合适的值 到 reqeust 的头部列表
    5. 如果 request 的优先级是 null，则用代理的初 initiator 和 destination 合适的设置 request 的优先级到用户代理定义的对象
    6. 如果 request 是子资源请求，则
        1. 让 record 为新的 fetch record，它由 request 和 fetch算法的实例组成
        2. 拼接 record 到 request 的 client 的 fetch group 列表
2. 如果被放弃：则
    1. 让 aborted 为终止放弃标志
    2. 如果 aborted 设置，则返回 放弃网络错误
    3. 返回一个网络错误
3. 返回使用 request 执行 main fetch 的结果

### 4.1 主获取
使用 request 执行一个 main fetch，可选的设置 CORS 标志，recursive 标志，执行下面步骤：

1. 让 response 为 null
2. 执行这些步骤，但是在进行中的 fetch 被终止的时候放弃：
3. 如果放弃了，则：
4. 如果 request 的 synchronous 标志是 unset 并且 recursive 标志是 unset，同步执行剩下的步骤
5. 如果 response 是 null，则设置 response 为执行第一个命中语句不步骤的结果：
6. 如果 recursive 标志设置，则返回 response
7. 如果 response 不是一个网络错误，并且 response 不是 filted response，则
8. 让 internalResponse 为 response，如果 response 是一个网络错误，否则，为 response 的 internal response
9. 如果 internalResponse 的 URL 列表为空，则设置它为 request URL 列表的克隆
10. 设置 internalResponse 的CSP 列表
11. 如果 response 不是一个网络错误，并且下面的算法返回 blocked，则设置 response 和 internalResponse 为一个网络错误
12. 如果 response 的 type 是“opaque”，internalResponse 的 status 是 206，internalResponse 的 range-requested 标志被设置，request 的头部列表不包含`Range`，则设置 response 和 internalResponse 为网络错误
13. 如果 response 不是一个网络错误，request 的方法是`HEAD`或者`CONNECT`，或 internalResponse 的status 是一个空正文状态，设置 internalResponse 的 body 为 null，不理会任何它前面的任何入队（如果存在）。
14. 如果 response 不是一个网络错误，request 的完整性元数据不是空字符串，则：
15. 如果 request 的 synchronous 标志被设置，等待 internalResponse 的正文，然后返回 response
16. 如果 request 当前 URL 的方案是一个 HTTPS 方案，则：
17. 在 request 上入队一个 fetch 任务，为 response 处理 response
18. 等待 internalResponse 的 body
19. 在 request 入队一个 fetch 任务，为 response 去处理响应正文结束
20. 等待 internalResponse 的 trailer，或者正在进行的 fetch 终结
21. 如果正在进行的 fetch 终结，则设置 internalResponse trailer 失败标志
22. 设置 request 的 done 标志
23. 在 request 上入队一个 fetch 任务，为 response 处理响应结束。

### 4.2 schema fetch
使用 request 执行一个 shcema fetch，根据 request 当前 URL 的方案，并执行关联的步骤：
- “about”
    如果 request 的当前 URL 的 cannot-be-a-URL 标志被设置，path 包含单独的字符串“blank”，则返回一个新的 response，它的 status message 是`OK`，头部列表由 name 为`Content-Type`，value 为`text/html;charset=utf-8`的单一头部组成，body 是空的字节序列，HTTPS state 是 request 的 client 的 HTTPS 状态，如果 request 的 client 是 null。
    否则，返回一个网络错误
- “blob”
    1. 执行这些步骤，但是当进行中的 fetch 放弃的时候终止：
        1. 让 blob 为 request 当前 URL 的 blob URL 实体 的对象
        2. 如果 request 的方法是`GET`，或者 blob 不是一个 Blob 对象，则返回一个网络错误
        3. 让 response 为新的 response，它的 status message 是`OK`
        4. 拼接`Content-Type`/blob 的 size 属性到 response 的头部列表
        5. 拼接`Content-Type`/blob 的 type 属性到 response 的头部列表
        6. 设置 response 的 HTTPS 状态到 request 的 client 的 HTTPS 状态，如果 request 的 client 不是 null
        7. 设置 resposne 的 body 为在 blob 上执行读操作的结果
        8. 返回 response。
    2. 如果放弃了，则：
        1. 让 aborted 为终止的放弃标志
        2. 如果 aborted 被设置，则返回一个放弃网络错误
        3. 返回网络错误

- “data”
    1. 让 dataURLStruct 为在 request 的当前 url 执行 data: URL processor 的结果
    2. 如果 dataURLStruct 失败，则返回一个网路错误
    3. 返回一个响应，它的 status message 是`OK`，头部列表由一个名字为`Content-Type`，值为 dataURLStruct 的 MIME 类型的头部组成，序列化，body 是 dataURLStruct 的 body， HTTP state 是 request 的 client 的 HTTPS state，如果 request 的 client 是 非 null。
- “file”
    很遗憾，对于现在来说，file URL 对读者来说更像一个练习
    如果有疑惑，返回一个网络错误
- “ftp”
    很遗憾，对于现在来说，ftp URL 对读者来说更像一个练习
    1. 让 body 为用户代理从 request 当前 URL 通过 FTP 从网络获取内容的结果
    2. 让 mime 为`application/octet-stream`
    3. 如果 body 是用户生成直接列表页面通过 FTP 的 LIST 命令，则设置 mime 为`text/ftp-dir`
    4. 返回一个 status message 为`OK`的响应，头部列表由一个名为`Content-Type`的头部，并且它的值为 mime，body 是 body，HTTPS state 是“none”
    如果有怀疑，返回一个网络错误
- “HTTP(S) schema”
    返回使用 request 执行 HTTP fetch 的结果
- 其他：返回一个网络错误
### 4.3 HTTP fetch
使用 request 执行一个 HTTP fetch，可选择设置 CORS 标着和 CORS 预检标志，执行下面步骤：
1. 让 response 为 null
2. 让 actualResponse 为 null，则
3. 如果 request 的 service-worker 的 mode 是“all”，则
4. 如果 response 是 null，则：
5. 如果 actualResponse 状态是重定向状态，则
6. 返回 response。
### 4.4 HTTP 重定向获取
1. 让 actualResponse 为 response，如果 actualResponse 不是一个过滤的 response，否者设置为 response 的内部响应
2. 如果 actualResponse 的定位 URL 是 null，则返回 resposne。
3. 如果 actualResponse 的定位 URL 失败，则返回一个网错误
4. 如果 actualResponse 的定位 URL 的方案不是 HTTP(S)方案，则返回一个网络错误
5. 如果 request 的重定向统计是20，返回一个网络错误
6. 自增 request 的重定向统计
7. 如果请求的 mode 是 cors，actualResponse 的本地 URL 包含认证，request 的 tainted origin 标志设置，或者 request 的 origin 和 actualResponse 定位 URL 的 origin 不是同源，则返回一个网络错误
8. 如果 CORS 标志设置，并且 actualResponse 的定位 URL 包含认证，则返回一个网络错误
9. 如果 actualResponse 的定位 URL 的 origin 和 request 当前 URL 的 orign
10. 如果 actualResponse 的 Loacation URL 的 origin 和 request 当前 URL 的 origin 不是同源，并且 request 的 origin 和 request 的当前 URL 的 origin 不是同源，则设置 request 的 tainted origin 标志
11. 如果下面其中一个是 true
    - actualResponse 的 status 是 301 或者 302 并且 request 的 method 是`POST`
    - actualResponse 的 status 是 303 并且 request 的 method 是`HEAD`
12. 如果 request 的 body 不是 null，则设置 request 的 body 为安全解析 request 的 body 的 source 的第一个返回值
13. 拼接 actualResponse 的 location URL 到 request 的 URL 列表
14. 在 request 和 actualResponse 上调用在重定向上设置 request 的引用策略
15. 返回使用 request 执行 main fetch的结果，
    - CORS 标志如果设置并且
    - recursive 标志设置，如果 request 的重定向模式是“manual”
        > 注意：当从 HTML 的导航算法调用的时候，这只能是”manual“
    > 注意：这需要去调用 main fetch 去正确获取响应的 tainting。
### 4.5 HTTP 网络或缓存获取
1. 让 httpRequest 为 null
2. 让 response 为 null
3. 让 storedResponse 为 null
4. 让 revalidatingFlag 为 unset
5. 执行下面步骤，但是在进行中的获取终止的时候放弃

6. 如果终止，则：
7. 如果 response 是 null，则
8. 如果 httpRequest 的 头部列表包含`Range`，则设置 response 的 range-requested 标志
9. 如果 CORS 标志是 unset，并且 request 和 response 的跨域资源策略检测返回 blocked，则返回一个网络错误
10. 如果 response 的 status 是 401，CORS 标志是 unset，credentials 标志设置，request 的 window 是环境设置对象，则
11. 如果 response 的状态是 407，则
12. 如果 authentication-fetch 标志设置，则为 request 和给定 realm 创建一个 authentication 入口
13. 返回响应


### 4.6 HTTP 网络获取
1. 如果 credentials 标志被设置，则设置 credentials 为 true，否则为 false
2. 让 response 为 null
3. 根据 request 的 mode 执行：
    - “websocket”：给定 request 当前的 URL，让 connection 为获取一个 WebSocket 链接的结果
    - 否则，给定 request 当前 URL 的 origin 和 credentials，让 connection 为获取一个链接的结果
4. 执行这些步骤，但是在进行中的获取终止的时候放弃：
    1. 如果 connection 是失败，则返回一个网络错误
    2. 如果 connection 不是一个 HTTP/2 链接，request 的 body 不是 null，并且 request 的 body 的 source 是 null，则拼接`Transfer=Encoding`/`chunked`到 request 的请求头部
    3. 设置 response 为使用 reqeust 生成一个基于 connection 的 HTTP request，并使用下列警告
        - 遵循 HTTP 的相关需求
        - 等待所有的头部被传输
        - 任何响应的状态在 100-199 闭区间内，但不是 101，都忽略
    如果 request 的头部列表包含`Transfer-Encoding`/`chunked`，并且 response 是通过 HTTP/1.0 或者更旧的协议传输，则返回一个网络错误
    如果 HTTP request 生成一个 TLS 客户端证书弹窗，则：
        1. 如果 request 的 window 是一个环境设置对象，在 request 的 window 生成一个弹窗
        2. 否则，返回一个网络错误
    如果 response 是从 HTTPS 取回，设置 HTTPS 状态为“deprecated”或者“modern”
    为 request 传输 body
5. 如果放弃，则：
    1. 让 aborted 为终止的放弃标志
    2. 如果 connection 使用 HTTP/2，则传输一个 RST_STREAM 帧
    3. 如果 aborted 被设置，则返回一个放弃网络错误
    4. 返回网络错误
6. 让 highWaterMark 用户选择的，为非负的，非 NAN 的数字，
7. 让 sizeAlgorithm 是一个散发，接收一个 chunk 对象，并返回一个用户代理选择的非负的，非 NAN 的，非无穷数。
8. 让 pull 为一个 action，如果它被挂起，则恢复进行中的获取
9. 让 cancel 为一个 acton，它终止进行中的获取，通过设置 aborted 标志。
10. 让 stream 为使用 highWaterMark，sizeAlgorithm，pull，cancel 构造一个 ReadableStrem 对象的结果
11. 运行这些步骤，但是在进行中的获取终止的时候放弃：
    1. 设置 response 的 body 为一个新的 body，它的 stream 是 stream
    2. 如果 response 有一个负荷正文长度，则设置 response body 的总字节为负荷正文长度
    3. 如果 response 不是一个网络错误，并且 reqeust 的 cache mode 不是“no-store”，为 request 更新 HTTP 缓存中的 response
    4. 如果 credentials 标志被设置，并且用户代理没有设置去堵塞 request 的 cookie，则在 request 的当前 URL 和 response 头部列表每一个名字字节大小写不敏感命中`Set-Cookie`的 header 的 value 上执行“set-cookie-string”转化算法，如果存在。
12. 如果放弃了，则：
    1. 让 aborted 为终止的放弃标志
    2. 如果 aborted 被设置，则设置 response 的放弃标志
    3. 返回 response
13. 同步执行这些步骤
    1. 执行这些步骤，但是在进行中的获取终止的时候放弃：
        1. 如果为真：
            1. 让 bytes 为传输的字节
            2. 使用 bytes 的长度增加 response 的 body 的传输字节
            3. 让 codings 为给定`Content-Encoding`和 response 头部列表解析头部列表值的结果
            4. 设置 bytes 为给定 codings 和 bytes 处理内容编码的结果
            5. 如果 bytes 失败，则终止进行中的获取
            6. 入队一个 Uint8Array 对象包裹一个 ArrayBuffer 包含 bytes 到 stream。如果抛出一个异常，终止进行中的获取，使用这个异常 error stream。
            7. 如果 stream 不需要更多的数据，并且 request 的同步标志是 unset，请求用户代理挂起进行中的获取
        2. 否则，如果 response 比特传输的信息正文正常完成并且 stream 是可读的，则关闭 stream 并放弃这些同步步骤。
    2. 如果放弃：
        1. 让 aborted 为终止的放弃标志：
        2. 如果 aborted 被设置，则
            1. 设置 response 的 aborted 标志
            2. 如果 stream 是可读的，使用“AbortError”DomException来 error stream
        3. 否则，如果 stream 是可读的，使用一个 TypeError 来 error stream
        4. 如果 connection 使用 HTTP/2，则传输一个 RST——STREAM 帧
        5. 否则，用户代理应该在它影响到性能的时候关闭连接
14. 返回 response

### 4.7 CORS 预获取
使用 request 执行一个 CORS 预检请求，执行下面步骤：
1. 让 preflight 为新的方法是`OPTIONS`的 request，URL 是 request 的当前 URL，initiator 是 request 的 initiator，destination 是 request 的 destination，origin 是 request 的 origin，referrer 是 request 的 referrer，referrer policy 是 request 的 referrer policy，mode 是“cors”，tainted origin 标志是 request 的 tainted origin 标志。
2. 在 preflight 的头部列表中拼接`Access-Control-Request-Method`到 reqeust 的 method
3. 让 headers 为 request 的头部列表的 CORS 不安全请求头部名字
4. 如果 headers 不是空，则：
    1. 让 value 为 headers 中每一项用`,`分割
    2. 在 preflight 的头部列表中拼接`Access-Control-Request-Headers`到 value
5. 让 response 为使用 preflight 和 CORS 标志执行一个 HTTP 网路或者缓存获取的结果
6. 如果 request 的 CORS 检查返回成功并且 response 的 status 是一个 ok 状态，则：
    1. 给定`Access-Control-Allow-Methods`和 response 的头部列表，让 methods 为解析头部列表值的结果
    2. 给定`Access-Control-Allow-Headers`和 response 的头部列表，让 headerNames 为解析头部列表值的结果
    3. 如果 methods 或者 headerNames 是失败，则返回一个网络错误
    4. 如果 methods 是 null，并且 request 的 use-CORS-preflight 标志设置，则设置 methods 为一个新的列表，并包含 request 的方法
    5. 如果 request 的 method 不再 methods 中，request 的 method 不是一个 CORS 安全方法，并且 request 的 credentials mode 是“include”或者 methods 不包含`*`，则返回一个网络错误
    6. 如果 request 的头部列表的名字中有一个是 CORS 非通配符头部名字，并且不是字节大小写不敏感的匹配 headerNames 中的每一项，则返回一个网络错误
    7. 使用 unsafeName 遍历 CORS 不安全请求头部名字和 request 的头部列表，如果 unsafeName 不是字节大小写不敏感命中 headerNames 中的一项并且 request 的 credentials mode 是“include”或者 headerNames 不包含`*`，则返回一个网络错误
    8. 给定`Access-Control-Max-Age`和 response 的头部列表，让 max-age 为解析头部列表值的结果
    9. 如果 max-age 是失败或者 null，则设置 max-age 为 0
    10. 如果 max-age 大于最大限制，则设置 max-age 为最大限制
    11. 如果用户代理不提供缓存，则返回 response
    12. 使用 method 遍历 methods，使用 request，如果某一个是一个缓存入口命中方法，设置命中入口的 max-age 为 max-age
    13. 使用 method 遍历 methods，使用 request，如果某一个不是一个缓存入口命中方法，使用 request，max-age，method，和 null 创建一个新的缓存入口
    14. 使用 headerName 遍历 headerNames，如果某一个是头部名称缓存入口命中，则设置命中入口的 max-age 为 max-age
    15. 使用 headerName 遍历 headerNames，如果某一个不是头部名称缓存入口命中，使用 request，max-age，null，和 headerName 创建一个新的缓存入口
    16. 返回 response
7. 否则，返回一个网路错误

### 4.8 CORS 预缓存
用户代理有一个关联的 CORS 预请求缓存，一个 CORS 预请求缓存是一个缓存条目的列表

一个缓存条目由下面组成：
- 序列化的 origin（一个字节序列）
- URL（一个 URL）
- max-age（一个秒数）
- credentials（一个 boolean）
- method（null，`*`，或者一个 method）
- 头部名（null，`*`，或者一个头部名）

缓存条目必须在从存储草他们 max-age 域指定的秒数过去之后移除。缓存条目可能在那一刻到来之前移除。

创建一个缓存实体，给定 request，method，和 headerName，执行下面步骤
1. 让 entry 为一个缓存实体，如下初始化
    - serialized origin：使用 request 序列化 request origin 的结果
    - URL：request 的当前 URL
    - max-age：max-age
    - credentials：如果 request 的 credentials mode 是“include”，则是 true，否则就是 false
    - method：method
    - header name：headerName
2. 拼接 entry 到用户代理的 CORS 预请求缓存


清理一个缓存条目，给定一个 request，移除所有用户代理的 CORS 预请求缓存中序列化 origin 为 request 序列化 request orign 的结果，URL 为 request 当前 URL 的条目。

request 命中一个缓存实体 entry，entry 的序列化 origin 是 request 序列化 request origin 的结果，entry 的 URL 是 request 当前 URL，并且下面一个条件为真：
- entry 的 credentials 是 true
- entry 的 credentials 是 false 并且 request 的 credentials mode 不是“include”

request 命中一个方法缓存实体 method，当用户代理的 CORS 预请求缓存中的某一项是缓存命中并且 method 是 method 或者`*`
request 命中一个头部名缓存实体 method，当用户代理的 CORS 预请求缓存中的某一项是缓存命中并且下面一个条件为真：
- 它的头部名称是字节大小写不敏感命中 headerName
- 它的头部名称是`*`并且 headerName 不是一个 CORS 通配符请求头部名字。

### 4.9 CORS 检查
为一个 request 和 response 执行一个 CORS 检查，执行下面步骤：
1. 让 origin 为从 response 头部列表获取`Access-Control-Allow-Origin`的结果
2. 如果 origin 是 null，则返回 failure
> 注意：Null 不是`null`
3. 如果 request 的认证模式不是"include"，并且 origin 是`*`，则返回成功。
4. 如果序列化 request 的源的结果不是 orign，则返回失败
5. 如果 request 的认证模式不是"include"，则返回成功
6. 让 credentials 为从 response 的头部列表获取  `Access-Control-Allow-Credentials` 的结果
7. 如果 credentials 是`true`，则返回成功 
8. 返回失败

### 5. 获取 API
fetch() 方法是获取资源更接近底层的 API 。它包含比 XMLHttpRequest 稍微更底层，尽管它当前缺少请求回来时候的进度（没有响应进度）。

fetch() 方法让获取资源更加直接，并解析内容为 Blob：
```
fetch("/music/pk/altes-kamuffel.flac")
  .then(res => res.blob()).then(playBlob)
```
如果你只想要记录一个特定的响应头部：
```
fetch("/", {method:"HEAD"})
  .then(res => log(res.headers.get("strict-transport-security")))
```
如果你想要检查一个跨域资源一个特定的响应头部并处理响应：
```
fetch("https://pk.example/berlin-calling.json", {mode:"cors"})
  .then(res => {
    if(res.headers.get("content-type") &&
       res.headers.get("content-type").toLowerCase().indexOf("application/json") >= 0) {
      return res.json()
    } else {
      throw new TypeError()
    }
  }).then(processJSON)
```
如果想要和 URL 查询参数一起使用：
```
var url = new URL("https://geo.example.org/api"),
    params = {lat:35.696233, long:139.570431}
Object.keys(params).forEach(key => url.searchParams.append(key, params[key]))
fetch(url).then(/* … */)
```
如果你想要接收正文数据进度：
```
function consume(reader) {
  var total = 0
  return pump()
  function pump() {
    return reader.read().then(({done, value}) => {
      if (done) {
        return
      }
      total += value.byteLength
      log(`received ${value.byteLength} bytes (${total} bytes in total)`)
      return pump()
    })
  }
}

fetch("/music/pk/altes-kamuffel.flac")
  .then(res => consume(res.body.getReader()))
  .then(() => log("consumed the entire body without keeping the whole thing in memory!"))
  .catch(e => log("something went wrong: " + e))
```

### 5.1 头部类
```
typedef (sequence<sequence<ByteString>> or record<ByteString,ByteString>) HeadersInit;

[Constructor(optional HeadersInit init), Exposed=(Window,Worker)]
interface Headers {
    void append(ByteString name, ByteString value);
    void delete(ByteString name);
    ByteString? get(ByteString name);
    boolean has(ByteString name);
    void has(ByteString name);
    void set(ByteString name, ByteString value);
    iterable<ByteString, ByteString>;
}
```
> 注意：不像一个头部列表，一个 Headers 对象不能有多余一个`Set-Cookie`头部。在某个方面来说，这是一个问题，因为不像其他头部，`Set-Cookie`头部不能被绑定，但是因为`Set-Cookie`头部并没有被暴露给客户端的 JavaScript，这稍微是一个可接受的拖鞋。实现可以选择更加有效的 Headers 底箱表示，甚至是一个头部列表，同时也支持一个关联的数据`Set-Cookie`对象。

一个 Headers 对象可以使用多种 JavaScript 数据结构初始化。
```
var meta = { "Content-Type": "text/xml", "Breaking-Bad": "<3" }
new Headers(meta)

// The above is equivalent to
var meta = [
  [ "Content-Type", "text/xml" ],
  [ "Breaking-Bad", "<3" ]
]
new Headers(meta)
```
一个 Headers 对象有一个关联的头部列表（一个头部列表），它初始化为空。
> 注意：这可以是一个指向头部列表的指针，比如 Request 对象的实例 request

一个 Headers 对象有一个关联的 guard，可能是“immutable”，“request”，“request-no-cors”，“response”或者“none”

拼接键/值对 name/value 到 Headers 对象（headers），执行下面的步骤：
1. 规范化 value。
2. 如果 name 不是一个 name 或者 value 不是一个 value，则抛出一个 TypeError。
3. 如果 headers 的 guard 是“immutable”，则爬出一个 TypeError。
4. 否则，如果头部的 guard 是“request”并且 name 是一个禁止的头部名字，则返回。
5. 否则，如果 headers guard 是“request-no-cors”。
    1. 让 temporaryValue 为从 headers 的头部列表获取名字的结果。
    2. 如果 temporaryValue 是 null，则设置 temporaryValue 为 value。
    3. 否则，设置 temporaryValue 为 temporaryValue，后跟 0x2C 0x20，后跟 value。
    4. 如果 name/temporaryValue 不是一个 非 CORS 安全请求头部，则返回。
6. 否则，如果 headers 的 guard 是“response”，并且 name 是一个禁止的响应头部名，返回。
7. 拼接 name/value 到 headers 的头部列表。
8. 如果 headers 的 guard 是“request-no-cors”，则从 headers 移除私有非 CORS 请求头部。

使用给定对象（object）填充 Headers 对象（headers），执行下面步骤：
1. 如果 object 是一个序列，则使用 header 遍历 object：
    1. 如果 header 不包含明确的两个项，则抛出 TypeError
    2. 拼接 header  第一个项/header 第二个项到 headers
2. 否则，对象是一个 record，则遍历 object 的 key->value，拼接 key/value 到 headers。

从 Headers 对象（headers）移除私有的非 CORS 请求头部，则执行以下步骤：
1. 使用 headerName 遍历私有的非 CORS 请求头部名字：
    1. 从 headers 的头部列表删除 headername。
> 注意：这将会在 headers 被非私有代码修改的时候

Header(init) 构造方法调用的时候，不惜执行下面的步骤：
1. 让 headers 为一个新的 Headers 对象，它的 guard 为“none”
2. 如果 init 给定，则使用 init 填充 headers
3. 返回 headers

append(name, value)j 方法，调用的时候，必须拼接 name/value 到上下文对象

delete(name) 方法调用的时候，必须执行下面的步骤：
1. 如果 name 不是一个 name，则抛出一个 TypeError
2. 如果上下文对象的 guard 是“immutable”，则抛出一个 TypeError。
3. 否则，如果上下文对象的 guard 是“request”并且 name 是一个禁止的头部名，返回。
4. 否则，如果上下文对象的 guard 是“request-no-cors”，name 不是一个非 CORS 安全请求头部名，并且 name 不是私有非 CORS 请求头部名，返回。
5. 否则，如果上下文对象的 guard 是“response”，并且 name 是禁止响应头部名，则返回。
6. 如果上下文对象的头部列表不包含 name，则返回。
7. 从上下文对象的头部列表中删除 name。
8. 如果上下文对象的 guard 是“request-no-cors”，则从上下文对象移除私有非 CORS 请求头。

get(name) 方法调用的时候，必须执行下面的步骤：
1. 如果 name 不是一个 name， 则抛出一个 TypeError。
2. 返回从上下文对象头部列表获取 name 的结果。

has(name) 方法调用的时候，必须执行下面的步骤：
1. 如果 name 不是一个 name，则抛出一个 TypeError。
2. 如果上下文独享的头部列表包含 name，则返回 true，否则返回 false。

set(name, value) 方法调用的时候，必须执行下面的步骤：
1. 规范化 value
2. 如果 name 不是一个 name 或者 value 不是一个 value，则抛出一个 TypeError
3. 如果上下文对象的 guard 是“immutable”，则抛出一个 TypeError。
4. 否则，如果上下文对象的 guard 是”request“，并且 name 是禁止的头部名，则返回。
5. 否则，如果上下文兑现的 guard 是”request-no-cors“，并且 name/value 不是一个非 CORS 安全请求头部，则返回。
6. 否则，如果上线问对象的 guard 是“response”，并且 name 是一个禁止的响应头部名，则返回。
7. 在上线问对象的头部列表中设置 name/value。
8. 如果上下文对象的 guard 是“request-no-cors”，则从上下文对象移除稀有的非 CORS 请求头部。

迭代的值是执行排序和绑定上下文对象头部列表的返回值。

### 5.2 正文混合
```
typedef(Blob or BufferSource or FormData or URLSearchParams or ReadableStream or USVString) BodyInit;
```
从 object 中安全的解析正文和`Content-Type`，执行下面的步骤：
1. 如果 object 是一个 ReadableStream 对象，则：
    1. 断言：object 不是 disturbed，也不是 locked。
2. 返回解析 object 的结果

注意：安全解析操作是解析操作的子集，它不会抛出异常。

从 object 中解析正文和一个`Content-Type`值，还有一个可选的 keepalive 标志，执行下面的步骤：
1. 让 stream 为构造 ReadableStrem 对象的结果。
2. 让 Content-Type 为 null
3. 让 action 为 null
4. 让 source 为 null
5. 根据对象的类型执行：
    - Blob
        设置 action 为读取 object 的 action。
        如果 object 的 type 属性不是空的字节序列，设置 Content-Type 为它的值
        设置 source 为 object
    - BufferSource
        入队一个 UinitArray 对象，它包裹一个 ArrayBuffer，它包含一个 object 只有的字节到 stream 和关闭 stream。如果这抛出一个异常，使用这个异常让 stream 错误。
    - FormData
        设置 action 为一个 action，它执行 multipart/form-data 编码算法，将 object 作为表单数据集，将 UTF-8 作为明确的字符编码。
        设置 Content-Type 为`multipart/form-data; boundary=`，跟随着 multipart/form-data 编码算法生产的 multipart/form-data 包裹字符串。
    - URLSearchParams
        设置 action 为一个 aciton，它对 对象的列表执行 application/x-www-form-urlencoded。
        设置 Content-Type 为`application/x-www-form-urlencoded;charset=UTF-8`。
        设置 source 为对象。
    - USVString
        设置 action 为一个 action，它在 object 上执行 UTF-8 编码。
        设置 Content-Type 为`text/plain;charset=UTF-8`。
        设置 source 为 object。
    - ReadableStream
        如果 keepalive 标志被设置，则抛出一个 TypeError。
        如果 object 是 disturbed 或者 locked，则抛出一个 TypeError。
6. 如果 action 不是 null，同步执行 action：
    1. 无论合适，一个或者多个字节可用，让 byte 为 btytes，并将一个 Uint8Array 对象包裹一个 ArrayBuffer 包含 bytes 推入 stream。如果创建 ArrayBuffer 抛出一个异常，使用异常让 stream 错误并取消执行的 action。
    2. 当执行 action 完成的时候，关闭 stream。
7. 让 body 为一个正文，它的 stream 是 stream，它的 source 是 source
8. 返回 body 和 Content-Type

```
interface mixin Body {
    readonly attribute ReadableStrem? body;
    readonly attribute boolean bodyUsed;
    [NewObject] Promise<ArrayBuffer> arrayBuffer();
    [NewObject] Promise<Blob> blob();
    [NewObject] Promise<FormData> formData();
    [NewObject] Promise<any> json();
    [NewObject] Promise<USVString> text();
}
```

注意：你不想依赖网络层的格式，比如 HTML，可能不会在这里暴露。相反，HTML 转化器 APi 可能接收一个流。

实现 Body 混合的对象获得一个关联的 body（null 或者 一个 body），和一个 MIME 类型（失败或者一个 MIME 类型）。

一个实现 Body 混入的对象被成为 disturbed，如果 body 不是 null，并且它的流是 disturbed。

一个实现 Body 混入的对象被称为 locked，如果 body 不是 null，并且它的流是 locked。

body 属性获取起必须返回 null，如果 body 是 null，并且 body 的流不是

bodyUsed 属性获取起必须返回 true，如果 disturbed，否则返回 false。

实现 Body 混入的对象同时有一个关联的 package data 算法，给定 bytes，一个 type 和一个 mimeType，根据 type 执行关联的步骤：
- ArrayBuffer
    返回一个新的 ArrayBuffer，它的内容是 bytes。
    > 申请一个 ArrayBuffer 可以抛出一个 RangeError
- Blob
    返回一个 Blob 它的内容是 byte 并且它的 type 属性是 mimeType。

- FormData
    如果 mimeType essence 是“multipart/form-data”，则：
        1. 根据表单中的返回值：multipart/form-data，使用 mimeType 中的`boundary`参数的值转化 bytes。
        每一个部分的`Content-Disposition`头部包含`filename`参数必须转为到一个 实体，它的值是一个 File 对象，它包含部分的内容，File 对象的 name 属性必须拥有部分`filename`参数的值。File 对象的 type 属性必须有`Content-Type`头部的值，如果有这个头部，否则就是`text/plain`（定义在[RFC7578]章节4.4的默认值）

        每个`Content-Disposition`头部不包含`filename`参数的部分必须转化到一个实体，它的值是这部分的 UTF-8 编码内容。

        注意：`Content-Disposition`头部包含`name`参数的部分，它的值是`_charset_`就像其他部分一样转化。他不改变编码。
        2. 如果因为一些理由失败，则抛出一个 TypeError。
        3. 返回一个新的 FormData 对象，拼接每一个实体，从转化操作得出来的结果，到实体。
    否则，如果 mimeType 的 essence 是“application/x-www-form-urlencoded”，则：
        1. 让 entries 为转化 bytes 的结果
        2. 如果 entried 是失败的，则抛出一个 TypeError。
        3. 返回一个新的 FormData 对象，它的 entries 是 entried。
    否则，抛出一个 TypeError。
- JSON
    返回在 bytes 上执行从 bytes 上转化 JSON 的结果。

- text
    返回在 bytes 上执行 UTF-8 解码的结果。

实现 Body 混入的对象有一个关联的消费者 body 算法，给定 type，执行以下步骤：
1. 如果这个对象是 disturbed 或者 locked，返回一个新的 promise，并使用一个 TypeError rejected。
2. 让 stream 为 body 的 stream，如果 body 不是 null，否则为一个 空的 ReadableStrem 对象
3. 让 reader 为从 stream 中获取一个 reader 的结果，如果抛出一个异常，返回一个新的 promise，使用这个异常 rejected。
4. 让 promise 为使用 reader 从 stream 中读取所有字节的结果
5. 返回传输 promise 一个 fulfillment 处理器 返回 package data 算法的结果，并使用它的第一个参数，type 和这个 对象的 MIME 类型。

arrayBuffer() 方法调用的时候，必须返回执行使用 ArrayBuffer 消费正文的结果

blob() 方法调用的时候，必须返回使用 Blob 执行消费正文的结果

formData() 方法调用的时候，必须返回使用 FormdData 执行消费正文的结果。

json() 方法调用的时候，必须返回使用 JSON 执行消费正文的结果。

text() 方法调用的时候，必须返回使用 text 执行消费正文的结果。


### 5.3 请求类
```
typedef (Request or USVString) RequestInfo;

[Constructor(RequestInfo input, optional RequestInit init = {}), Exposed=(Window,Worker)]
inter Request {
    readonly attribute ByteString method;
    readonly attribute USVString url;
    [SameObject] readonly attribute Headers headers;

    readonly attribute RequestDestination destination;
    readonly attribute USVString regerrer;
    readonly attribute ReferrerPolicy referrerPolicy;
    readonly attribute RequestMode mode;
    readonly attribute RequestCredentials credentials;
    readonly attribute ReqeustRedirect redirect;
    readonly attribute DOMString integrity;
    readonly attribute boolean keepalive;
    readonly attribute boolean isReloadNavigation;
    readonly attribute boolean isHistoryNavigation;
    readonly attribute AbortSignal signal;

    [Newobject] Request clone();
}
Request includes Body;

dictionary RequestInit {
    ByteString method;
    HeadersInit headers;
    BodyInit? body;
    USVString referrer;
    ReferrerPolicy referrerPolicy;
    RequestMode mode;
    RequestCredentials credentials;
    RequestCache cache;
    RequestRedirect redirect;
    DOMString integrity;
    boolean keepalive;
    AbortSignal? signal;
    any window; // 只能被设置为 null
}

enum RequestDestination { "", "audio", "audioworklet", "document", "embed", "font", "image", "manifest", "object", "paintworklet", "report", "script", "sharedworker", "style", "track", "video", "worker", "xslt"};

enum RequestMode { "navigate", "same-origin", "no-cors", "cors"};

enum RequestCredentials { "omit", "same-origin", "include" };

enum RequestCache { "default", "no-store", "reload", "no-cahce", "force-cahce", "only-if-cached" };

enum RequestRedirect { "follow", "error", "manual" };
```

> 注意：“serviceworker”的 RequestDestination 是缺失的，因为它不能被 JavaScript 观察到。实现将还要支持它作为目标。“websocket” 的 RequestMode 是缺失的，因为它不能从 JavaScript 被使用或者被观察到。

一个 Request 对象有一个关联的请求（一个 request）。
一个 Request 对象有一个关联的 headers（null 或者 Headers 对象）。初始化为 null。
一个 Request 对象有一个关联的 signal（一个 AbortSignal 对象），初始化为一个新的 AbortSignal 对象。
一个 Request 对象的正文是请求的正文。

```
request = new Request(input [, init])
    返回一个新的 request 它的 url 属性是 input，如果 input 是一个字符串，是 input 的 url 属性，如果 input 是 Request 对象。

    init 属性是一个对象，它的属性可以被设置为如下：

    method: 设置 request method 的方法

    headers: 一个 Headers 对象，一个对象字面量，或者一个两项数组组成的数组，用来设置 request 的 headers
    
    body: 一个 BodyInit 对象或者 null，用来设置 request 的 body。

    referrer: 一个同源 URL 的字符串，"about:client"，或者一个空字符串，用来设置 request 的 referrer。

    refererPolicy: 一个引用策略，用来设置好 referrerPolicy

    mode: 一个字符串，指示请求将会使用 CORS，或者将会严格同源 URL，设置 request 的 mode
    
    credentials: 一个字符串，指示是否会将认证和请求一起发送，总是，永不，或者只发送到同源 URL。设置 request 的 credentials。

    cache: 一个字符串，指示请求将会如何和浏览器的缓存交互，设置 request 的 cache

    redirect: 一个字符串，指示 request 是否遵循重定向，遭遇重定向的时候导致一个错误或者返回一个重定向（在一个透明风格）。设置 request 的 redirect。

    integrity:  request 一个代表资源的哈希符号。设置 request 的 integrity

    keepalive: 一个 boolean，设置 request 的 keepalive

    signal: 一个 AbortSignal 去设置 request 的 signal

    window: 只能是 null，用来从任意 Window 解绑 request。

request . method：返回 request 的 HTTP 方法，默认是“GET”

request . url：返回 request 的 URL

request . headers：返回一个和 request 关联的头部组成的 Headers 对象。注意用户代理在网络层添加的头部将不会计入，比如“Host”头部

request . destination：返回 request 请求的资源类型

request . referrer：返回 request 的引用。它的值可以是同源 URL，如果 init 中明确的设置，空字符串表示没有引用，全局默认是“about:client”。这用在获取的时决定请求生产的`Referer`头部。

request . referrerPolicy：返回 request 关联的引用策略。这用来获取时计算 request 引用的值。

request . mode：返回 request 关联的 mode，是一个字符串，指示请求将会使用 CORS，或者将会严格的同源 URL。

request . credentials：返回 request 关联的认证模式，它是一个字符串，指示认证是否会和请求发送，总是，永不，或者当且仅当发送同源 URL。

request . cache：返回 request 关联的缓存模式，它是一个字符串，指示请求获取的时候如何和浏览器的缓存交互

request . redirect：返回 request 关联的重定向模式，它是一个字符串，指示获取的时候请求如何处理重定线。一个请求默认遵循重定向。

request . integrity：返回 request 子资源直接信息，是被请求资源的 hash 记号。他的值由多个被空白分离的哈希值。

request . keepalive：返回一个布尔值，指示请求创建之后是否可以在全局存活。

request . isReloadNavigation：返回一个布尔值，指示 request 是否用来重新加载导航。

request . isHistoryNavigation：返回一个布尔值，指示 request 是否是一个历史导航（比如，向后导航）

request . signal：返回 request 关联的信号，是一个 AbortSignal 对象，指示 request 是否被放弃，并且放弃时间处理器。

```
Request(input, init) 构造器必须执行下面步骤：
1. 让 request 为 null
2. 让 fallbackMode 为 null
3. 让 fallbackCredentials 为 null
4. 让 baseURL 为当前设置对象的 API 的基本 URL
5. 让 signal 为 null
6. 如果 input 是一个字符串，则
    1. 让 parsedURL 为 input 和 baesURL 转化的结果
    2. 如果 parsedURL 是失败的，则抛出一个 TypeError
    3. 如果 parsedURL 包含认证，则抛出一个 TypeError
    4. 设置 request 为一个新的 request，它的 url 是 parsedURL
    5. 设置 fallbackMode 为“cors”
    6. 设置 fallbackCredentials 为“same-origin”
7. 否则（input 是 Request 对象）：
    1. 设置 request 为 input 的 request
    2. 设置 signal 为 input 的 signal
8. 让 origin 为当前设置对象的 origin
9. 让 window 为 “client”。
10. 如果 request 的 window 是一个环境设置对象，并且它的 origin 是和 origin 是同源的，设置 window 到 request 的 window
11. 如果 init["window"] 存在并且非 null，则抛出一个 TypeError
12. 如果 init[“window”] 存在，则设置 window 为 “no-window”
13. 设置 request 为一个新的 request，并使用下面的属性：
    - URL： request 当前的 URL
    - method：request 的方法
    - header list：request 头部列表的拷贝
    - unsafe-request flag：设置
    - client：当前设置对象
    - window：window
    - priority：request 的优先级
    - origin：“client”
    - referer：request 的引用
    - referer policy：request 的引用策略
    - mode：request 的 mode
    - credentials mode：认证模式
    - cache mode：request 的缓存模式
    - redirect mode：request 的重定向模式
    - integrity metadata：request 的完整性信息
    - keepalive flag：request 的 keepalive 标志
    - reload-navigation flag：request 的 reload-navigation 标志
    - history-navigation flag：request 的 history-navigation 标志
14. 如果 init 不是空，则：
    1. 如果 request 的 mode 是“navigate”，则设置他到“same-origin”
    2. 不设置 request 的 reload-navigation 标志
    3. 不设置 request 的 history-navigation 标志
    4. 设置 request 的 referrer 到“client”
    5. 设置请求的 referer policy 为空的字符串
15. 如果 init["referrer"] 存在，则
    1. 让 referrer 为 init["referrer"]
    2. 如果 referrer 为空字符串，则设置 request 的 referrer 为“no-referrer”
    3. 否则
        1. 让 parsedReferrer 为 referer 和 base URL 转化的结果
        2. 如果 parsedReferrer 失败，则抛出一个 TypeError
        3. 如果下面一个是真的
            - parsedReferrer 的 cannot-be-a-base-URL 标志被设置，方案是“about”，并且 path 包含单一的字符串“client”
            - parsedReferrer 的 origin 和 origin 不是同源
        4. 否则设置 request 的 referrer 为 parsedReferrer
16. 如果 init["referrerPolicy"] 存在，则设置请求的引用策略为它。
17. 如果 init["mode"] 存在，则设置 mode 为它，否则设置为 fallbackMode。
18. 如果 mode 是“navigate”，则抛出一个 TypeError
19. 如果 mode 不是 null，则设置 request 的 mode 为 mode
20. 让 credentials 为 init[“credentials”]，如果它存在，否则设置为 fallbackCredentials。
21. 如果 credentials 不是 null，则设置 request 的 credentials mode 为它。
22. 如果 init["cache"] 存在，则设置 request 的缓存模式为它
23. 如果 request 的缓存模式是“only-if-cached”，并且 request 的模式不是“dame-origin”，则抛出一个 TypeError
24. 如果 init["redirect"]存在，则设置 request 的重定向模式为它
25. 如果 init["integrity"]存在，则设置 request 的完整性愿信息为它
26. 如果 init["keepalive"]存在，则设置 request 的 keepalive 标志，如果 init["keepalive"] 为 true，否则不设置
27. 如果 init["method"] 存在，则
    1. 设置 method 为 init["method"]
    2. 如果 method 不是一个方法，或者 method是一个禁止的方法，则抛出一个 TypeError
    3. 规范化 method
    4. 设置 request 的 method 为 method
28. 如果 init["signal"] 窜在，则设置 signal 为它。
29. 让 r 为一个新的 Request 对象，关联 request
30. 如果 signal 不是 null，让 r 的 signal 和 signal 一样
31. 设置 r 的头部到新的 Header 对象，它的头部列表是 request 的头部列表，并且它的 guard 是“request”。
32. 如果 init 不是空的，则：
    1. 让 headers 为 r 的头部和他关联的头部列表
    2. 如果 init["headers"] 村阿紫，则设置 headers 为 init["headers"]
    3. 让 r 的头部的头部列表为空
    4. 如果 r 的请求的模式是“no-cors”，则：
        1. 如果 r 的请求的方法不是 CORS 安全方法，则抛出一个 TypeError
        2. 设置 r 的头部的 guard 为“request-no-cors”
    5. 如果 headers 是一个 Headers 对象，则使用 header 遍历头部列表，拼接 header 的 name/header 的value 到 r 的 Headers 对象。
33. 让 inputBody 为 input 的 request 的 body，如果 input 是一个 Request 对象，否则设置为 null
34. 如果 init["body"]存在并且不是 null，或者 inputBody 是非空的，则 request 的方法是`GET`或者`HEAD`，则抛出一个 TypeError
35. 让 body 为 inputBody
36. 如果 init["body"] 存在并且不是 null，则：
    - 让 Content-Type 为 null
    - 如果 init[keepalive]存在并且是 true，则设置 body 和 Content-Type 为init["body"] 和 keepalive flag 设置的解析结果
    - 否则，设置 body 和 Content-Type 为解析 init["body"] 的结果
    - 如果 Content-Type 不是 null，并且 r 的头部的头部列表不包含`Content-Type`，则拼接`Content-Type`/Content-Type 到 r 的头部
37. 如果 body 不是 null，并且 body 的source 不是 null，则：
    1. 如果 r 的请求的模式不是“same-origin”也不是“cors”，则抛出一个 TypeError。
    2. 设置 r 的请求的 sue-CORS-preflight 标志
38. 如果 inputBody 是 body 并且 input 是 disturbed 或者 locked，则抛出 TypeError
39. 如果 inputBody 是 body，并且 inputBody 不是 null，则
    1. 设置 rs 为一个 ReadableStream 对象，它读出来的数据和 inputBody 的 stream 读出来的数据完全一样
    2. 设置 body 为一个新的 body，它的 stream 是 rs，它的 source 是 inputBody 的source，并且全部字节是 inputBody 的全部字节。
40. 设置 r 的request 的 body 为 body
41. 设置 r 的 MIME 类型为从 r 的请求的头部列表解析 MIME 类型的结果
42. 返回 r。

method 属性获取器调用的时候，必须放回上下文对象的请求的方法

url 属性获取器调用的时候，必须返回上下文对象的请求的 URL

headers 属性获取器调用的时候，必须返回上下文对象的头部

destination 属性获取器调用的时候，必须返回上下文对象的 destination

referrer 属性获取器调用的时候，必须返回空字符串，如果上下文对象的请求的引用是“no-referrer”，如果上下文对象的请求的引用是“client”，则返回“about:client”，否则返回上下文对象的请求的引用的序列化结果

referrerPolicy 属性获取器调用的时候，必须返回上下文对象的请求的引用策略

mode 属性获取器调用的时候，必须返回上下文对象的模式

credentials 属性获取器调用的时候，必须返回上下文对象的请求的认证模式

cache 属性获取器调用的时候，必须返回上下文对象的请求的缓存模式

redirect 属性调用的时候，必须返回上下文对象的请求的重定向模式

integrity 属性获取器调用的时候，必须返回上下文对象的请求的完整性元数据

keepalive 属性获取器调用的时候，如果上下文对象的请求的 keepalive 标志设置了，则返回 true，否则返回 false

isReloadNavigation 属性获取器调用的时候，必须返回 true，如果上下文对象的 reload-navigation 标志设置，否则返回 false

isHistoryNavigatin 属性获取器调用的时候，如果上下文对象的请求的 history-navigation 设置了，则返回 true，否则返回 false

signal 属性获取器调用的时候，必须返回上下文对象的 signal

clone() 方法调用的时候，必须执行下面的步骤：
1. 如果上下文对象是 disturbed 或者 locked，则抛出一个 TypeError
2. 让 clonedRequestObject 为一个新的 Request 对象
3. 让 clonedRequest 为克隆上下文对象的请求的结果
4. 设置 clonedRequestObject 的请求为 clonedRequest
5. 设置 clonedRequestObject 的头部为新的 Headers 对象，并使用下面的属性：
    - 头部列表： clonedRequest 的头部列表
    - guard：上下文对象头部的 guard
6. 让 clonedRequestObject 的 signal 和上下文对象的 signal 一致
7. 返回 clonedRequestObject。

### 5.4 响应类
```
[Constructor(optional BodyInit? body = null, optional Responseinit init = {}), Exposed=(Window,Worker)]
interface Response {
    [NewObject] static Response error();
    [NewObject] static Response redirect(USVString url, optional unsigned short status = 302)
    readonly attribute ResponseType type;

    readonly attribute USVString url;
    readonly attribute boolean redirected;
    readonly attribute unsigned short status;
    readonly attribute boolean ok;
    readonly attribute ByteString statusText;
    [SameObject] readonly attribute Headers headers;
    readonly attribute Promise<Headers> trailer;

    [NewObject] Response clone();
}
Response includes Body;

directionary ResponseInit {
    unsigned short status = 200;
    ByteString statuText = "";
    HeadersInit headers;
}

enum ResponseType { "basic", "cors", "default", "error", "opaque", "opaqueredirect" }
```

一个 Response 对象有一个关联的 response（一个 reponse）

一个 Response 对象有一个关联的 headers（null 或者 Headers 对象），初始化为 null。

一个 Response 对象有一个关联的 trailer promise（一个 promise）。注意：？？？

Response(body, init) 构造器，当调用的时候，必须执行下面的步骤：
1. 如果 init["status"] 不是 200 - 599 闭区间内的值，则抛出 RangeError。
2. 如果 init["statusText"] 不匹配原因章节生产式，则抛出一个 TypeError。
3. 让 r 为新的 Response 对象，关联一个新的响应
4. 让 r 的 headers 为一个新的 Headers 对象，它的头部列表是 r 的响应的头部列表，并且 guard 是“response”
5. 设置 r 的 response 的 status 为 init["status"]
6. 设置 r 的 response 的 status message 为 init["statusText"]
7. 如果 init["headers"] 存在，则使用 init["headers"] 填充 r 的 headers
8. 如果 body 不是 null， 则：
    1. 如果 init["state"] 是空正文状态，则抛出一个 TypeError
    2. 让 Content-Type 为 null。
    3. 设置 r 的 response 的 body 和 Content-Type 为解析 body 的结果
    4. 如果 Content-Type 不是 null，并且 r 的响应的头部列表不包含`Content-Type`，则拼接`Content-Type`/Content-Type 到 r 的 response 的头部列表
9. 设置 r 的 MIME 类型为从 r 的 response 的头部列表解析 MIME 类型的结果
10. 设置 response 的 HTTPS 状态为当前设置对象的 HTTP status。
11. 使用一个 guard 为“immutable”的新的 Headers 对象 resolve r 的 trailer promise。
12. 返回 r。

静态的 error() 方法调用的时候，必须执行下面的步骤：
1. 让 r 为新的 Response 对象，它 response 是一个网络错误
2. 让 r 的头部为新的 Headers 对象，它的 guard 为“immutable”
3. 返回 r。

静态的 redirect(url, status) 方法调用的时候，必须执行下面的步骤：
1. 让 parsedURL 为使用当前设置对象的 API base URL 解析 url 的结果
2. 如果 parsedURL 失败，则抛出 TypeError
3. 如果 status 不是一个重定向状态，则抛出 RangeError
4. 让 r 为新的 Response 对象，它的 response 是新的 response
5. 设置 r 的头部为新的 Headers 对象，它的 guard 是“immutable”
6. 设置 r 的 response 的 status 为 status
7. 在 r 的响应的头部列表拼接`Location`到 parsedURL，序列化并同步编码，
8. 返回 r。

type 属性获取器，当调用的时候，必须返回上下文对象的响应的 type

url 属性获取器，当调用的时候，如果上下文对象的响应的 URL 是 null，则返回空字符串，否则，返回上下文对象的响应的 URL 在 exclude-fragment flag 设置的情况下序列化的结果，

redirected 属性获取器调用的时候，必须返回 true，如果上下文对象的响应的 URL 列表有多余一个，否则返回 false。

注意：为了过滤出重定向结果的响应，直接通过这个 API，比如 fetch（url, {redirect:"error"}）。通过这种方式，一个不安全的 response 不会被意外的遗漏。

status 属性获取器，当调用的时候，必须返回上下文对象的响应的状态。

ok 属性获取器，当调用的时候，如果上下文对象响应的状态是 ok 状态的时候，必须返回 true，否则返回 false。

statusText 属性获取器，当调用的时候必须返回上下文对象响应的 status。

headers 属性获取器，当调用的时候，必须返回上下文好对象的头部。

trailer 属性获取器，当调用的时候，必须返回上下文对象的 trailer promise

clone() 方法调用的时候，必须执行下面的步骤：
1. 如果上下文对象是 disturbed 或者 locked，则抛出一个 TypeError
2. 让 clonedResponseObject 为一个新的 Response 对象。
3. 让 clonedResponse 为克隆上下文对象响应的结果。
4. 设置 clonedResponseObject 的 response 为 clonedResponse
5. 设置 clonedResponseObject 的 头部为新的 Headers 对象，它的头部列表设置为 clonedResponse 的头部列表，并且它的 guard 是上下文对象的 guard
6. 一旦上下文对象的 triler promise 完成了，使用 guard 为”immutable“，头部列表是 clonedResponse 的 trailer 的Header 对象 resolve clonedResponseObject 的 trailer promise。
7. 返回 clonedResponseObject
8. 返回 clonedResponse


### 5.5 获取方法
```
partial interface mixin WindowOrWorkerGlobalScope {
    [NewObject] Promise<Response> fetch(RequestInfo input, optional RequestInit init);
}
```
fetch(input, init) 方法调用的时候，执行下面的步骤：
1. 让 p 为新的 promise。
2. 让 requestObject 为使用 input 和 init 作为参数构造调用 Request 的初始结果，如果失败了，使用它 reject，并返回 p
3. 让 request 为 requestObject 的 reject
4. 如果 requestObject 的 signal 的 aborted flag 设置了，则：
    1. 使用 p，request，和 null 放弃 fetch
    2. 返回 p
5. 如果 request 的 client 的全局对象是一个 ServiceWorkerGlobalScope 对象，则设置 request 的 service-worker mode 为“none”
6. 让 responseObject 为一个新的 Response 对象，并和一个 guard 为“immutable”的新的 Headers 对象关联
7. 让 locallyAborted 为 false。
8. 添加下面的放弃步骤到 requestObject 的 signal：
    1. 设置 locallyAborted 为 true
    2. 使用 p，request，和 responseObject 放弃 获取。
    3. 设置放弃标志并终止正在进行的获取
9. 同步执行下面的步骤：
    获取请求：
    处理 response 的 response，执行下面子步骤：
    1. 如果 locallyAborted 是 true，终止这些子步骤
    2. 如果 response 的 aborted 标志被设置，则使用 p，request 和 responseObject 放弃 fetch，并终止这些子步骤
    3. 如果 response 是网路哦错误，则使用 TypeError reject p 并终止这些子步骤
    4. 关联 responseObject 和 response
    5. 使用 responseObject resolve p
    处理 response 的 done，执行下面的子步骤：
    1. 如果 locallyAborted 是 true，终止这些子步骤
    2. 让 trailerObject 为一个新的 Headers 对象，它的 guard 是“immutable”
    3. 如果 response 的 trailer failed 标志被设置，则：
        1. 如果 response 的 aborted 标志被设置，使用一个“AbortError”DOMException 来 reject responseObject 的 trailer promise
        2. 否则，使用一个 TypeError 来 reject responseObject 的 trailer promise
        3. 终止这些子步骤
    4. 关联 responseObject 和 response
    5. 使用 responseObject resolve p
10. 返回 p

使用 promise，request，和 responseObject 放弃获取，执行下面步骤
    1. 让 error 为一个“AbortError” DOMException
    2. 使用 error reject promise
    3. 如果 request 的 body 不是 null，并且是可读的，则使用 error 取消请求的 body
    4. 如果 responseObject 是 null，则返回
    5. 使用 error reject responseObject 的 trailer promise
    6. 让 response 为 responseObject 的 response
    7. 如果 responseObject 的 body 不是 null，并且是可读的，则使用 error 来 error response 的 body

### 5.6 垃圾收集
用户代理可以终止获取，因为终止不可以被观察
```
fetch("https://www.example.com/")
```
用户代理不能终止获取，因为终止可以通过 promise 观察到：
```
window.promise = fetch("https://www.example.com/")
```
用户代理可以终止获取，因为关联的正文不能被观察：
```
window.promise = fetch("https://www.example.com/").then(res => res.headers)
```
用户代理可以终止获取，因为终止不能被观察到：
```
fetch("https://www.example.com/").then(res => res.body.getReader().closed)
```
用户代理不能终止获取，因为可以通过为 promise 对象注册一个处理器来观察到终止：
```
window.promise = fetch("https://www.example.com/")
  .then(res => res.body.getReader().closed)
```
用户代理不能终止回去，因为终止可以通过注册处理器观察到。
```
fetch("https://www.example.com/")
  .then(res => {
    res.body.getReader().closed.then(() => console.log("stream closed!"))
  })
```
(上面不能观测的栗子假设内置的属性和函数没有被重写，比如 body.getReader())。

### 6. WebSocket 协议修改

### 7. data: URL
关于 data: URL 的详细信息，可以查阅 RFC2397。这个章节替换了 RFC 的规范过程需求，为了兼容部署的内容。

一个 data: URL 结构是一个由 MIME 类型（一个 MIME 类型）和一个正文（一个比特序列）组成的。

一个 data: URL 处理器接收一个 URL dataURL，并执行以下步骤：
1. 断言：dataURL 的方案是 data:
2. 让 input 为在 dataURL 上执行 URL 序列化的结果，并设置忽略片段标志。
3. 移除 input 开头的“data:”
4. 让 position 指向 input 的开头
5. 让 mimeType 为收集码点序列的结果，不等于 U+002C(,)，给定 position
6. 移除 mimeType 开始和结尾的 ASCII 空白。
> 注意：这将只会移除 U+0020 SPACE 码点。
7. 如果 position 超过 input 的长度，则返回失败。
8. position 自增 1
9. 让 encodeedBody 为 input 剩余内容
10. 让 body 为 encodedBody 字符串百分比编码
11. 如果 mimeType 以 U+003B(;) 结尾，后面跟着0或者更多的 U+0020 SPACE，后面跟着一个 ASCII 大小写不敏感命中“base64”，则：
    1. 让 stringBody 为 body 的同形解码
    2. 设置 body 为 stringBody 的宽容 base64 解码
    3. 如果 body 失败，则返回失败
    4. 移除 mimeType 最后6个码点
    5. 移除 mimeType 的最后的 U+0020 SPACE
    6. 移除 mimeType 最后的 U+003B(;) 码点
12. 如果 mimeType 以 U+003B(;) 开始，则添加 mimeType 前缀"text/plain"
13. 让 mimeTypeRecord 为转化 mimeType 的结果
14. 如果 mimeTypeRecord 失败，则设置 mimeTypeRecord 为 text/plain;charset=US-ASCII
15. 返回一个新的 data:URL 结构，它的 MIME 类型是 mimeTypeRecord，并且正文是 body。

### 背景阅读