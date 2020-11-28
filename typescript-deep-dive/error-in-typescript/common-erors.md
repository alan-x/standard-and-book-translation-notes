[已校对]
# 常见错误

在这个章节，我们解释一些用户在真实世界会遇见常见错误码。

### TS2304

例子：
> `Cannot find name ga Cannot find name $ Cannot find module jquery·`

你可能使用一个三方库（比如，谷歌分析），但是没有他的`declare`。TypeScript 尝试错误拼写错误和使用没有声明的变量中保护你，因此你需要明确在运行时可用的任何东西，因为你包含一些额外的库（[更多怎样修复这个](https://basarat.gitbook.io/typescript/type-system/intro/d.ts)）。

### TS2307

例子：
> Cannot find module 'underscore'

你可能使用一个三方库（比如，underscore）作为一个模块（[了解更多模块](https://basarat.gitbook.io/typescript/project/modules)）并且没有环境声明文件（[了解更多环境声明](https://basarat.gitbook.io/typescript/type-system/intro/d.ts)）。

### TS1148

例子：

> Cannot compile modules unless the '--module' flag is provided

查阅[模块章节](https://basarat.gitbook.io/typescript/project/modules)

### 捕获语句变量不能有类型声明

例子：
```ts
try { something(); }
catch (e: Error) { // Catch clause variable cannot have a type annotation
}
```

TypeScript 从 JavaScript 在自然状态下错误中保护你。使用一个类型守卫替代：
```ts
try { something(); }
catch (e) {
  if (e instanceof Error){
    // Here you go.
  }
}
```

### 接口`ElementClass`不能同时扩展类型`Component`和`Component`

这发生在你有两个`react.d.ts`(`@types/react/index.d.ts`)在编译上下文的时候。

**修复**

- 删除`node_modules`和任何`package-lock`（或者 yarn lock），然后再一次`npm install`。

- 如果不起作用，找到无效的模块（`react.d.ts`作为`peerDependency`，而不是严格的`dependency`）并在他们的项目报告这个。