# RFC 2234: Augmented BNF for Syntax Specifications: ABNF

> 扩展的巴克斯范式

### 1. 介绍

互联网技术规则通常需要定义一个格式化语法，自由的使用任何作者觉得有用的符号。随着时间的流逝，一个修改版的巴克斯范式（BNF）在许多互联网规格中流行起来，成为扩展的 BNF（ABNF）。它权衡了紧凑和简单，还有合理的表达能力。在 Arpanet 早期，每一个规格都有它自己定义的 ABNF，这包括了邮件规格，RFC733 和之后 RFC822 成为了定义 ABNF 的常见引用。现在这个文档将这些定义分离出来，允许有选择的引用。可预见的，这也提供了一些修改和增强。

BNF 和 ABNF 之间的区别涉及规则命名，重复，替换，顺序无关，和值区间。附录A（Core）提供了规则定义，并为一些互常见联网规格编码核心词法分析器。它是为了方便的和定义在这个文档主体中的元语言分离而提供的，并且是正式的分离。

### 2. 规则定义

2.1 规则命名

规则的名字就是名字本身；它是由一些列字符，由字母字符开始，后面跟随一个字母、数字、连字符（破折号）的混合体。

> 注意：规则名字是大小写不敏感的

命名&lt;rulename&gt;、&lt;Rulename&gt;、&lt;RULENAME&gt; 和 &lt;rUlENamE&gt; 都引用相同的规则。

不像原始的 BNF，角括号（"&lt;"，"&gt;"）不是必须的。

但是，如果他们的存在可以促进分辨出规则名称的使用的时候，就可以在规则名称周围包裹角括号。这通常限制于在自由格式中引用规则名，或者区分结合到不使用空格分离的部分规则，比如下面讨论到的重复。

2.2 规则格式

一条规则定义为一下序列：

```
name = elements crlf
```

&lt;name&gt; 是规则的名字，&lt;elements&gt; 是一个或多个规则名字，或者终结说明，&lt;crlf&gt;是行终结指示器，回车后面跟随一个换行。等号分离了名字和规则的定义。elemens 构成了一些列一个或多个规则名字和/或值定义，通过一系列定义在这个文档的操作符结合起来，比如替换和重复。

从视觉来说，规则定义是左对齐的。当一条规则需要多行的时候，接下来的行将会缩进。左对齐和缩进是相对于 ABNF 规则的第一行来说的，不需要匹配文档的左外边距。

2.3 终结值

规则解析为一个终结值字符串，有时候叫做字符。在 ABNF 中，一个字符不过时一个非负整数。在确定的上下文中，一个指定的值到字符集（比如 ASCII）的映射（编码）将会被指定。

```
Terminals are specified by one or more numeric characters with the
```

```
   base interpretation of those characters indicated explicitly.  The
   following bases are currently defined:
```

终结值由一个或者多个数字字符指定，并明确指出这些字符的基本解释。下面定义了几个基础的：

```
b    = binary
d    = decimal
x    = hexadecimal
```

因此：

```
CR   = %d13
CR   = %d0D
```

各表示 \[US-ASCII\] 回车的十进制和十六进制定义。

```
 A concatenated string of such values is specified compactly, using a
```

```
   period (".") to indicate separation of characters within that value.
   Hence:
```

许多这种值的紧凑的联合在一起，值与值之间使用句点（“.”）表示隔开。因此：

```
CRLF    = %d13.10
```

ABNF 允许直接指定文本字符串字面量，用双引号包裹，因此：

```
command    = "command string"
```

文本字符串字面量解释为一系列联合起来可打印的字符。

> 注意：ABNF 字符串是大小写不敏感的，并且这些字符串的字符集是 us-ascii。

因此：

```
rulename    = "abc"
```

和：

```
rulename    = "aBc"
```

将会匹配“abc”、“Abc”、“aBc”、“abC”、“ABc”、“aBC”、“AbC”和“ABC”。

指定一个大小写敏感的规则，要单独的指定字符。

比如：

```
rulename    = %d97 %d98 %d99
```

或者：

```
rulename    = %d97.98.99
```

将只会匹配由小谢字符构成的字符串，abc。

2.4 外部编码

终结值字符的外部表示将会根据存储或传输的限制有所不同。因此，相同的基于 ABNF 的语法有不同的外部编码，比如 7 比特 US-ASCII 环境，二进制八位环境，使用 16 位 Unicode 也是不同的。编码细节超出了 ABNF 的范围，虽然附录 A（核心）提供了 7 比特 US-ASCII 定义，就像大部分互联网环境常见的。

将外部编码从语法中分离是为了可以在不同编码环境使用相同语法。

### 3. 操作符

3.1 连续

一条规则可以定义一个简单的，有序的字符串值 —— 比如，一连串相邻的字符 —— 通过列出一系列的规则名称。比如

