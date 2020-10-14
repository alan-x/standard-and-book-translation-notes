# 入门

- [从 TypeScript 开始]()
- [TypeScript Version]()

### TypeScript 入门

TypeScript 编译成 JavaScript。JavaScript 是你实际运行的东西（不管是在浏览器还是在服务器）。因此你需要下面的东西：

- TypeScript 编译器（OOS 可以在[源码]()和[npm]()得到）
- 一个 TypeScript 编辑器（如果你喜欢你可以使用记事本，但是我使用[vs code]()🌹和一个[我编写的创建]()。当然[也有很多 IDE 也都支持它]()）


### TypeScript 版本

与其使用稳定版的 TypeScript 编译器，我们将在这本书中展示很多新的东西，这些东西还没有和一个版本号关联。我通常推荐人们使用 nightly 版本，因为**编译器测试套件只会随着时间捕获更多的错误**。

你可以在命令行如下安装它：
```
npm install -g typescript@next
```

现在命令行`tsc`将会是最新和最大的。大部分的 IDE 也都支持它，比如。

- 你可以让 vscode 使用这个版本，通过使用下面的内容创建`.vscode/settings.json`文件：
```
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}

```

### 获取源码

这本书的源码在这本书的 github 仓库 [https://github.com/basarat/typescript-book/tree/master/code]()可以得到，大部分的代码例子可以复制到 vscode 中按照原来的例子使用。对于需要额外设置的代码例子（比如，npm 模块），在展示代码之前，我们将会把你链接到代码。比如。
`this/will/be/the/link/to/the/code.ts`

```
// This will be the code under discussion
```

在完成开发设置之后，开始进入 TypeScript 语法吧。
