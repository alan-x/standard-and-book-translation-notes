### 概述
XMLHttpRequest 标准定义了一个 API，它提供客户端使用脚本在客户端和服务端之间传输数据的功能。

### 1. 介绍
**这个章节不是规范的**
XMLHttpRequest 对象是一个用于获取资源的 API。

XMLHttpRequest 这个名字是历史遗留下来的，和他的功能没啥关系。

Example
下面是操作从互联网获取到的 XML 文档中的数据的一些简单代码
```
function processData(data) {
  // taking care of data
}

function handler() {
  if(this.status == 200 &&
    this.responseXML != null &&
    this.responseXML.getElementById('test').textContent) {
    // success!
    processData(this.responseXML.getElementById('test').textContent);
  } else {
    // something went wrong
    …
  }
}

var client = new XMLHttpRequest();
client.onload = handler;
client.open("GET", "unicorn.xml");
client.send();
```
如果你只是想记录一条信息到服务端：
```
function log(message) {
  var client = new XMLHttpRequest();
  client.open("POST", "/log");
  client.setRequestHeader("Content-Type", "text/plain;charset=UTF-8");
  client.send(message);
}
```
或者你想要检查一个文档在服务端的状态：
```
function fetchStatus(address) {
  var client = new XMLHttpRequest();
  client.onload = function() {
    // in case of network errors this might not give reliable results
    returnStatus(this.status);
  }
  client.open("HEAD", address);
  client.send();
}
```
### 1.1 规格历史
XMLHttpRequest 对象一开始定义为 WHATWG 的 HTML 的一部分。（基于微软很多年以前的实现）它在 2006 年移动到 W3C。XMLHttpRequest 的扩展（比如，进度事件和跨域请求）直到 2011 年末在一个分离的草案被开发（XMLHttpRequest Level 2），此时两个草案已经被合并，XMLHttpRequest 从标准角度又开始成为一个单一的实体。2012 年末又移回了 WHATWG。

到现在这个草案的讨论可以在下面的邮件列表档案找到：
- whatwg@whatwg.org
- public-webapps@w3.org
- public-webapi@w3.org
- public-appformats@w3.org


### 2. 一致性
这个规则中所有的图，例子，和笔记都是非规范的，所有明确的标记为非规范的章节也是一样。其他的一切都是规范的。

在这个规格中规范部分的关键字“必须”，“不必须”，“需要”，“应该”，“不应该”，“推荐”，“可能”，和“可选”可以用 RFC2019 中的描述解释。为了可读性，这些词汇不会以全大写的形式出现在这个规则中。[RFC2119]

### 2.1 扩展性
强烈鼓励用户代理，工作组，和其他相关方和 WHATWG 社区讨论新特性。

### 3. 术语
这个规格使用的术语，链接跨域 DOM，DOM 的转换和序列化，编码，特性策略，获取，文件 API，HTML，URL，WEB IDL，和 XML。

它使用 HTML 的印刷规范。[HTML]

### 4. 接口 XMLHttpRequest
```
IDL
[Exposed=(Window,DedicatedWorker,ShareWorker)]
interface XMLHttpRequestEventTarget : EventTarget {
    // 事件处理器
    attribute EventHandler onloadstart;
    attribute EventHandler onprogress;
    attribute EventHandler onabort;
    attribute EventHandler onerror;
    attribute EventHandler onload;
    attribute EventHandler ontimeout;
    attribute EventHandler onloadend;
}

[Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequestUpload : XMLHttpRequestEventTarget {

};

enum XMLHttpRequestResponseType {
    "",
    "arraybuffer",
    "document",
    "json",
    "text"
}

[Constructor exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequest : XMLHttpRequestEventTarget {
    // 事件处理器
    attribute EventHandle onreadystatechange;

    // 状态
    const unsigned short UNSENT = 0;
    const unsigned short OPENED = 0;
    const unsigned short HEADERS_RECEIVED = 0;
    const unsigned short LOADING = 0;
    const unsigned short DONE = 0;
    readonly attribute unsigned short readyState;

    // 请求
    void open(ByteString method, USVString url);
    void open(ByteString method, USVString url, boolean async, optional USVString? username = null, optional USVString? pasword = null);
    void setRequestHeader(ByteString name, ByteString value);
        attribute unsigned long timeout;
        attribute boolean withCredentials;
    [SameObject] readonly attribute XMLHttpRequestUpload upload;
    void send(optiondal (Document or BodyInit)? body = null);
    void abort();

    // 响应
    readonly attribute USVString responseURL;
    readonly attribute unsigned short status;
    readonly attribute ByteString statusText;
    ByteString? getResponseHeader(ByteSring name);
    ByteString getAllResponseHeader();
    void overrideMimeType(DOMString mime);
        attribute XMLHttpRequestResponseType responseType;
    readonly attribute any response;
    readonly attribute USVString responseText;
    [Exposed=Window] readonly attribute Document? responseXML;
}
```
一个 XMLHttpRequest 对象有一个关联的 XMLHttpRequestUpload 对象。

一个 XMLHttpRequest 对象有一个关联的*状态*，是 unset，opended，headers received，loading，和 done 中的一种。除非被设置否则他是 unset。

一个 XMLHttpRequest 对象有一个关联的 send() 标志。除非设置否则就是 unset。

### 4.1 构造器
```
对于 web 开发者（非规范）
client = new XMLHttpRequest()
    返回一个新的 XMLHttpRequest 对象
```
当 XMLHttpRequest() 构造器调用的时候，必须返回一个新的 XMLHttpRequest 对象。