```
foo    = %x61
bar    = %x62
mumble = foo bar foo
```

所以，规则 &lt;mumble&gt; 匹配小写字符串“aba”

LINEAR WHITE SPACE：连续是 ABNF 解析模型的核心。相邻字符（值）构成的字符串的解析基于定义在 ABNF 中的规则。对于互联网规格来说，一些历史原因使得线性空白符（空格或者水平制表符）可以自由并隐含的围绕在主结构，比如分隔特殊字符或者原子字符串。

任何允许线性空白符的围绕分隔符或者字符串片段的语法必须明确的指定。通常常在“核心”规则提供这些空白符然后在高级规则中广泛使用是有用的。“核心”规则可能构成一个词法分析器或者指示主要规则集的一部分。

3.2 替换

使用反斜杠（“/”）分隔的元素是可替换的。

因此

```
    foo / bar
```

将会接受 &lt;foo&gt; 或者 &lt;bar&gt;。

> 注意：一个用引号包裹的包含字母字符的字符串是指定可替换字符的特殊形式，解释为一个使用所包含字符并按照指定顺序结合的，但是任意混合大小写的集合的字符串非终结表示。

3.3 递增替换

指定一个可替换的列表的片段有时候是非常方便的。比如，一个初始化的规则匹配一个或者多个替换，接下来的规则定义添加替换的集合。这对于那些非常有用-从同一个父规则集独立出来的定义，比如经常和参数列表出现。ABNF 允许通过这种结构的自增定义：

```
    oldrule    =/ additional-alernatives
```

所以，规则集

```
    ruleset    =  alt1 / alt2
    ruleset    =/ alt3
    ruleset    =/ alt4 . alt5
```

和下面的定义是一致的：

```
    ruleset    = alt1 / alt2 / alt3 /alt4 / alt5
```

3.4 可替换值区间

一个区间的可替换数字值可以定义的更紧凑，使用横杆（“-”）指示可替换的值的区间。因此

```
    DIGIT    = %x30-39
```

和下面相等：

```
    DIGIT    = "0" / "1" / "2" / "3" / "4" / "5" / "6" /
                    "7" / "8" / "9"
```

不能在同一个字符串中指定联合数字值和数字值区间。一个数字值可能使用点来表示联合或者使用横杠来表示定义一个值区间。因此，指定一个可打印的字符，在行序列之间，定义可能是这样的：

```
    char-line = %x0D.0A %x20-7E %x0D.0A
```

3.5 序列组

包裹在圆括号内的的元素被认为是一个元素，其内容是严格有序的，因此：

```
    elem (foo / bar) blat
```

匹配 \(elem foo balt\) 或者 \(elem bat blat\)。

```
    elem foo / bar blat
```

匹配 \(elem foo\) 或 \(bar blat\)。

> 注意：当替换由多个规则名或者字面量构成的时候，强烈建议使用分组声明，而不是依赖于合适的阅读“赤裸”的替换。

因此，建议用以下形式替换以上形式：

```
    (elem foo) / (bar blat)
```

这可以避免偶然的阅读者错误的阐释。

序列分组在自由文字中也用于对元素序列的引用。

3.6 变量重复

元素前面的操作符“\*”用来指示重复。完整的格式是：

```
    <a>*<b>element
```

&lt;a&gt; 和 &lt;b&gt; 都是可选择数字值，表示最少 &lt;a&gt;，最多 &lt;b&gt;次出现该元素。

默认值是 0 和 无限，因此 \*&lt;element&gt; 允许任何数量，包括 0；1\*&lt;element&gt;需要至少一个；3\*3&lt;element&gt;允许精确的 3 个；1\*2&lt;element&gt; 允许一个或者两个。

3.7 指定重复

一个规则的格式：

```
    <n>element
```

和下面相等：

```
    <n>*<n>element
```

也就是，在 &lt;element&gt; 面前有一个精确的 &lt;n&gt;，所以，2DIGIT 是一个 2位的数字， 3ALPHA 是3个字母字符串组成的字符串。

3.8 可选序列

方括弧包裹一个可选的元素序列：

```
    [ foo bar]
```

和下面相等：

```
    *1(foo bar)
```

3.9 注释

在一行的结尾用一个封号开始注释。

这是在规格中并行包含有用笔记最简单的方式。

3.10 操作符优先级

上面描述的所有机制都有如下的优先级，越上面优先级越高，越下面优先级越低：

```
    字符串，命名格式
    注释
    值区间
    重复
    分组，可选
    连接
    替换
```

使用替换操作符，和连接操作符自由混合很容易被迷惑。

再次声明，推荐使用分组操作符可以让连接更明确。

### 4. 使用 ABNF 定义 ABNF

这里的语法使用 附录A（核心）提供的规则。

