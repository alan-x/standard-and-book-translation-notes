[已校对]

# TypeScript 编译器内部

TypeScript 编译器的源码位于[src/compiler](https://github.com/Microsoft/TypeScript/tree/master/src/compiler)文件夹下。

它分为以下几个关键部分：
- 扫描器（`scanner.ts`）
- 转化器（`parser.ts`）
- 绑定器（`binder.ts`）
- 检查器（`checker.ts`）
- 生成器（`emitter.ts`）
这些在源码中都有他们唯一的文件。这将在这个章节之后解释。

### 语法 vs 语义

语法上正确不代表语义上正确，考虑下面的 TypeScript 代码块，尽管语法有效，但是语义是错的

```ts
var foo: number = "not a number";
```

`Semantic`英语中的意味着“意义”。这个概念在你的脑袋中很有用

### 流程概述

下面是一个 TypeScript 编译器这些关键部分组合的一个快速回顾：
```ts
SourceCode ~~ scanner ~~> Token Stream
```
```ts
Token Stream ~~ parser ~~> AST
```
```ts
AST ~~ binder ~~> Symbols
```
`symbol`是 TypeScript 语义系统的主要构建块。如下所示符号是作为绑定的结果创建的。符号链接 AST 中的声明节点到其他贡献到相同入口的声明。

符号 + AST 是检查器用于验证源代码语义的东西
```ts
AST + Symbols ~~ checker ~~> Type Validation
```

最终当一个 JS 输出被请求：
```ts
AST + Checker ~~ emitter ~~> JS
```

这里是 TypeScript 编译器一些额外的文件，提供工具给这些我们接下来覆盖主要部分。

### 文件：工具

`core.ts`：TypeScript 编译器使用的主要工具。一些重要的点：

- `let objectAllocator: ObjectAllocator`：是一个全局单例变量。它提供了`getNodeConstructor`（节点将会在我们查看`parse`/`AST`的时候覆盖），`getSymbolConstructor`（符号在`binder`覆盖），`getTypeConstructor`（类型在`checker`覆盖），`getSignatureConstructor`（签名是索引，调用和构造签名），

### 文件：关键数据解构

`types.ts`包含关键数据解构和接口，贯穿编译器使用。这是一些简单的关键点：

- `SyntaxKind`

AST 节点类型通过`SyntaxKind`枚举定义

- `TypeChekcer`

TypeChecker 提供的接口

- `CompilerHost`

这是被`Program`用于和`System`交互

- `Node`

一个 AST 节点

### 文件：系统

`system.ts`。TypeScript 编译器和操作系统的所有交互都通过一个`system`接口。接口和他的实现（`WScript`和`Node`）都定义在`system.ts`。你可以把它看作操作环境（OE）

现在你有了大部分文件的概览，我们可以看看`Program`的概念了。