### 4.2 垃圾收集

当一个 XMLHttpRequest 对象的*状态*是 opened 并且 end() 标志被设置，header received，或者 loading，并且它有一个或者多个类型为 readystatechange，progress，abort，error，load，timeout，和 loadend 中个一个的事件监听器注册的时候，它不应该被垃圾收集。

当一个 XMLHttpRequest 链接在打开的时候被垃圾收集，那么用户代理必须停止 XMLHttpRequest 对象正在进行的获取操作。

### 4.3 事件处理器
下面是实现从 XMLHttpRequestEventTarget 作为属性继承的接口的对象必须支持的事件处理器（和他们相符的事件处理器事件类型）：

| 事件处理器 | 事件处理器事件类型 |
| - | - |
| onloadstart | loadstart |
| onprogress | progress |
| onabort | abort |
| onerror | error |
| onload | load |
| ontimeout | timeout|
| onloadend | loadend |

下面是 XMLHttpRequest 对象单独作为属性必须支持的事件处理器（和他们相符的事件处理器事件类型）：

| 事件处理器 | 事件处理器事件类型 |
| - | - |
| onreadystatechange | readystatechange |

### 4.4 状态
```
对于 web 开发者（非规范）
    client .  readyState
      返回客户端的[状态]()
```
readyState 属性获取器必须返回下面表格第二列的值，第一列则是上下文对象的状态值：

| | | |
| - | - | - |
| unsent | UNSENT(数字值 0) | 对象被构造了 |
| opend | OPENED(数字值 1) | open() 方法被成功调用了。在这个状态，请求可以使用 setRequestHeader() 来设置状态，并且可以使用 send() 方法初始化请求。 |
| headers received | HEADERS_RECEIVED(数字值 2) | 所有的重定向（如果存在）已经被遵循，并且所有响应的的 HTTP 头部已经接收到。 | 
| loading | LOADING(数字 3) | 响应体被接收 |
| done | DONE(数字值 4) | 数据传输已经完成或者传输出错（比如，无限重定向）。 |

### 4.5 请求

每一个 XMLHttpRequest 对象都有一系列请求相关的概念：*请求方法*。*请求 URL*，*作者请求头部*，*请求体*，*同步标志*，*上传完成标志*，*上传监听标志*，和*超时标志*。

用户请求头部 初始化为一个空的头部列表。

请求题初始化为空。

同步标志，上传完成标志，上传监听标志和超时标记初始化为未设置。

> 注意：在 XMLHttpRequestUpload 对象注册一个或者多个事件监听器将会导致一个 CORS-预请求。（这是因为注册一个事件监听器导致上传监听标志被设置，导致使用 CORS 预请求标志被设置）

4.5.1 open() 方法

对于 web 开发者（非规范）
```
  client . open(method, url [, async = true [, username = null [, password = null]]])
    设置请求方法，请求 URL，和同步标志
    
    如果*方法*不是一个合法的 HTTP 方法或者*url*不能被解析，抛出“SyntaxError”的 DOMException。

    如果*方法*是大小写不敏感的`CONNECT`、`TRACE`、`TRACK`，则抛出“SecurityError”的“DOMException”。

    如果*async*是 false，当前的全局对象是一个 Window 对象，并且 timeout 属性不是 0，或者 responseType 属性不是 空字符串，则抛出“InvalidAccessError”的 DOMException。
```
> 注意：wocker 之外同步的 XMLHttpRequest 正在从 web 平台移除，因为它对终端用户的体验有很坏的影响。（这是一个需要很多年的过程）开发者必须不传输 false 给 async 参数。当当前的全局对象是一个 Window 对象。强烈鼓励用户代理在开发工具中警告这种使用，并在出现时尝试抛出“InvalidAccessError”的 DOMException。

open(method, url) 和 open(method, url, async, username, password) 方法调用的时候，必须执行下面的步骤：

1. 让 settingsObject 为上下文对象响应的设置对象
2. 如果 settingsObjecct 有负责的文档并且它不是完全活跃的，则爆出一个"InvalidStateError"的 DOMException。
3. 如果 method 不是一个方法，则抛出“SyntaxError”的 DOMException。
4. 如果方法是禁用的方法，则抛出“SecurityError”的 DOMException。
5. 正常化方法。
6. 让 parsedURL 为 settingsObject 的 API 基本 URL 和 settingObjects API URL 字符编码的转化结果。
7. 如果 parsedURL 失败，则抛出“SyntaxError”的 DOMException。
8. 如果 async 参数缺失，设置 async 为 true，并设置 username 和password 为空。
> 注意：不幸运的是，当这个参数缺失的时候，遗留内容阻止对待 async 参数为 undefined 。
9. 如果 parsedURL 的 host 不是空的，那么：
  1. 如果 username 参数不是空，使用给定的 parsedURL 和 username 设置 usersename。
  2. 如果 password 参数不是空，使用给定的 parsedURL 和 password 设置 password。
10. 如果 async 是 false，当前全局对象是一个 Window 对象，并且 timeout 属性值是非 0 或者 responseType 属性值不是空字符串，则抛出“InvalidAccessError”的 DOMException。
11. 终止 XMLHttpRequest 独享正在进行的请求。
> 注意：一个请求可以在这个点继续进行。
12. 如下设置对象的关联变量：
  - 取消设置 send() 标志和上传监听标志。
  - 设置请求方法为 method。
  - 设置请求 URL 为 parsedURL
  - 如果 async 是 false，设置同步标志，否则设置同步标志。
  - 将作者请求头部置空。
  - 设置响应为网络错误。
  - 设置接收比特为空比特序列。
  - 设置响应对象为空。
