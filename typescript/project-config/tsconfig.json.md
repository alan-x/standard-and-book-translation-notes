### 概述

`tsconfig.json`文件存在在一个文件夹指示这个文件夹是 TypeScript 项目的根目录。`tsconfig.json`文件指定根文件和编译这个项目需要的配置选项。

JavaScript 项目可以使用`jsconfig.json`文件替代，表现几乎相同，但是有些 JavaScript 相关的编译器标志默认启用。

一个项目以下面的方式之一编译项目：

### 使用 tsconfig.json 或者 jsconfig.json

- 通过调用 tsc 而不使用输入文件，这种场景下，编译器在当前文件夹和父文件夹链搜索`tsconfig.json`文件。

- 通过调用 tsc 而不使用输入文件，和一个`--project`（或者指示`-p`）命令行选项指定一个包含`tsconfig.json`文件的文件夹路径，或者一个有效的包含配置的`.json`文件路径。

当输入文件指定在命令行的时候，`tsconfig.json`就会被忽略。

### 例子

使用`tsconfig,json`文件：

- 使用`files`属性
```ts
{
"compilerOptions": {
  "module": "commonjs",
  "noImplicitAny": true,
  "removeComments": true,
  "preserveConstEnums": true,
  "sourceMap": true
},
"files": [
  "core.ts",
  "sys.ts",
  "types.ts",
  "scanner.ts",
  "parser.ts",
  "utilities.ts",
  "binder.ts",
  "checker.ts",
  "emitter.ts",
  "program.ts",
  "commandLineParser.ts",
  "tsc.ts",
  "diagnosticInformationMap.generated.ts"
]
}
```

- 使用`include`和`exclude`属性
```ts
{
"compilerOptions": {
  "module": "system",
  "noImplicitAny": true,
  "removeComments": true,
  "preserveConstEnums": true,
  "outFile": "../../built/local/tsc.js",
  "sourceMap": true
},
"include": ["src/**/*"],
"exclude": ["node_modules", "**/*.spec.ts"]
}
```

### TSConfig 基础

取决于你想要你的代码运行的 JavaScript 运行时环境，有一些基础配置你可以使用，位于[github.com/tsconfig/bases](github.com/tsconfig/bases)。那里有`tsconfig.json`文件可以让你的项目继承，简化你的`tsconfig.json`，通过处理运行时支持。

比如，如果你编写的项目使用 Node.js 12 或者更早，则你可以使用 npm 模块[@tsconfig//node12]()：
```ts
{
  "extends": "@tsconfig/node12/tsconfig.json",

  "compilerOptions": {
    "preserveConstEnums": true
  },

  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```
这让你的`tsconfig.json`聚焦于你的项目特定的选项，而不是所有的运行时机制。已经有一些基础 tsconfig，我们希望社区可以添加更多不同的环境。

- [Recommended]()

- [Node 10]()

- [Node 12]()

- [Deno]()

- [React Native]()

- [Svelte]()

### 详情

`compilerOptions`属性可以缺省，这种场景下，编译器的默认选项没使用。查阅我们完整的[编译器选项]()支持列表。

### TSConfig 索引

为了学习关于[TSConfig 索引]()中上百中的配置选项的。

### 模式

`tsconfig.json`模式可以在[JSON Schema 商店]()找到。