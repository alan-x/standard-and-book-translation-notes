### JavaScript 1.0 and 1.1

Netscape Communitations Corporation 和 Sun Microsystems 在 1995 年 9 月 4 宣布 JavaScript，在一个联合发行中[Netscape 和 Sun 1995，附录 F]。联合发行中描述 JavaScript 为“一个对象脚本语言”，将用于编写脚本，动态“修改 Java 对象的属性和行为”。它将作为“Java 简单在线应用开发的补充”。这些公司尝试在 Java 和 JavaScript 之间建立强大的品牌关联，尽管他们的技术设计知识表面相似。名字的相似性和他暗示的相近的语言关系是持续混淆的根源。

// 这里缺少一张图片

图片 2. Mocha Console。Brendan Eich 关于“Mocha Console”的初始化 demo，运行在一个 SGI Unix 工作站的 pre-alpha 版本的 Netscape 2。相同的 Mocha Console 包装在，除了名字基本没变，作为 Netscape 2 产品发行的一部分。这是运行在 Windows 95 的 Netscape 2.02 的屏幕截图。Mocha 控制台通过输入 mocha: 到浏览器地址栏激活 --因为 Netscape2 的产品就是这样改变到 javascript: 但是 mocha: 依旧工作。激活控制台导致一个双 frame 页面在浏览器打开。Mocha 表达式输入到下面的 frame，执行效果在上面 frame 的上下文。这个例子展示了内置的 alert 函数被调用先是一个弹窗，展示表达式计算的结果。原始的 demo 版本将会展示“Mocha Alert”在弹窗，而不是“JavaScript Alert”

JavaScript，在 1995 年 9 月作为 Netscape Navigator 2.0 第一次 beta 发行[Netscape 1995b]的一部分以“LiveScript”的名字第一次暴露在公众面前。这个发行后面还有 4 个 beta 发行，直到 1996 年 5 月的 Naviagor 2.0 产品发行，支持 JavaScript 1.0。Netscape Enterprise Server 2.0 也在 March[Netscape 1996f] 中打包了它，并在他的 LiveWire 服务端脚本组件中和 JavaScript 1.0 交互。

JavaScript 只是 Netscape Navigator 相对次要的功能。因此，他的功能受限于 Navigator 2.0 计划，该计划要求在 1995 年冻结功能。JavaScript 1.0 特性集基本上是 8 月实现的 Mocha 的功能。相对于预想的语言设计，特性集是不健全的，尽管 Eich 从 Mocha 实现后到 Navigator 2.0 发行过程中持续修复问题，但是一系列的问题和边缘行为出现了。1.0 发行前的简短访问[Shah 1996]中，Brendan Eich 回应了 JavaScript 的官方定位是 Java 的附属，是初始发行的匆忙实现：

BE[Brendan Eich]: 我希望它[JavaScript]将会被其他供应商实现，基于我和 Bill Joy 正在编写的规范。我希望看到他依旧小，但是开始变得在网页上无处不在，作为将 HTML 元素和行为使用 Java applet 和其他组件粘合在一起的最好方式。
BE: ...据我所知，最常见的使用场景是让页面更智能，更活跃--比如，让一个链接根据一天的事件加载不同的 URL。
...

BE: 隧道的尽头有曙光，尽管因为 JavaScript 过于单人秀，[Netscape] 2.0 将会包含一系列烦人的错误，我希望是所有的大的问题都有解决方案，我已经花了大量的事件和开发者去寻找 bug 和解决方案。

我正在跟随 2.1，通过寻找问题，添加特性，并尝试去让 JavaScript 在我们的平台保持一致。我不知道 2.1 啥时候发布，但是我相信一定在下一个秋天之前--我们行动非常迅速。

JavaScript 1.0[Netscape 1996d] 是一个简单动态类型语言，支持数字，字符串和布尔值；一级函数；和，一个对象数据类型。语法上，JavaScript 类似 Java，属于 C 家族，有 C 的控制流语句，表达式语句博涵 C 的大部分数字操作符。JavaScript 有一个小的内置函数库。JavaScript 1.0 源代码通常直接嵌入 HTML 文件，但是内置的库包含一个 eval 函数，可以转化和执行编码为 JavaScript 字符串值的 JavaScript 源代码。JavaScript 1.0 是非常精简的语言。图 3 是缺少的一些特性的总结，很多的缺少可能会让现代 JavaScript 程序员非常吃惊。

