[已校对]
# 转化器

TypeScript 转化器的源代码完全位于`parse.ts`。扫描器在内部被`Parser`控制用于转化源代码到一个 AST。这是一个期待的结果的一个回顾。
```ts
SourceCode ~~ scanner ~~> Token Stream ~~ parser ~~> AST
```

转化器实现为一个单例（和`scanner`的原因类似，不需要重新创建他如果我们可以重新初始化它）。它其实实现为`namespace Parser`，包含 Parser 的状态变量，和单例`scanner`类似。正如前面提到的，它包含一个`const scanner`。转化器函数管理这个扫描器。

### program 的使用

转化器直接被 Program 驱动（他实际间接的被我们前面提到的`CompilerHost`调用）。基本上这是简化的调用栈。

`ParseSourceFile`不仅仅启动转化器的状态，也启动`scanner`的状态，通过调用`initializeState`。然后它开始使用`parseSourceFileWorker`转化源代码。

### 简单使用

在我们深入转化器内部之前，这是一个例子代码，使用 TypeScript 的转化器去获取一个源文件的 AST（使用`ts.createSourceFile`），然后打印它。

`code/compiler/parser/runParser.ts`

```ts
import * as ts from "ntypescript";

function printAllChildren(node: ts.Node, depth = 0) {
    console.log(new Array(depth + 1).join('----'), ts.formatSyntaxKind(node.kind), node.pos, node.end);
    depth++;
    node.getChildren().forEach(c=> printAllChildren(c, depth));
}

var sourceCode = `
var foo = 123;
`.trim();

var sourceFile = ts.createSourceFile('foo.ts', sourceCode, ts.ScriptTarget.ES5, true);
printAllChildren(sourceFile);
```


这将会输出如下的东西：
```ts
SourceFile 0 14
---- SyntaxList 0 14
-------- VariableStatement 0 14
------------ VariableDeclarationList 0 13
---------------- VarKeyword 0 3
---------------- SyntaxList 3 13
-------------------- VariableDeclaration 3 13
------------------------ Identifier 3 7
------------------------ FirstAssignment 7 9
------------------------ FirstLiteralToken 9 13
------------ SemicolonToken 13 14
---- EndOfFileToken 14 14
```

这看起来像一棵树（非常右边），如果你倾斜你的头到左边。