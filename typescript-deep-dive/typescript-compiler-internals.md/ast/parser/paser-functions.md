[已校对]
# 转化器函数

### 转化器函数

正如提到的，`parseSourceFile`设置初始化状态，并传递工作到`parseSourceFileWorker`函数。

`parseSourceFileWorker`

开始创建一个`SourceFile` AST 节点。然后从`parseStatements`函数开始转化源代码。一旦返回，它完成`SourceFile`，携带额外的信息，比如它的`nodeCount`，`identifierCount`之类的。

`parseStatements`

其中一个最重要的`parseFoo`风格函数（我们将在下面覆盖的概念）。它和当前从扫描器中返回的`token`交换。比如，如果当前的词素是一个`SemicolonToken`，它将会调用`SemicolonToken`去创建一个空语句的 AST 节点。

### 节点创建

转化器有一系列的`parserFoo`函数，它的函数题创建`Foo`节点。这些通常在（从其他转化器函数）一个`Foo`节点被期待的时候调用。这个过程的一个典型例子是`parseEmptyStatement()`函数，用于转化一个空的语句，比如`;;;;;;`。这是整个函数
```ts
function parseEmptyStatement(): Statement {
    let node = <Statement>createNode(SyntaxKind.EmptyStatement);
    parseExpected(SyntaxKind.SemicolonToken);
    return finishNode(node);
}
```

它展示了三个标准函数`createNode`，`parseExpected`，和`finishNode`。

`createNode`

转化器的`createNode`函数`function createNode(kind: SyntaxKind, pos?: number): Node`用于创建一个节点，设置它的`SyntaxKind`作为传入，并设置出事位置，如果传入（或者使用当前的扫描器张泰的位置）。

`parseExpected`

转化器的`parseExpected`函数`function parseExpected(kind: SyntaxKind, diagnosticMessage?: DiagnosticMessage): boolean`将会检查当前的词素是否匹配期待的`SyntaxKind`。如果不，他捡回报告传入的`diagnosticMessage`，或者创建一个通用的`foo expected`形式。它内部使用`foo expected`函数（使用扫描位置）去提供一个好的错误报告。

`finishNode`

转化器的`foo expected`函数`function finishNode<T extends Node>(node: T, end?: number): T`设置节点的`end`位置和额外的有用的东西，比如`parserContextFlags`，它将会在下面转化，如果在转化这个节点之前有一些错误也是一样（如果有，我们不能宠幸使用这个 AST 节点在增量转化）。