> 注意：覆盖 MIME 类型不是在这里被覆盖，overrideMimeType() 方法可以在 open() 方法调用前调用。
13. 如果状态不是 opended，那么：
    1. 设置状态为 opended。
    2. 在 this 上触发 readystatechange 事件。
> 注意：这里有两个 open() 方法定义的原因是 XMLHttpRequest 标准编写工具的限制。

4.5.2 setRequestHeader() 方法
```
对于 web 开发者（非规范）
  client . setRequestHeader(name, value)
    在作者请求头部绑定一个头部

    如果状态是 opened 或者 send() 标志被设置， 则抛出一个“InvalidStateError”的 DOMException。

    如果 name 不是一个头部名字或者值不是一个头部值，则抛出“SyntaxError”的 DOMException。
```
setRequestHeader(name, value)方法必须执行一下步骤：
  1. 如果状态不是 opened，则抛出“InvalidStateError”的 DOMException。
  2. 如果 send() 标志被设置，则抛出“InvalidStateError”的 DOMException。
  3. 规范化值
  4. 如果 name 不是 name 或者 value 不是 value，则抛出“SyntaxError”的 DOMException。
    > 注意：一个空比特序列表示一个空的头值。
  5. 如果 name 是一个禁止头部名，终止这些步骤。
  6. 在作者请求头部中绑定 name/value
  
  ```
  例子
  一些简单的代码示范了设置相同的头部两次会发生什么。
  // 下面的脚本：
  var client = new XMLHttpRequest();
  client.open('GET', 'demo.cgi');
  client.setRequestHeader('X-Test', 'one');
  client.setRequestHeader('X-Test', 'two');
  client.send();

  // …导致发送下面的头部:
  // X-Test: one, two
  ```

### 4.5.3 timeout 属性
```
对于 web 开发者（非规范）
  client . timeout
    可以设置一个毫秒级的单位。当设置为一份非 0 的值的时候，将会导致在给定的时候之后获取将会被终结。当事件过去，请求还没有完成，并且同步标志没有被设置，一个 timeout 事件将会被派发，或者一个“TimeoutException”的 DOMException 将会被抛出（对于 send() 方法）。

    当设置的时候，如果同步标志被设置，并且当前的全局对象是一个 Window 对象，将会抛出一个“InvalidAccessError”的 DOMException。
```
timeout 属性必须返回它的值。初始值必须设置为0。

设置 timeout 属性必须执行下面的步骤：

1. 如果当前的全局对象是 Window 对象，并且同步标志被设置，那么抛出一个“InvalidAddressError”的 DOMException。
2. 设置它的值为新的值。

> 注意：这意味着 timeout 属性可以在获取过程中设置，如果这发生了，它将会以获取开始的事件继续测量。

### 4.5.4 withCredentials 属性
```
对于 web 开发者（非规范）
  client. withCredentials
    跨域请求的时候，如果需要认证，那么就是 true。如果需要在跨域请求中排除并在响应中忽略 cookie，则为 false。初始值为 false。

    当设置的时候，如果*状态*是 unsent 或者 opened，或者 send() 标志位被设置，那么就抛出“InvalidStateError”的 DomException。
```

withCredentials 属性必须返回它的值。初始值必须是 false。
设置 withCredentials 属性必须执行以下步骤：
1. 如果*状态*是 unsent 或者 opened，就抛出“InvalidStateError”的 DOMException。
2. 如果 send() 标志被设置，就抛出“InvalidStateError”的 DOMException。
3. 设置 withCredentials 属性的值为给定的值。

> 注意：withCredentials 属性对于同源资源获取没有用处。

### 4.5.5 upload 属性
```
对于 web 开发者（非规范）
  client . upload
    返回关联的 XMLHttpRequestUpload 对象。当数据传输给服务端的时候，它可以用来收集传输信息。
```

upload 属性必须返回关联的 XMLHttpRequestUpload 对象。

> 注意：就像早前指出的，每一个 XMLHttpRequest 对象有一个关联的 XMLHttpRequestUpload 对象。

### 4.5.6 send() 方法

```
对于 web 开发者（非规范）

  client . send([body = null])
    初始化请求。body 参数提供请求体，如果请求方法是 GET 或者 HEAD，将会忽略任何请求体

    如果*状态*是 opened 或者 send() 标志被设置，则抛出“InvalidStateError”的 DOMException。
```
send(body)方法必须执行下面的步骤：
1. 如果*状态*不是 opened，则抛出“InvalidStateError”的 DOMException。
2. 如果 send() 标志被设置，则抛出“InvalidStateError”的 DOMException。
3. 如果请求方法是 GET 或者 HEAD，设置 body 为 null。
4. 如果 body 不是 null，则：
    1. 让 extractedContentType 为 null。
    2. 如果 body 是一个 Document，那么设置请求体为 body，序列化，转化成 Unicode 和 UTF-8 编码。
    3. 否则，设置请求体和 extractedContentType 为 body 的解析结果
    4. 如果用户请求头部包含`Content-Type`，则：
        1. 如果 body 是一个 Document 或者一个 USVString，那么：
            1. 让 originalAuthorContentType 为作者请求头部中名字为字节大小写不敏感的`Content-Type`的值，
            2. 让 contentTypeRecord 为 originalAuthorContentType 的转化结果。
            3. 如果 contentTypeRecord 没有失败，contentTypeRecord 的参数 charset 存在，并且参数 charset 不是一个 ASCII 大小写不敏感的“UTF-8”，则：
                1. 设置 contentTypeRecord 的参数 carset 为“UTF-8”。
                2. 让 newContentTypeSerialized 为序列化 contentTypeRecord 的结果
                3. 设置`Content-Type`/newContentTypeSerialized 到作者请求头部
    5. 否则：
      1. 如果 body 是一个 HTMl docuemnt，设置`Content-Type`/`text/html;charset=UTF-8`到作者请求头部。
      2. 否则，如果 body 是一个 XML document，设置`Content-Type`/`application/xml;charset=UTF-8`到作者请求头部。
      3. 否则，如果 extractedContentType 不是 null，设置 `Content-Type`/extractedContentType 到作者请求头部。
