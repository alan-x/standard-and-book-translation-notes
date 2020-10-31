# Husky

> Husky 可以阻止坏的提交，推送和更多！

如果你想要运行一些 JavaScript / TypeScript 代码，在一个提交发生之前，husky 就是这么一个工具。

比如，你可以使用 husky 去确保文件被自动被 prettier 格式化，因此不需要要担心手动格式化文件并关注代码的目标。这是设置：

- `npm install husky -D`
- 添加`script`到`package.json`：
```ts
    "precommit": "npm run prettier:write",
```

现在，当你提交代码的时候，任何需要执行的格式化改变，将会作为修改文件在你的 git 日志。你现在可以：

- 如果你已经推送了代码，简单使用一个评论`pretty`提交他们。
- 如果你还没推送他们，添加他们到你的罪行提交，就像一个超级英雄。