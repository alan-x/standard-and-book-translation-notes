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