5. 如果一个或者多个事件监听器被注册到关联的 XMLHttpRequestUpload 对象，然后设置请求监听器的标志。
6. 让 req 为一个新的请求，初始化如下：
    - 方法：请求方法
    - url：请求 url
    - 头部列表：作者请求头部
    - 不安全请求位：设置
    - 请求体：请求体
    - 客户端：上下文对象的相关设置对象
    - 同步标志：如果同步标志被设置，就设置。
    - 模式：“cors”
    - 使用 CORS 预检标志：如果上传监听器标志被设置，就设置
    - 认证模式：如果 withCredentials 属性值为 true，则是“include”，否则就是“same-origin”。
    - 使用 URL 认证标志：如果请求 URL 的 username 和 password 都不为空，就设置。
7. 不设置上传完成标志
8. 不设置超时标志
9. 如果 req 的请求体是 null， 设置上传完成标志。
10. 设置 send() 标志。
11. 如果同步标志没有设置，则：
    1. 以 0  和 0 在 this 上触发一个名为 loadstart 的处理事件
    2. 如果上传完成标志没有设置，并且上传监听标志被设置，那么就使用 0  和 req 的请求体的总字节数在 this 上触发名为 loadstart 的处理事件。
    3. 如果*状态*是 opened 或者 send() 标志没有被设置，则返回。
    4. 获取 req，按下面每一步来处理网络任务源上的任务队列。
    
    同步运行以下步骤：
      1. 等待直到 req 的完成标志被设置或者
          1. 当步骤开始到timeout 属性值毫秒数过去
          2. 当 timeout 属性值不是 0。
      2. 如果 req 的完成标志未设置，则设置超时标志并终止获取。
      
    为了处理请求的请求体，运行以下步骤：
      1. 如果大约 50ms 内这些步骤调用过，终止这些步骤
      2. 如果上传监听标志被设置，则使用请求体的以传输字节数和请求体的总字节数在 this 的 XMLHttpRequestUpload 对象触发一个名为 progress 的处理事件
    >  注意：这些步骤只会在有新的字节被传输的时候触发。

    为了处理请求体结束请求，运行下面这些步骤：
      1. 设置上传完成标志。
      2. 如果上传监听标志位未设置，则终止这些步骤。
      3. 让 transmitted 为请求体的传输字节数。
      4. 让 length 为请求体的总字节数。
      5. 使用 transmitted 和 length 在 this 的 XMLHttpRequestUpload 对象上触发名为 progress 的处理事件
      6. 使用 transmitted 和 length 在 this 的 XMLHttpRequestUpload 对象上触发名为 load 的处理事件
      7. 使用 transmitted 和 length 在 this 的 XMLHttpRequestUpload 对象上触发名为 loaded 的处理事件

    为了处理响应，执行以下步骤：
      1. 设置 response 为响应。
      2. 为 response 处理错误。
      3. 如果 response 是一个网络错误，返回。
      4. 设置*状态*为 headers received。
      5. 在 this 上触发名为 readystatechange 的事件。
      6. 如果状态不是 headers receive，那么就返回。
      7. response 的请求体是 null，那么执行处理响应体并返回。
      8. 让 reader 为从 response 请求体流中获取一个 reader 的结果。
        > 这个操作不会抛出一个异常
      9. 让 read 为使用 reader 从请求体流中读取串的结果
        当 read 被一个 done 属性是 false 并且 value 属性是一个 Uint*Array 的对象满足的时候，一遍又一遍的执行下面的步骤：
          1. 拼接 value 属性到接受到的比特。
          2. 如果从调用大约50ms，则终止这些步骤
          3. 如果*状态*是 headers received，那么设置*状态*为 loading。
          4. 在 this 上触发名为 readystatechange 到事件。
          > 注意：web 兼容性是 readystatechange 比 state 改变触发更经常的原因。
          5. 使用 response 的响应体的传输比特和响应体的全部比特在 this 上触发名为 progress 的处理事件。
          >  注意：这些步骤只会在有新的字节被传输的时候触发。
          当 ready 被一个 done 属性为 true 的对象满足的时候，为 response 执行处理响应体。

        当 read 被一个异常拒绝的时候，为 response 执行错误处理。

