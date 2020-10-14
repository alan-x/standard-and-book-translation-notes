# JSX

TypeScript 支持 JSX 编译和代码分析。如果你对 JSX 不熟悉，这有一个[官方网站]()的摘要：

> JSX 是 ECMAScript 一个 XML 类似的语法，没有任何定义的语义。不是为了被引擎或者浏览器实现。不是一个将 JSX 继承到 ECMAScript 标准的提案。而是用于各种各样的预处理（编译器）去转化这些令牌到标准 ECMAScript。

JSX 背后的动机是允许用户在 JavaScript 去编写 HTML 类似的视图，因此你可以：

- 通过相同代码拥有视图类型检测去检查你的 JavaScript
- 让视图意识到它正在操作的上下文（比如，在传统 MVC 中强化控制-视图）。
- 为 HTML 维护重用 JavaScript 模式，比如`Array.prototype.map`，`?:`，`switch`等，而不是创建一个新的（和可能不好的输入）替代。

这减少了错误的机会并城建你的用户接口的维护性。现在 JSX 的主要消费者是[来自 facebook 的 ReactJS]()。这也是我们将会讨论的 JSX 的使用。