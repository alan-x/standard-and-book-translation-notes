# ChangeLog

> 阅读一个关于项目进度的 markdown 文件比阅读一个 commit 日志更简单

从提交信息自动生成 changelog 是现在的常见模式。有一个项目叫做[conventional-changelog]()可以从提交信息生成一个遵守约定的 changelog。

### 提交信息约定

最常见的约定是 angular 提交信息约定，在这里[详细记录]()。

### 设置

- 安装：
```ts
npm install standard-version -D
```
- 添加一个`script`目标到你的`package.json`：
```ts
{
  "scripts": {
    "release": "standard-version"
  }
}
```

- 可选的：为了自动推送新的提交信息和增加标签发布到 npm 添加一个`postrelease`脚本：
```ts
{
  "scripts": {
    "release": "standard-version",
    "postrelease": "git push --follow-tags origin master && npm publish"
  }
}
```

### 发布

简单运行：
```ts
npm run release
```

基于提交信息`major`|`minor`|`patch`是自动决定的。为了明确指定一个版本，你可以指定`--release-as`，比如：
```ts
npm run release -- --release-as minor
```