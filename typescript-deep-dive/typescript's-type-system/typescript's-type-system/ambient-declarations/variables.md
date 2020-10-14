# 变量

比如告诉 TypeScript 关于[`process`变量]()，你可以这么做：
```ts
declare var process: any;
```
> 你不需要为`process`做这些东西，已经有一个[社区维护的`node.d.ts`]()。

这芸汐你去使用`process`变量而不需要 TypeScript 编译：
```ts
process.exit();
```

我们推荐在可能的地方使用接口，比如：
```ts
interface Process {
    exit(code?: number): void;
}
declare var process: Process;
```

这允许其他人去扩展这些全局变量，同时告诉 TypeScript 关于这些修改。比如，假设下面的场景，我们为了娱乐添加了一个`exitWithLogging`函数到 process：
```ts
interface Process {
    exitWithLogging(code?: number): void;
}
process.exitWithLogging = function() {
    console.log("exiting");
    process.exit.apply(process, arguments);
};
```
接下来更仔细看看接口吧