12. 否则，如果同步标志被设置，执行这些步骤：
    1. 如果上下文对象的相关设置对象又一个负责的 document，并且该 document 不允许使用“同步的 xhr”，则为一个网络错误执行处理请求体结束并返回。
    2. 让 response 为 req 获取的结果。
      如果 timeout 属性值不是 0，并且它没有在 timeout 指定的毫秒数内返回，则设置超时标志并终止请求。
    3. 如果 response 的响应体不是 null， 则执行处理响应体结束并返回。
    4. 让 reader 为从 response 响应体中获取流的结果。
    > 这个操作不会抛出一个异常
    5. 让 promise 为使用 reader 从 response 的响应体流读取所有字节的结果
    6. 等待 promise 转化成 fullfilled 或者 rejected。
    7. 如果 promise 是 fullfilled 并伴随着 bytes，则拼接 bytes 到接收到的比特中。
    8. 为 response 执行处理响应结束
    
为 response 处理响应体结束，执行下面的步骤：
  1. 如果同步标志被设置，则设置 response 为 response。
  2. 为 response 处理错误
  3. 如果 response 是一个网络错误，则返回。
  4. 如果同步标志为被设置，使用 response 更新响应体。
  5. 让 trasmitted 为 response 的响应体的传输比特。
  6. 让 length 为 response 的相应体的总比特。
  7. 如果同步标志为被设置，使用 transmiited 和 length 在 this 上出触发名为 progress 的处理事件。
  8. 设置*状态*为 doen。
  9. 不设置 send() 标志。
  10. 在 this 上触发名为 readystatechange 事件。
  11. 使用 transmitted 和 length 在 this 上触发名为 load 的处理事件。
  12. 使用 transmitted 和 length 在 this 上触发名为 loaded 的处理事件。

为 response 处理错误执行以下步骤：
  1. 如果 send() 未设置，返回。
  2. 如果超时标志被设置，则为事件 timeout 执行请求错误步骤和异常“TimeoutException”的 DOMException。
  3. 如果 response 是一个网络错误，则为事件 error 执行请求错误步骤和异常“NetworkError”的 DOMException。
  4. 否则，如果 response 的 请求体流是错误的，那么：
      1. 设置*状态*为 done。
      2. 不设置 send() 标志。
      3. 设置 response 为一个网络错误。
  5. 否则，如果 response 放弃标志被设置，则为事件 abort 执行请求失败步骤和异常“AbortError”的 DOMException。

事件 event 的请求错误步骤和可选的异常 exception 如下：

1. 设置*状态*为 done。
2. 不设置 send() 标志。
3. 设置 response 为网络错误。
4. 如果同步标志被设置，抛出一个异常 exception。
5. 在 this 上触发名为 readystatechange 的事件。
> 注意：此时很明显，同步标记未被设置。
6. 如果上传完成标志为被设置，则：
    1. 设置上传完成请求。
    2. 如果上传监听标志被设置，则：
        1. 使用 0 和 0 在 this 的 XMLHttpRequestupload 对象上触发名为 event 的处理事件。
        2. 使用 0 和 0 在 this 的 XMLHttpRequestupload 对象上触发名为 loadend 的处理事件。
7. 使用 0 和 0 在 this 上触发名为 event 的处理事件。
8. 使用 0 和 0 在 this 上触发名为 loadend 的处理事件。

### 4.5.7 abort() 方法
```
对于 web 开发者（非规范）
  client . abort()
    取消任何网路活动
```
abort() 方法调用的时候，必须执行以下步骤：
1. 通过设置 aborted 标志终止正在进行的获取。
2. 如果*状态*是 opened 并且 send() 标志被设置，headers receiveed，或者 loading，为事件 abort 执行请求错误步骤。
3. 如果状态是 done，则设置状态为 unsent 并且返回一个网络错误。
> 注意：readystatechange 事件已经派发了。

### 4.6 响应

一个 XMLHttpRequest 有一个关联的 response，除非标记，否则是一个网络错误。

一个 XMLHttpRequest 还有一个关联的 接收到的比特（一个比特序列）。除非标记，否则就是一个空的比特序列。

### 4.6.1 responseURL 属性
如果 response url 是 null，responseURL 属性必须返回空的字符串，否则将会使用 exclude fragment flag 把它序列化。

### 4.6.2 status 属性
status 属性必须返回 response 的 status。

### 4.6.3 statusText 属性
statusText 属性必须返回响应的状态信息

### 4.6.3 getResponseHeader() 方法
当 getResponseHeader(name) 方法调用的时候，必须返回从响应的头部列表获取名字的结果。
> 注意：Fetch 标准过滤了响应的头部列表

栗子：
有如下脚本：
```
var client = new XMLHttpRequest();
client.open("GET", "unicorns-are-awesome.txt", true);
client.send();
client.onreadystatechange = function() {
  if(this.readyState == this.HEADERS_RECEIVED) {
    print(client.getResponseHeader("Content-Type"));
  }
}
```
print() 函数将会这样处理：
```
text/plain; charset=UTF-8
```

### 4.6.5 getAllResponseHeader() 方法
如果以下步骤返回 true，则一个比特序列 a 的遗留大写比特小于比特序列 b：
1. 让 A 为 a 的大写比特形式。
2. 让 B 为 b 的大写比特形式。
3. 返回 A 是否在比特上小于比特 B。

getAllResponseHeader() 方法调用的时候，必须执行下面步骤：
1. 让 output 为一个空的比特序列。
2. 让 initialHeaders 为执行排序和绑定响应头部列表的结果。
3. 让 headers 为 initialHeaders 在升序排列的结果，如果 a 的名字的遗留的大写比特小于 b 的名字。
> 注意：不幸的是，这需要部署内容的兼容。
4. 在 headers 使用 header 遍历，拼接 header 的 name，跟随着一个 0x3A 0x20 字节对，跟随着 header 的 value，跟随着 0x0D 0x0A 字节对，到 output。
5. 返回 outout。
> 注意：Fetch 标准过滤了响应的头部列表