在 1996 年早期，在“Atlas”[Netscape 1996g]工作的早期，代码名字表示什么东西可以打包到 1996 年 8 月的 Netscape Navigator 3。0 中。Brendan Eich 能够重新恢复在 1995 年 2.0 特性冻结之后不完善或者缺失的特性。只有在 nevigator 3.0 的发行中，JavaScript 1.1[Netscape 1996a,e] 的初始化定义和开发才完成。下面的章节展示了 JavaScript 1.0/1.1 语言的设计概览。

### 3.1 JavaScript 语法

JavaScript 1.0 的语法直接来自 C 编程语言[ANSI X3 1989]，一些 AWK[Aho et al, 1998]来自灵感。一个脚本是一系列的语句和声明。不像 C，JavaScript 语句不限制出现在函数体内。在 JavaScript 1.0，脚本源代码被嵌入 HTML 文档，通过一个 <script></script> 标签包裹。

JavaScript 1.0 中受 C 启发的语句是表达式语句；if 条件语句；for 和 while 迭代语句；语句块使用 {} - 限制一系列的语句使用就像单一的语句。if，for 和 while 语句都是 compound 语句。JavaScript 1.0 不包含 C 的 do-while 语句，switch 语句，语句标签，或者 goto 语句。

对于 C 语句基础套装，JavaScript 1.0 添加两个 compund 语句，用于访问他的对象数据类型属性。迭代一个对象的属性  for-in 语句启发的 AWK。在一个 with 语句体内部，指定的对象的属性可以像被声明的变量一样访问。因为属性可以动态添加（在之后的语言版本删除），可见的变量绑定可能在 with 语句体内指定过程可能改变。

JavaScript 声明不遵循 C 或者 Java 的风格。JavaScript 是动态类型；甚至，他们有语言级别类型名字去支持语法前缀去标示声明。相反，JavaScript 声明是关键字前缀。JavaScript 1.0 有两种形式的声明：function 声明和 var 声明。function 声明语法直接来自 AWK。一个 function 声明定义了名字，正式参数，和一个单独可调用的函数的语句体。一个 var 声明引入一个或者多个变量绑定，和可选的变量赋值。所有的 var 声明都被认为是语句，可能吹安在任何语句上下文，包括块级语句。在 JavaScript 1.0/1.1 中，function 声明只能出现在脚本的顶级，不能在 function 声明中嵌套。一个 var 声明可能出现在函数体，和变量定义，通过类似声明，作为函数的本地声明变量。

不像 C，JavaScript 1.0 语句块不产生声明上下文。在一个函数体内，在一个块内的 var 声明在整个函数体内是本地可见的。函数外的一个块内的 var 声明有全局作用域。赋值一个不在函数作用域或者没有 var 声明的变量名意味着创建一个这个名字的全局变量。这个行为被证明是一个重要的错误来源，因为错误的输入一个已声明的变量名静默创建了一个新的错误输入的变量。

和传统 C 语法一个主要的不同是 JavaScript 对待语句结尾封号的方式。C 将封号认为是语句的终结符，JavaScript 允许语句终结封号缺省，当他们是一行的最后一个字符的时候。这个明确的规则没有包含在 JavaScript 1.0 文档中。Netscape 2.0 手册没有展示封号，当藐视多种 JavaScript 语句形式的时候。他简单说：“一个单独的语句可能分割在多行。多行语句可能出现在单行，如果每一个语句使用封号[Netscape 1996d]分离”。一个封号自由的编码风格在手册的 JavaScript 代码例子中经常如下出现：
```js
var a, x, y
var r=10
with ( Math ) {
    a = PI * r * r
    x = r * cos(PI)
    y = r * sin(PI /2)
}
```

编写 JavaScript 代码不实用封号的能力被成为自动封号插入（ASI）。ASI 依旧控制在 JavaScript 程序员手中。一个重要的事实是程序员依旧更喜欢封号自由的编写风格，其他人则希望没有人使用 ASI。


### 3.2 数据类型和表达式

JavaScript 1/0/1.1 是一个动态类型语言，有 5 个基础数据类型：number，string，Boolean，object，和 function。所谓“动态类型”，我们意思是运行时类型信息和每一个值关联，而不是类似变量的值容器。运行时类型检测确保操作只能应用在支持这个操作的数据值。

