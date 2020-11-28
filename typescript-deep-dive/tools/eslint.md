[已校对]
# ESLint

ESLint 用于对齐 JavaScript，但是现在也成为[TypeScript](https://github.com/Microsoft/TypeScript/issues/29288)实际上的 linter，感谢两个团队的[合作](https://eslint.org/blog/2019/01/future-typescript-eslint)

### 安装

为 TypeScript 设置 ESLint ，你需要下面的包：
```ts
npm i eslint eslint-plugin-react @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

> 提示：eslint 调用包包含 lint 规则作为“插件”

- eslint：核心 eslint
- eslint-plugin-react：eslint 提供的 react 规则。[支持的规则列表](https://github.com/yannickcr/eslint-plugin-react#list-of-supported-rules)
- @typescript-esling/parse：让 eslint 理解 ts/tsx 文件
- @typeScript-eslint/eslint-plugin：为 TypeScript规则。[支持的规则列表](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin#supported-rules)

> 就像你看到的，这里有两个 eslint 包（为了和 js 或者 ts 使用）和两个 @typescript=eslint 包（为了和 ts 使用）。因此 TypeScript 的消耗不是非常大。

### 配置
创建`.eslintrc.js`：
```ts
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json',
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
  ],
  rules:  {
    // Overwrite rules specified from the extended configs e.g. 
    // "@typescript-eslint/explicit-function-return-type": "off",
  }
}
```

### 运行
在你的`package.json`添加脚本：
```ts
{
  "scripts": {
    "lint": "eslint \"src/**\""
  }
}
```

### 配置 VSCode

- 安装扩展[https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- 添加`settings.json`：
```ts
"eslint.validate":  [
"javascript",
"javascriptreact",
{"language":  "typescript",  "autoFix":  true  },
{"language":  "typescriptreact",  "autoFix":  true  }
],
```