栗子：
有以下脚本：
```
var client = new XMLHttpRequest();
client.open("GET", "narwhals-too.txt", true);
client.send();
client.onreadystatechange = function() {
  if(this.readyState == this.HEADERS_RECEIVED) {
    print(this.getAllResponseHeaders());
  }
}
```
print() 函数将会这样处理：
```
connection: Keep-Alive
content-type: text/plain; charset=utf-8
date: Sun, 24 Oct 2004 04:58:38 GMT
keep-alive: timeout=15, max=99
server: Apache/1.3.31 (Unix)
transfer-encoding: chunked
```

### 4.6.6 响应体
响应的 MIME 类型是运行下面步骤的结果：
1. 让 mimeType 为从响应的头部列表解析 MIME 类型的结果。
2. 如果 mimeType 是错误的，那么设置 mimeType 为 text/html。
3. 返回 mimeType。

覆盖 MIME 类型初始化为 null，可以在调用 overrideMimeType() 的时候获取一个值。如果它是 null，最终的 MIME 类型是覆盖 MIME 类型，否则它是响应的 MIME 类型。

最终的字符集是下面步骤运行的结果：
1. 让 label 为 null。
2. 如果响应的 MIME 类型的“charset“参数存在，则设置 label 为它。
3. 如果覆盖 MIME 类型的”charset“参数存在，则设置 label 为它。
4. 如果 label 是空的，则返回空。
5. 让 encoding 为从 label 获取编码的结果。
6. 如果 encoding 是失败，则返回 null。
7. 返回编码。
> 注意：上面的步骤故意不使用最终 MIME 类型，因为它可能返回错误的结果。

一个 XMLHttpRequest 对象有一个关联的响应对象（一个对象，失败，或者 null）。除非标记，否则就是 null。

一个 arraybuffer 响应是执行下面步骤的结果：
1. 设置响应对象为一个新的 ArrayBuffer 对象表示接收到的字节。如果这里抛出一个异常，则设置响应对象为 failure 并返回 null。
> 注意：申请一个 ArrayBuffer 对象并不确保成功。
2. 返回响应对象。

一个 blob 响应是执行下面步骤的结果：
1. 设置响应对象为一个新的 Blob 对象表示接收到的字节，并设置 type 为最终的 MIME 类型。
2. 返回响应对象。

一个 document 响应是执行下面步骤的结果：
1. 如果响应体是 null，则返回 null。
2. 如果最终 MIME 类型不是一个 HTML MIME 类型或则会 XML MIME 类型，那么返回 null。
3. 如果 responseType 是一个空的字符串，并且最终 MIME 类型是一个 HTML MIME 类型，那么设置为 null。
> 注意：responseType 为“document”是受限制的，为了防止破坏遗留内容。
4. 如果最终 MIME 类型是一个 HTML MIME 类型，那么：
  1. 让 charset 为最终编码。
  2. 如果 charset 是 null，向前扫描收到的字节的第一个 1024 字节，如果没有不成功的停止，则让 charset 为返回的值。
  3. 如果 charset 是 null，则设置 charset 为 UTF-8。
  4. 让 document 为一个 document，表示的是将接收到的字节禁止脚本执行，使用一个已知的编码 charset 使用 HTML 标准第四章节的规则转化的结果。
  5. 标记 document 为一个 HTML document。
5. 否则，让 document 为 document，表示在接收到的比特上禁止 XML 脚本并执行 XML 转化的结果，如果这失败了（不支持的字符集编码，不友好的命名空间错误），那么就返回 null。
> 注意：引用的资源将不会被加载，也不会应用任何关联的 XSLT。
6. 如果 charset 为 null，则设置 charset 为 UTF-8。
7. 设置 document 的编码为 cahrset。
8. 设置 document 的内容类型为最终 MIME 类型。
9. 设置 document 的 URL 为响应的 url。
10. 设置 docuemnt 的源为上下文对象关联的设置对象的源。
11. 设置响应对象为 document 并返回它。

一个 JSON 响应是执行下面步骤的结果：
1. 如果响应体为空，则返回 null。
2. 让 jsonObject 为在接收到的字节上运行从字节转化为 JSON 的结果。如果抛出一个异常，则返回 null。
3. 设置响应对象为 jsonObject 并返回它。

一个 text 响应是执行下面步骤的结果：
1. 如果响应体为空，则返回空字符串。
2. 让 charset 为最终编码。
3. 如果 responseType 是空字符串，charset 为 null，并且最终 MIME 类型是一个 XML MIME 类型，则使用 XML 规格的规则集去决定编码。让 charset 为确定的编码。
> 注意：responseType 为空字符串是受限制的，为了保证没有遗留的 responseTYpe 值“text”简单。
4. 如果 charset 为 null，则设置 charset 为 UTF-8。
5. 返回在接收到的字节上使用回滚编码字符集运行解码的结果。
> 注意：强烈推荐作者使用 UTF-8 编码他们的资源。

### 4.6.7 overrideMimeType() 方法

对于 web 开发者（非规范）
```
  client . overrideMimeType(mine)
    表现的像响应的`Content-Type`头部为 mime。（它没有真的改变头部）
    如果状态是 loading 或者 done，则抛出一个“InvalidStateError”的 DOMException。
```
overrideMimeType(mime)方法调用的时候，必须执行以下步骤：
1. 如果状态是 loading 或者 done，则“InvalidStateError”的 DOMException。
2. 设置覆盖 MIME 类型为 mime 转化的结果。
3. 如果覆盖 MIME 类型是 failure，则设置覆盖 MIME 类型为 applocation/octet-stream。

