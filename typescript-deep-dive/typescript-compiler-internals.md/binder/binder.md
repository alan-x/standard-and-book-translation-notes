[已校对]
# 绑定器

这里之外的大部分 JavaScript 转化器都比 TypeScript 简单，因为它提供了很小的代码分析方式。经典的 JavaScript 转化器只有下面的流程：

```ts
SourceCode ~~Scanner~~> Tokens ~~Parser~~> AST ~~Emitter~~> JavaScript
```

尽管前面的架构可以作为简单的理解 TypeScript js 生成，TypeScript 的一个关键特性是它的语义分析。为了辅助类型检测（被`checker`执行），`binder`（在`binder.ts`）用于链接多个部分的源代码到一个连贯的类型系统，可以背`checker`使用。绑定器具的主要责任是去创建符号。

### 符号

符号链接 AST 的声明节点到贡献给相同入口的其他声明。符号是语义系统最基本的构建块。副高构造器定义在`core.ts`（`binder`实际使用`objectAllocator.getSymbolConstructor`得到）。这是副高构造器：
```ts
function Symbol(flags: SymbolFlags, name: string) {
    this.flags = flags;
    this.name = name;
    this.declarations = undefined;
}
```

`SymbolFlags`是一个标志枚举，实际用于标示额外的符号类别（比如，变量范围标示`FunctionScopedVariable`，或`BlockScopedVariable`等其他）

## 被检查器使用

`binder`在内部实际被`checker`类型使用，一次被`program`使用。简化的调用栈看起来像：
```ts
program.getTypeChecker ->
    ts.createTypeChecker (in checker)->
        initializeTypeChecker (in checker) ->
            for each SourceFile `ts.bindSourceFile` (in binder)
            // followed by
            for each SourceFile `ts.mergeSymbolTable` (in checker)
```

绑定器的工作单元是一个 SourceFile。`binder.ts`被`checker.ts`驱动。