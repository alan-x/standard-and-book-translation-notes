# Node.js 入门

TypeScript 从一开始就对 Node.js 有第一级的支持。这是快速设置 Node.js 项目的设置：

> 注意：很多这些步骤实际上只是 Node.js 设置步骤的常见实践

1. 设置一个 Node.js 项目的`package.json`。快速：`npm init -y`
2. 添加 TypeScript（`npm install typescript --save-dev`）
3. 添加`node.d.ts`(`npm install @types/node --save-dev`)
4. 为 TypeScript 初始化一个`tsconfig.json`，并在你的 tsconfig.json 设置一些关键选项（`npx tsc --init --rootDir src --outDir lib --esModuleInterop --resolveJsonModule --lib es6,dom  --module commonjs`）

就是这样！启动你的 IDE（比如，`code .`）并运行。现在你可以使用所有内置的 node 模块（比如，`import * as fs from 'fs';`），还有 TypeScript 的安全性和开发者生态！

你的所有 TypeScript 代码都在`src`，生成的 JavaScript 在`lib`。

### Bonus：实时编译 + 运行

- 添加`ts-node`，我们将在代码使用实时编译+运行（`npm install ts-node --save-dev`）
- 添加`nodemon`，调用`ts-node`，当一个文件改变的时候（`npm install nodemon --save-dev`）

现在添加`script`目标到你的`package.json`，基于你的应用入口，比如，假设是`index.ts`：
```ts
  "scripts": {
    "start": "npm run build:live",
    "build": "tsc -p .",
    "build:live": "nodemon --watch 'src/**/*.ts' --exec \"ts-node\" src/index.ts"
  },

```

因此你现在可以运行`npm start`，当你编辑`index.ts`：

- nodemon 返回它的命令（ts-node）
- ts-node 转化自动选中 tsconfig.json 和安装的 TypeScript 版本
- ts-node 通过 Node.js 运行输出的 JavaScript

当你准备去部署你的 JavaSript 应用，运行`npm run build`

### Bonus 的观点

搜索和 browerify（使用tsify）或者 webpack（使用 ts-loader）合作很好的 NPM 模块。

