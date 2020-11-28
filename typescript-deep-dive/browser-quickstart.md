[已校对]
# 浏览器入门

### 浏览器中的 TypeScript

![](https://raw.githubusercontent.com/basarat/typescript-book/master/images/designtsx-banner.png)

如果你使用 TypeScript 去创建一个 web 应用，这是我推荐的 TypeScript + React（我选择的 UI 框架）入门项目设置


#### 通用机器设置

- 安装[Node.js](https://nodejs.org/en/download/)
- 安装[Git](https://git-scm.com/downloads)

#### 快速项目设置

使用[https://github.com/basarat/react-typescript](https://github.com/basarat/react-typescript)作为基础
```ts
git clone https://github.com/basarat/react-typescript.git
cd react-typescript
npm install
```

现在使用使用它作为基础，并跳到[开发你的令人吃惊的应用](https://basarat.gitbook.io/typescript/browser#develop-your-amazing-application)

#### 项目设置详情

如果你想要学习更多关于项目是如何创建的细节（而不是使用它作为基础），这里是它怎样从头设置的步骤：

- 创建一个项目文件夹：
```ts
mkdir your-project
cd your-project
```
- 创建一个`tsconfig.json`：
```ts
{
  "compilerOptions": {
    "sourceMap": true,
    "module": "commonjs",
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "experimentalDecorators": true,
    "target": "es5",
    "jsx": "react",
    "lib": [
      "dom",
      "es6"
    ]
  },
  "include": [
    "src"
  ],
  "compileOnSave": false
}
```

- 创建`package.json`
```ts
{
  "name": "react-typescript",
  "version": "0.0.0",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/basarat/react-typescript.git"
  },
  "scripts": {
    "build": "webpack -p",
    "start": "webpack-dev-server -d --content-base ./public"
  },
  "dependencies": {
    "@types/react": "16.4.10",
    "@types/react-dom": "16.0.7",
    "clean-webpack-plugin": "0.1.19",
    "html-webpack-plugin": "3.2.0",
    "react": "16.4.2",
    "react-dom": "16.4.2",
    "ts-loader": "4.4.2",
    "typescript": "3.0.1",
    "webpack": "4.16.5",
    "webpack-cli": "3.1.0",
    "webpack-dev-server": "3.1.5"
  }
}
```

- 创建一个`webpack.config.js`去打包你的模块到一个单独的`app.js`文件，包含所有你的资源：
```ts
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/app/app.tsx',
  plugins: [
    new CleanWebpackPlugin({
      cleanAfterEveryBuildPatterns: ['public/build']
    }),
    new HtmlWebpackPlugin({
      template: 'src/templates/index.html'
    }),
  ],
  output: {
    path: __dirname + '/public',
    filename: 'build/[name].[contenthash].js'
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js']
  },
  module: {
    rules: [
      { test: /\.tsx?$/, loader: 'ts-loader' }
    ]
  }
}
```

- `src/templates/index.html`文件。它将会用作 webpack 生成的`index.html`的模板。生成的文件将会在`public`文件夹，并在你的 webserver 提供服务：
```ts
<html>
  <body>
      <div id="root"></div>
  </body>
</html>
```

- `src/app/app.tsx`是你的前端应用的入口：
```ts
import * as React from 'react';
import * as ReactDOM from 'react-dom';

const Hello: React.FunctionComponent<{ compiler: string, framework: string }> = (props) => {
  return (
    <div>
      <div>{props.compiler}</div>
      <div>{props.framework}</div>
    </div>
  );
}

ReactDOM.render(
  <Hello compiler="TypeScript" framework="React" />,
  document.getElementById("root")
);
```

#### 开发你的令人吃惊的应用

> 你可以使用`npm install typescript@latest react@latest react-dom@latest @types/react@latest @types/react-dom@latest webpack@latest webpack-dev-server@latest webpack-cli@latest ts-loader@latest clean-webpack-plugin@latest html-webpack-plugin@latest --save-exact`获取最新的包

- 通过运行`npm start`执行实时开发。
    - 访问[http://localhost:8080/](http://localhost:8080/)
    - 编辑`src/app/app.tsx`（或者任何 ts/tsx 文件以一些方式被`src/app/app.tsx`），应用实时加载
    - 编辑`src/templates/index.html`，服务端热加载
- 构建产品资源，通过运行`npm run build`
    - 从你的服务器提供`public`文件夹（包含构建的资源）