就像我们之前提到的，检查器是让 TypeScript 比其他 JavaScript 转化器威力更强大的东西。检查器位于`checker.ts`，这时候有 23k+ 行的TypeScript（编译器最大的部分）。


### 被 Program 使用

`checker`被`program`初始化。下面是简化的调用栈（我们在查看`bidner`的时候显示了相同的一个）：
```ts
program.getTypeChecker ->
    ts.createTypeChecker (in checker)->
        initializeTypeChecker (in checker) ->
            for each SourceFile `ts.bindSourceFile` (in binder)
            // followed by
            for each SourceFile `ts.mergeSymbolTable` (in checker)
```

### 和发射器关联

`getDiagnostics`调用后会发生真实的类型检测。这个函数在请求被`Program.emit`创建的时候调用，这种场景下，检查器返回了一个`EmitResolver`（program 调用检测器的`getEmitResolver`函数），只是位于`createTypeChecker`的一个集合的函数。当我们查看发射器的时候，我们会再一次提到。

下面是`checkSourceFile`的调用栈（一个位于`createTypeChecker`的函数）
```ts
program.emit ->
    emitWorker (program local) ->
        createTypeChecker.getEmitResolver ->
            // First call the following functions local to createTypeChecker
            call getDiagnostics ->
                getDiagnosticsWorker ->
                    checkSourceFile

            // then
            return resolver
            (already initialized in createTypeChecker using a call to local createResolver())
```