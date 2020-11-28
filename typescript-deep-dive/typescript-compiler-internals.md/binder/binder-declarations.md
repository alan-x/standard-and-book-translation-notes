[已校对]
# 绑定器声明

绑定一个`node`和一个`symbol`通过一些函数执行。一个函数用于绑定`SourceFile`节点到源文件符号（如果是一个外部模块）的是`addDeclarationToSymbol`函数。

注意：一个外部模块源文件的`Symbol`设置为`flags : SymbolFlags.ValueModule`和`name: '"' + removeFileExtension(file.fileName) + '"')`。

```ts
function addDeclarationToSymbol(symbol: Symbol, node: Declaration, symbolFlags: SymbolFlags) {
    symbol.flags |= symbolFlags;

    node.symbol = symbol;

    if (!symbol.declarations) {
        symbol.declarations = [];
    }
    symbol.declarations.push(node);

    if (symbolFlags & SymbolFlags.HasExports && !symbol.exports) {
        symbol.exports = {};
    }

    if (symbolFlags & SymbolFlags.HasMembers && !symbol.members) {
        symbol.members = {};
    }

    if (symbolFlags & SymbolFlags.Value && !symbol.valueDeclaration) {
        symbol.valueDeclaration = node;
    }
}
```

重要的链接部分：

- 从 AST 节点（`node.symbol`）创建一个链接到符号
- 添加节点作为符号的声明之一（`symbol.declarations`）

### 声明

声明只是一个`node`，和一个可选的名字，在`types.ts`
```ts
interface Declaration extends Node {
    _declarationBrand: any;
    name?: DeclarationName;
}
```