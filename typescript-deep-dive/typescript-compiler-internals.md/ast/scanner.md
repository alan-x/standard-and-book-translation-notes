# 扫描器

TypeScript 扫描器的源代码完全位于`scanner.ts`。扫描在内部被`Parser`控制用于转化源代码到 AST。这是期待的结果：
```ts
SourceCode ~~ scanner ~~> Token Stream ~~ parser ~~> AST
```
### 被转化器使用

`parser.ts`创建了一个单例`scanner`去避免一次又一次的创建扫描器带来的损耗。这个扫描器最初被转化器用在`initializeState`函数。

这是一个转化器中的真实代码的简化版本，你可以运行展示这个概念：
`code/compiler/scanner/runScanner.ts`
```ts
import * as ts from "ntypescript";

// TypeScript has a singleton scanner
const scanner = ts.createScanner(ts.ScriptTarget.Latest, /*skipTrivia*/ true);

// That is initialized using a function `initializeState` similar to
function initializeState(text: string) {
    scanner.setText(text);
    scanner.setOnError((message: ts.DiagnosticMessage, length: number) => {
        console.error(message);
    });
    scanner.setScriptTarget(ts.ScriptTarget.ES5);
    scanner.setLanguageVariant(ts.LanguageVariant.Standard);
}

// Sample usage
initializeState(`
var foo = 123;
`.trim());

// Start the scanning
var token = scanner.scan();
while (token != ts.SyntaxKind.EndOfFileToken) {
    console.log(ts.formatSyntaxKind(token));
    token = scanner.scan();
}
```

这将会输出下面：
```ts
VarKeyword
Identifier
FirstAssignment
FirstLiteralToken
SemicolonToken
```

### 扫描器状态

当你调用`scan`，扫描器更新他的本地状态（扫描的位置，当前的词素细节等）。扫描器提供一连串的工具函数去获取当前扫描器状态。下面的例子，我们创建了一个扫描器，然后使用它去标示词素和他们在代码中的位置：
```ts
// Sample usage
initializeState(`
var foo = 123;
`.trim());

// Start the scanning
var token = scanner.scan();
while (token != ts.SyntaxKind.EndOfFileToken) {
    let currentToken = ts.formatSyntaxKind(token);
    let tokenStart = scanner.getStartPos();
    token = scanner.scan();
    let tokenEnd = scanner.getStartPos();
    console.log(currentToken, tokenStart, tokenEnd);
}
```
这将会输出下面的内容：
```ts
VarKeyword 0 3
Identifier 3 7
FirstAssignment 7 9
FirstLiteralToken 9 13
SemicolonToken 13 14
```

### 单例扫描器

尽管 TypeScript 转化器有一个单例扫描器，你可以使用`createScanner`创建一个单例扫描器，使用它的`setText`/`setTextPos`去扫描一个文件的不同位置，作为消遣。