[已校对]
# 生成器


TypeScript 编译器中提供了两个`emitters`“
- `emitter.ts`：是你最可能感兴趣的部分。是 TS -> JavaScript 生成器。
- `declarationEmitter.ts`：这是用于一个 TypeScript 源文件创建声明文件（一个`.d.ts`）的生成器（一个`.ts`文件）

我们将在这个章节查看`emitter.ts`

### 被`program`使用

program 提供了一个`emit`函数。这个函数主要代理到`emitter.ts`的`emitFiles`函数。这是调用栈：
```ts
Program.emit ->
    `emitWorker` (local in program.ts createProgram) ->
        `emitFiles` (function in emitter.ts)
```

`emitWorker`提供给发射器的一个东西是一个`EmitResolver`。`EmitResolver`是由 program 的 TypeChecker 提供的，基本上它是本地函数`createChecker`的一个子集