```
    rulelist      = 1*( rule / (*c-wsp c-nl) )
    rule          = rulename defined-as elements c-nl
                        ; 如果第二行是以空格开始则继续
    rulename      = ALPHA *(ALPHA / DIGIT / "-")
    defined-as    = *c-wsp ("=" / "=/") *c-wsp
                        ; 基本规则定义和自增替换
    elements      = alternation *c-wsp
    c-wsp         = WSP / (c-nl WSP)
    c-nl          = comment / CRLF
                        ; 注释或者新行
    comment       = ";" *(WSP / VCHART) CRLF
    alternation   = concatenation
                    *(*c-wsp "/" *c-wsp concatenation)
    concatenation = repetition *(1*c-wsp repetition)
    repetition    = [repeat] element
    repeat        = 1*DIGIT / (*DIGIT "*" *DIGIT)
    element       = rulename / group / option /
                    char-val / num-val / prose-val
    group         = "(" *c-wsp alternation *c-wsp ")"
    option        = "[" *c-wsp alternation *c-wsp "]"
    char-val      = DQUOTE *(%x20-21/%x23-7E) DQUOTE
                        ; 包裹 SP 和 VCHART 的字符串，不包行 DQUOTE
    num-val       = "%" (bin-val / dec-val /hex-val)
    bin-val       = "b" 1*BIT
                    [ 1*("." 1*BIT) / ["-" 1*BIT] ]
                        ; 一些列连接的比特值或者单个 ONEOF 区间
    dec-val       = "d" 1*DIGIT
                    [ 1*("." 1*HEXDIG) / ("-" 1*DIGIT) ]
    hex-val       = "x" 1*HEXDIG
                    [ 1*("." 1*HEXDIG) / ("-" 1*HEXDIGIT) ]
    prose-val     = "<" *(%x20-3D / %x3F-7E) ">"
                        ; 括号包裹起来的字符串不需要角括号散文描述
                        ; 作为最后的排序手段
```

### 5. 安全考虑

可以确认安全和这个文档无关。

### 6. 附录 A - 核心

确认的基本规则是大写的，比如 SP，HTAB，CRLF、DIGIT、ALPHA、etc。

```
    ALPHA    = %x41-5A / %x61-7A     ; A-Z / a-z
    BIT      = "0" / "1"
    CHAR     = %x01-7F               ; 任意 7 位 US-ASCII 字符串，除了 NUL
    CR       = %x0D                  ; 回车
    CRLF     = CR LF                 ; 互联网标准新行
    CTL      = %x00-1F / %x7F        ; 控制符
    DIGIT    = %x30-39               ; 0-9
    DQUOTE   = %x22                  ; "（双引号）
    HEXDIG   = DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
    HTAB     = %x09                  ; 水平制表符
    LF       = %x0A                  ; 换行
    LWSP     = *(WSP / CRLF WSP)     ; 线性空白符 (过去的新行)
    OCTET    = %x00-FF               ; 8 位数据
    SP       = %x20                  ; 空格
    VCHAR    = %x21-7E               ; 可见（打印）字符
    WSP      = SP / HTAB             ; 空白符
```

6.2 常见编码

表面上，数据相当于“互联网虚拟 ASCII”，即 7 位 US-ASCII 在8位域中，其中最高位（第八）设置为 0，“互联网比特顺序”的字符串的高位比特表现在左边手边，并且最先被发送到互联网。

### 7. 知识

ABNF 最开始定义在 RFC 733，SRI international 的 Ken L.Harrenstien，负责重新编写 BNF 到 扩展 BNF，让它表现的更小更简单的去理解。

最近的项目开始于简单的努力，将 RFC 822 中重复被非 email 规格引用的部分独立出来，即扩展的BNF描述。相对于简单和盲目的将存在的文字转变为分离的文档，工作组选择小心的考虑过去 15 年可用的存在的规格和相关的规格，考虑他们的缺失，从而追逐增强。这让项目比一开始的目的更加远大。有趣的是这并没有让结果和原始相差很大，尽管移除列表符号的决定就像一个惊喜。

这一轮规范是 DRUMS 工作组的一部分，和来自 Jerome Abela , Harald Alvestrand, Robert Elz, Roger Fajman, Aviva Garrett, Tom Harsch, Dan Kohn, Bill McQuillan, Keith Moore, Chris Newman , Pete Resnick 和  Henning Schulzrinne 的重要贡献。

### 8. 引用

\[US-ASCII\] 字符编码集——7位信息交换美国标准编码，ANSI X3.4-1986。

\[RFC733\] Crocker, D., Vittal, J., Pogran, K., and D. Henderson，"ARPA 网络文字信息标准格式"，RFC 733，十一月，1977。

\[RFC822\]  Crocker, D.，“ARPA 网络文字信息标准格式”，STD 11，RFC 822，八月，1982。



