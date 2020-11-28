[已校对]
# 贡献

TypeScript 是 OSS，并且在 GitHub，并且团队欢迎社区输入。

### 设置

非常简单：
```ts
git clone https://github.com/Microsoft/TypeScript.git
cd TypeScript
npm install -g jake
npm install
```

### 设置 fork

明显你需要设置 Microsoft/TypeScript 作为一个`upstream`远程，还有你的 for（使用 GitHub fork 按钮）作为`origin`：
```ts
git remote rm origin
git remote rm upstream
git remote add upstream https://github.com/Microsoft/TypeScript.git
git remote add origin https://github.com/basarat/TypeScript.git
``` 
此外，我喜欢在类似`bas/`的分支工作，让他在分支列表中显示的更清晰。

### 运行测试

有很多`test`和`build`选项在他们的 JakeFile。你可以使用`jake runtests`运行所有测试。

### 基线

基线用于管理，如果 TypeScript 编译器的期待输出于任何改变。基线位于`tests/baselines`。

- 索引（期待的）基线：`tests/baselines/reference`
- 生成（在这个测试运行）基线：`tests/baselines/local`（这个文件位于 .gitignore）。

> 如果有任何不同，这些测试集合将会失败。你可以使用比较两个文件夹，比如 BeyondCompare 或者 KDiff3。

如果你认为生成文件的这些改是有效的，则使用`jake baseline-accept`接受基线。`reference`基线的改变将会展示位一个 git diff，你可以提交。

> 注意：如果 in不运行所有的测试则使用`jake baseline-accept[soft]`，它只赋值新文件，不会删除整个`reference`文件夹。

### 测试分类

对于不同的场景有不同的测试，甚至有不同的测试基础设施。这里是一些解释

#### 编译器测试

这确保编译一个文件：

- 生成预期的错误
- 生成预期的 JS
- 类型如预期定义
- 符号如预期定义

这些预期被验证，使用基线架构

##### 创建一个编译器测试

可以通过添加一个新的文件`yourtest.ts`到`tests/cases/compiler`创建。一旦你这么做并运行测试，你应该割稻一个基线错误，接受这些基线（让他们在 git 中展示），并调整他们到你期待的...现在让这些测试通过。


使用`jake runtests tests=compiler`独立运行，或者只是你的新文件`jake runtests tests=compiler/yourtest`。

我将会经常执行`jake runtests tests=compiler/yourtest || jake baseline-accept[soft]`，并在`git`中得到 diff

### 调试测试

`jake runtests-browser tests=theNameOfYourTest`并在浏览器调试好，通常工作的很好。

### 更多

- 一个 Remo 文章：[https://dev.to/remojansen/learn-how-to-contribute-to-the-typescript-compiler-on-github-through-a-real-world-example-4df0](https://dev.to/remojansen/learn-how-to-contribute-to-the-typescript-compiler-on-github-through-a-real-world-example-4df0)