布尔，字符串，和数字是不可变的值。Boolean 类型有两个值，名为  true 和 false。字符串值由不可变的 8 位字符码构成。不支持 Unicode。数字类型犹豫 IEEE 754[IEEE 2008]双精度 64 位浮点数的所有可能值构成，还有一个异常的单一 NaN 值被暴露。一些操作对数字有特殊对待，表示无符号 32 位数字和有符号 32位二进制数字。Mocha 尝试使用一个替代的表示这些数字值，但是官方只有一个单独的数字数据类型。

JavaScript 1.0  有两个特定的值去表示有用的数据值的缺省。为初始化变量被设置为特殊的值 undefined。这个值在程序尝试访问一个对象不存在的属性的时候也会返回。在 JavaScript 1.0，值 undefined 可能通过声明和访问一个未初始化的变量得到。null 值倾向于在期待一个对象值的表示上下文“没有对象”。他是模仿 Java 的 null 值，它促进了 JavaScript 和 Java 实现的对象的集成。纵观它的整个历史，这两个相似但是明显不同的值在 JavaScript 程序员中导致了混淆，他们不确定什么使用他们中的一个而不是另一个。

JavaScript 1.0 的表达式语句从 C 中复制过来，还有常用的操作符集合和优先级规则。主要的缺省是 C 的指针和类型相关的操作符，和一元 + 操作符。二进制 + 操作符被重写为执行数字加法和字符串连接。位移和按位与逻辑操作符基于位级别 32位二进制的补位操作。如果需要，操作数被截断位整数并取模减少到 32 位值。>> 操作符执行 32 位数字值的右移。JavaScript 添加 >>> 操作符，来自 Java，执行一个无符号右移。

JavaScript 1.1 添加 delete，typeof，和 void 操作符。在 JavaScript 1.1，delete 操作符简单设置它的值或者对象属性操作数为值 null。typeof 操作符返回一个字符串表示操作数的原始类型。它可能的字符串值为“undefined”，“object”，“function”，“boolean”，“string”，“number”，或者一个实现定义的字符串值，标示一个类型的宿主定义的对象。意外的，typeof null 返回字符串“object”，而不是“null”。这和 Java 保持一致，所有值都是对象，“null”是“没有对象”的对象。然而，Java 缺少一个相同的 typeof 操作符，并使用 null 作为未初始化值的默认值。Brendan Eich 的回忆是 typeof null 的值是原始 Mocha 实现的一个抽象泄漏的结果。null 的运行时值使用和对象相同的内部标签值编码，因此 typeof 操作值实现返回的“object”，不需要任何额外的特殊场景逻辑。这个选择被证明是一个巨大的烦恼，JavaScript 程序要通常去测试一个值是否是一个对象，在尝试使用他的值作为属性访问的基础。但是 typeof 一个值的测试是一个“object”是一个不足的保证一个属性访问，因为尝试放 null 的一个属性长生一个运行时错误。

void 操作符简单执行它的操作数，然后返回 undefined。访问 undefined 的一个典型方式是 void 0。void 操作符引入是为了定义 HTML 超链接在点击的时候执行 JavaScript 代码，比如：
```
<a href="javascript:void usefulFunction()">Click to do something useful</a>
```

href 属性的值应该是一个 URL，javascript: 是一个特殊的 URL 协议，浏览器可以识别他。它意味着执行后面跟随的 JavaScript 代码并使用他的结果，转化为一个字符串，就好像使用一个常规 href URL 响应的文档。<a> 元素将会尝试处理响应文档，除非他是 undefined。通常一个 Web 开发者只想要 JavaScript 表达式在连接被点击的时候执行。在表达式前面添加一个 void 允许它使用这种方式避免<a>元素后续的处理。

C 和 JavaScript 表达式最大的差别是 JavaScript 操作符自动强制他们的操作数的数据类型转化为操作符的域。JavaScript 1.1 添加了一个可配置的机制强制任意的对象转化为数字或者字符串。图 4 总结了 JavaScript 1.1 强制规则。

### 3.3 对象



### 3.4 函数对象

### 3.5 内置库

### 3.6 执行模型

### 3.7 怪事和错误