### 4.6.8 responseType 属性
对于 web 开发者
```
  client . responseType [ = value ]
    返回响应类型
    可以设置去改变响应的类型，值是：空字符串（默认），“arraybuffer”，“blob”，“document”，“json”，和“text”。
    当设置的时候：设置为“document”将会被忽略，如果当前的全局对象不是一个 Window 对象。
    当设置的时候：如果状态是 loading 或者 done，则“InvalidStateError”的 DOMException。
    当设置的时候：如果同步标记被设置并且当前全局对象是一个 Window 对象，则抛出“InvalidStateError”的 DOMException。
```
responseType 属性必须返回它的值。初始化它的值必须是一个空字符串。

设置 responseType 属性必须执行下面的步骤：
1. 如果当前全局对象不是一个 Window 对象，并且给定的值是“document”，终止这些步骤。
2. 如果状态是 loading 或者 done，则抛出“InvalidStateError”的 DOMException。
3. 如果当前的全局对象是一个 Window 对象并且同步标记被设置，则抛出“InvalidStateError”的 DOMException。
4. 设置 responseType 属性的值为给定的值。

### 4.6.9 response 属性
对于 web 开发者（非规范）
```
  client . response
    返回响应体
```
response 属性必须返回执行下面步骤的结果：
如果 responseType 是空字符串或者“text”
  1. 如果状态不是 loading 或者 done，则返回空字符串
  2. 返回 text 响应。
否则
  1. 如果状态不是 done，则返回 null。
  2. 如果响应对象是 failure，则返回 null。
  3. 如果响应对象是非 null 的，则返回它
  4. 如果 responseType 是“arraybuffer”
        返回 atraybuffer 响应。
     如果 responseType 是“blob”
        返回 blob 响应。
     如果 responseType 是“document”
        返回 document 响应。
     如果 responseType 是“json”
        返回 JSON 响应。 


### 4.6.10 responseText 属性
对于 web 开发者（非规范）
```
  client . responseText
    返回 text response
    如果 responseType 不是空字符串或者“text”，则抛出一个“InvalidStateError”的 DOMException。
```
responseType 必须返回执行下面步骤的结果：
1. 如果 responseType 不是空字符串或者“text”，则抛出一个“InvalidStateError”的 DOMException。
2. 如果状态不是 loading 或者 done，则返回空字符串。
3. 返回 text 响应。

### 4.6.11 responseXML 属性
对于 web 开发者（非规范）
```
  client . responseXML
    返回 document 响应
    如果 responseType 不是一个空字符串或者“document”，则抛出一个“InvalidStateError”的 DOMException。
```
responseXML 属性必须返回执行以下步骤的结果：
1. 如果 responseType 不是一个空字符串或者“document”，则抛出一个“InvalidStateError”的 DOMException。
2. 如果状态不是 done，则返回 null。
3. 断言：响应对象不是 failure。
4. 如果响应对象不是 null，则返回它。
5. 返回 document 响应。

### 4.7 事件概述
这个章节是非规范的。

下面的事件在 XMLHttpResponse 或者 XMLHttpRequestupload 对象：

| 事件名 | 接口 | 派发时机 |
| - | - | - |
| readystatechange | Event | readyState 属性改变值，期待当他变成 UNSENT 的时候 |
| loadstart | ProgressEvent | 获取初始化 |
| progress | ProgressEvent | 传输数据 |
| abort | ProgressEvent | 当获取被中止。比如，通过调用 abort() 方法 |
| error | ProgressEvent | 获取失败 |
| load | ProgressEvent | 获取成功 |
| timeout | ProgressEvent | 在获取完成之前，作者指定超时事件已经过去 |
| loadend | ProgressEvent | 请求完成（失败或者成功） |

### 4.8 特性集成策略
这个规格定义了一个策略控制特性，通过字符串“sync-xhr”标识。它默认允许列表是 *。

### 5 接口 FormData

```
typedef (File or USVString) FormDataEntityValue;

[Constructor(optional HTMLFormElement form), Exposed=(Window,Worker)]
interface FormData {
  void append(USVString name, USVString value);
  void append(USVString name, Blob blobValue, optional USVString filename);
  void delete(USVString name);
  FormDataEntryValue? get(USVString name);
  sequence<FormDataEntryValue> getAll(USVString name);
  boolean has(USVString name);
  void set(USVString name, USVString value);
  void set(USVString name, Blob blobValue, optional USVString filename);
  iterable<USVString, FormDataEntryValue>;
}
```

每一个 FormData 对象都有一个关联的 entry list（一个 list 或者 entries）。它初始化为一个空的列表。

一个实体由名字和值组成。

为了和其他算法交互的目的，一个 entry 的文件名在值不是一个 File 对象的时候为空字符串，否则它的文件名是 entry 的 value 的 name 属性。

为了使用 name，value，和可选的 filename 创建一个 entry，执行下面的步骤：

1. 让 entry 为一个新的 entry。
2. 设置 entry 的 name 为 name。
3. 如果值是 Blob 对象而不是 File 对象，则设置 vakue 为新的 File 对象，表示相同的字节，但是 name 属性值为 blob。
4. 如果 value 是（现在）一个 File 对象，并且给出了 filename，则设置 value 为 新的 File 对象，表示相同的字节，但是 name 属性值为 filename。
5. 设置 entry 的值为 value。
6. 返回 entry。

