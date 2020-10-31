# AST

### Node

抽象语法树最基本的构成块。通常一个`Node`在语言语法中表示非终结符；然而，一些终结符是保存在树上的，比如标示符和字面量。

两个关键动机构成了一个 AST 节点的文档。节点的`SyntaxKing`表示在 AST 中的类型，和他的`interface`，当它初始化到 AST 的节点的 API

这里是一些关键的`interface Node`成员：
- `TextRange`：表示节点在源文件的`start`和`end`位置的成员
- `parent?: Node`：AST 中节点的父节点

`Node`还有其他的标示符和修饰器。你可以通过在源文件中搜索`interface Node`查找，但是有一个对于节点遍历是必须提到的。

### SourceFile

- `SyntaxKind.SourceFile`
- `interface SourceFile`

每一个`SourceFile`都是顶级 AST 节点，包含在`Program`