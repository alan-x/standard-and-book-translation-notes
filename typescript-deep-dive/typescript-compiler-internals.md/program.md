# Program
---

定义在`program.ts`。编译上下文（）在 TypeScript 编译器中标示为`Program`。它由`SourceFile`和编译器选项组成。



### `CompilerHost`的使用

它是和 OE 的交互机制：

`Program`-uses->`CompilerHost`-uses->`System`

有一个`CompilerHost`作为间接点是因为它允许它的接口对`Program`更加友好，而不需要 OE（比如，`Program`不关心`System`提供的`fileExists`函数）。

也有其他`System`用户（比如，测试）

### SourceFile

program 提供一个 API 去获取源文件`getSourceFiles(): SourceFile[];`，每一个都表示为 AST（叫做`SourceFile`）的一个根节点。