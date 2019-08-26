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

