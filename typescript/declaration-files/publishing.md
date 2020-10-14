随着这个指南的步伐，现在你已经制作了一个声明文件，是时候去发布它到 npm。有两种主要方式去发布你的声明到 npm：

1. 打包你的 npm 包
2. 发布到 npm 的[@types 组织]()

如果你的类型是通过源代码生成的，和你的源代码发布。TypeScript 和 JavaScript 项目都可以通过[--declaration]()生成。

此外，我们推荐提交类型到 DefinitelyTyped，将会发布他们到 npm 的`@types`组织。

### 在你的 npm 包引入声明

如果你的包有一个主要的`.js`文件，你将需要在你的`package.json`去指示主要的声明文件。设置`types`属性去指出你的打包的声明文件。比如：
```ts
{
  "name": "awesome",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts"
}
```

注意`typings`域和`types`是一样的，可以同样使用。

同时也要注意如果你的主要声明文件叫做`index.d.ts`，并且在包的根目录（在`index.js`之后），你不需要标志`types`属性，尽管这推荐这么做。

### 依赖

所有的依赖被 npm 管理。确保你依赖的所有的声明包取决于你的`package.json`中`dependencies`域的正确声明。比如，想象我们使用 Browserify 和 TypeScript 制作了一个包。
```ts
{
  "name": "browserify-typescript-extension",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts",
  "dependencies": {
    "browserify": "latest",
    "@types/browserify": "latest",
    "typescript": "next"
  }
}
```

这里，我们的包依赖了`browserify`和`typescript`包。`browserify`不使用 npm 包打包他们依赖文件，因此，我们需要依赖`@types/browserify`他的声明。`typescript`，相反，打包他的声明文件，因此不需要任何额外的依赖。

我们的包暴露了声明文件，任何我们的`browserify-typescript-extension`的用户也需要这些依赖。为了这个原因，我们使用`dependencies`，而不是`devDependencies`，因为我们其他的消费者将会需要手动安装这些包。如果我们只是编写命令行应用程序，不希望我们的包被用作库，我们可以使用`devDependencies`。


### 红色标志

#### /// <reference path="..." />

不要在你的声明文件中使用`/// <reference path="..." />`
```ts
/// <reference path="../typescript/lib/typescriptServices.d.ts" />
....
```

使用`/// <reference types="..." />`替代
```ts
/// <reference types="typescript" />
....
```

确保重新访问[依赖消费]()章节了解更多信息

#### 打包依赖声明

如果你的类型声明取决于其他包：

- 不要和它绑定，将宝们每一个保存在各自的文件

- 不要复制声明文件到你的包

- 依赖于 npm 类型声明包，如果它不需要打包它自己的声明文件。


### 使用 typesVersions 版本选择

当 TypeScript 打开一个`package.json`文件去指出它需要读取哪一个文件，它首先查看`typesVersions`的域。

一个有`typesVersions`的`package.json`域可能看起来像这样：
```
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.1": { "*": ["ts3.1/*"] }
  }
}
```

`package.json`告诉 TypeScript 去检测当前运行的 TypeScript 版本。如果他是 3.1 或者更新，它指出你相对于包要导入的路径，并从`ts3.1`读取。这也是`{ "*": ["ts3.1/*"] }`的意义 - 如果你熟悉路径映射，它和这个工作的很像。

在签名的例子中，如果我们从`package-name`导入，TypeScript 将会尝试从`[...]/node_modules/package-name/ts3.1/index.d.ts`（和其他相对路径）解析，当运行在 TypeScript 3.1 的时候。如果我们从`package-name/foo`导入，我们将尝试查找`[...]/node_modules/package-name/ts3.1/foo.d.ts`和`[...]/node_modules/package-name/ts3.1/foo/index.d.ts`。

如果我们不是在 TypeScript 3.1 运行呢？也就是，如果没有命中`typesVersions`的域，TypeScript 回落到`types`域，因此，这里，TypeScript 3.0 和更早的版本将会重定向到`[...]/node_modules/package-name/index.d.ts`。

#### 命中行为

TypeScript 决定哪一个版本的编译器和语言被命中，通过[语义化范围]()。

### 多域

`typesVersions`可以支持多个域，每一个域的明知通过范围去匹配。
```ts
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.2": { "*": ["ts3.2/*"] },
    ">=3.1": { "*": ["ts3.1/*"] }
  }
}

```

因为范围是可以重载的，决定哪一个直接应用是顺序指定的。这意味着在前面的例子中，景观`>=3.2`和`>=3.1`匹器支持 TypeScript 3.2 和更高，将顺序相反将有一个还权不同的行为，因此，前面的例子和下面不同：
```ts
{
  name: "package-name",
  version: "1.0",
  types: "./index.d.ts",
  typesVersions: {
    // NOTE: this doesn't work!
    ">=3.1": { "*": ["ts3.1/*"] },
    ">=3.2": { "*": ["ts3.2/*"] },
  },
}
```


### 发布到[@types]()

[@types]()组织下面的包使用[类型发布者工具]()自动从[DefinitelyTyped]()发布。为了得到你的声明发布为`@type`包，请提交一个 pull request 到[DefinitelyTyped]()。你可以在[共享指南页面]()找到更多细节。