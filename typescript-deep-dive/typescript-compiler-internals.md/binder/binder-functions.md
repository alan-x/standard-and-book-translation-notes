[已校对]
# 绑定器函数

两个标准绑定器函数是`bindSourceFile`和`mergeSymbolTable`。我们将在下面看到这些。

### `bindSourceFile`

基本上检测`file.locals`是否定义，如果没有，它就转交为（一个本地函数）`bind`。

注意：`locals`定义在`Node`，它的类型是`SymbolTable`。注意`SourceFile`也是一个`Node`（实际上是 AST 的根节点）。

提示：本地函数在 TypeScript 中大量使用。一个本地函数非常可能使用父函数的变量（被闭包捕获）。这个场景中，`bind`（一个`bindSourceFile`内的本地函数）它（或者一个它调用的函数）将会设置`SourceFile`和`classifiableNames`等，然后存储在返回的`SourceFile`。

### `bind`

绑定接受任何`Node`（不仅仅是`SourceFile`）。他做的第一件事是赋值`node.parent`（如果`parent`变量被设置...这也是绑定起在它的的`bindChildren`函数中的处理过程做的），然后放手给`bindWorker`去执行繁重的工作。最后它调用`bindChildren`（一个简单存储绑定起状态的函数，比如，它的函数的本地变量的当前`parent`，然后在每一个子节点调用`bind`，然后重新存储绑定器状态）。现在来看看`bindWorker`，是一个更有趣的函数。

### `bindWorker`

这个函数打开了`node.kind`（类型是`SyntaxKind`），并且代理工作到适合的`bindFoo`函数（当然定义在`binder.ts`内）。比如，如果`node`是一个`SourceFile`，一个`SourceFile`它调用（最终当且仅当它是一个外部文件模块）`bindAnonymousDeclaration`

### `bindFoo`函数

有一些模式对于`bindFoo`函数很常见，一些工具函数也使用。一个函数总是使用的是`createSymbol`函数。它完全标示在下面：
```ts
function createSymbol(flags: SymbolFlags, name: string): Symbol {
    symbolCount++;
    return new Symbol(flags, name);
}
```

正如你看到的，它只是保持`symbolCount`（一个本地`symbolCount`）更新，并使用指定的参数创建符号。