FormData(form) 构造器必须执行下面的步骤：
1. 让 fd 为新的 FormData 对象。
2. 如果给定 form，则：
    1. 让 list 为为 form 构造 entry 列表的结果。
    2. 如果 list 是 null，则抛出一个“invalidStateError”的 DomException。
    3. 设置 fd 的 entry 列表为 list。
3. 返回 fd。

appen(name, value) 和 append(name, blobValue, filename) 方法调用的时候，必须执行下面的步骤：
1. 如果给定value，则让 value 为 value，否则让 value 为 blobValue。
2. 让 entry 为使用 name，value，filename 创建一个实体的结果。
3. 拼接 entry 到上下文对象的 entry list。

> 这里有一个参数的名字是 value，同时又是 blobValue 的原因是因为用来编写XMLHttpRequest 编辑软件的的限制。

delete(name) 方法调用的时候，必须从上下文对象的 entry list 移除所有名字为 name 的 entries。

get(name) 方法调用的时候，必须返回上下文对象的 entry list 第一个名字为 name 的entry 的值，否则返回 null。

getAll(name) 方法调用的时候，必须返回上下文对象的 entry list 所有名字为 name 的 entries 的值，否则返回一个空的 list。

has(name) 方法调用的时候，如果上下文对象的 entry list 中有一个 entry 名字为 name，则必须返回 true，否则返回 false。

set(name, value) 和 set(name, blobValue, filename) 方法调用的时候，必须执行下面的步骤：

1. 如果 value 有值，让 value 为 value，否则，让 value 为 blobValue。
2. 让 entry 为用 name，value，filename 创建一个 entry的结果。
3. 如果上下文对象的 entry list 中有名字为 name 的 entries，则使用 entry 替换第一个 entry，并移除其他。
4. 否则，拼接 entry 到上下文对象的 entry list。

> 注意：这里有一个参数的名字是 value，同时又是 blobValue 的原因是因为用来编写XMLHttpRequest 编辑软件的的限制。

用来迭代的值对是上下文对象的 entry list 的 entries，key 作为 name，value 作为 vaue。

### 6 接口 ProgressEvent
```
[Constructor(DOMString type, optional ProgressEventinit eventInitDict), Exposed=(Window,DedicatedWorker,SharedWorcker)]
interface ProgressEvent : Event {
  readonly attribute boolean lenthCompotable;
  readonly attribute unsigned long long loaded;
  readonly attribute unsigned long long total;
};

dictionary ProgressEventInit : EventInit {
  boolean lengthCompitable = false;
  unsigned long long loaded = 0;
  unsigned long long total = 0;
};
```
事件使用 ProgressEvent 接口知识一些类型的进程。

lengthComputable，loaded，和 total 属性必须返回他们初始化的时候的值。

### 6.1 使用 Progress 接口触发事件

为了在 target 上触发名为 e 的处理事件，给定 transmitted 和 length，意味着在 target 触发一个名为 e 的事件，使用 ProgressEvent，和 loaded 属性初始化 transmitted，并且如果 length 不是 0，lengthComputable 属性初始化为 true，并且 total 属性初始化为 length。

### 6.2 使用 ProgressEvent 接口推荐的事件名称。
这个章节是非规范的。

使用 ProgressEvent 接口推荐的事件的 type 属性的值在下表描述。规格编辑者可以根据他们的场景自由的调整细节，尽管强烈鼓励和 WHATWG 社区讨论他们的使用，以确保熟悉这个课题的人提出建议。

| type 属性值 | 描述 | 次数 | 时机 |
| - | - | - | - |
| loadstart | 进度开始 | 一次 | 第一 |
| progress | 正在进行 | 一次或者多次 | loadstart 被派发之后 | 
| error | 进度失败 | 0 或者 1 次（相互排斥） | 在 progress 被派发之后 | 
| abort | 进度被终止 |  |  | 
| timeout | 进度因为预定的时间过期导致终止 |  |  | 
| load | 进度成功 |  |  | 
| loadend | 进度停止 | 一次 | 在 error，abort，timeout，或者 load 被派发之后 | 

error，abort，timeout，和 load 事件类型是相互排斥的。

几乎所有的 web 平台 error，abort，timeout，和 load 事件类型都有他们的 bubbles 和 cancelable 属性初始化为 false，所以推荐所有使用 ProgressEvent 接口的事件都这么做，以保持一致性。

### 6.3 安全考虑

对于跨域请求有一些选择，比如，定义在 Fetch 标准的 CORS 协议需要在使用 ProgressEvent 接口的事件被派发之前使用，因为信息（比如，大小）将会显示无法获取。

### 6.4 栗子

在这个栗子中的 XMLHttpRequest，结合了在前面章节定义的钙奶呢，并且 HTML progress 元素被用来显示获取一个资源的进度。
```
<!DOCTYPE html>
<title>Waiting for Magical Unicorns</title>
<progress id=p></progress>
<script>
  var progressBar = document.getElementById("p"),
      client = new XMLHttpRequest()
  client.open("GET", "magical-unicorns")
  client.onprogress = function(pe) {
    if(pe.lengthComputable) {
      progressBar.max = pe.total
      progressBar.value = pe.loaded
    }
  }
  client.onloadend = function(pe) {
    progressBar.value = pe.loaded
  }
  client.send()
</script>
```
完整的可工作的代码当然会更加的精细，并处理更多常见，比如网络错误或者用户终止请求。
