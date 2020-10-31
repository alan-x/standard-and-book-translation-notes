# Prettier

Prettier 是一个 facebook 提供的非常好的工具，让代码格式化非常简单，它值得提到。使用我们推荐的项目设置（aka `src`文件夹中的所有东西） TypeScript 非常简单：

### 设置

- `npm install prettier -D `
- 添加`scripts`到`package.json`：
```ts
    "prettier:base": "prettier --parser typescript --single-quote",
    "prettier:check": "npm run prettier:base -- --list-different \"src/**/*.{ts,tsx}\"",
    "prettier:write": "npm run prettier:base -- --write \"src/**/*.{ts,tsx}\""
```

### 使用

在你的构建服务器：
- `npm run prettier:check`
调试开发（或者前置 commit  hook）：
- `npm run prettier:write`