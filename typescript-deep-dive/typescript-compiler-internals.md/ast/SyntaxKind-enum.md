[已校对]
# 提示：SymtaxKind 枚举

`SyntaxKind`定义为一个`const enum`，这是一个例子：
```ts
export const enum SyntaxKind {
    Unknown,
    EndOfFileToken,
    SingleLineCommentTrivia,
    // ... LOTS more
```

这是一个`const enum`（[我们之前提到](https://basarat.gitbook.io/typescript/type-system/enums)的概念），因此它被内联（比如，`ts.SyntaxKind.EndOfFileToken` 变成 1），并且没有解索引消耗，当和 AST 一起使用的时候。然而，编译器使用`--preserveConstEnums`编译器选项，因此枚举依旧在运行时可用。因此在 JavaScript，你可以使用`ts.SyntaxKind.EndOfFileToken`，如果你想要。此外，聂可以转化这些枚举成员去显示字符串，使用下面的函数：
```ts
export function syntaxKindToName(kind: ts.SyntaxKind) {
    return (<any>ts).SyntaxKind[kind];
}
```