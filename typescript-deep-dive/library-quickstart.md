# 库入门

- [一个创建 TypeScript 节点模块的课程](https://egghead.io/lessons/typescript-create-high-quality-npm-packages-using-typescript)

使用 TypeScript 编写的模块非常有趣，因为你会德奥非常好的编译时安全性和自动完成（基本上是可执行的文档）。

TypeScript 模块可以被 nodejs（照常）和浏览器（使用类似 webpack 的东西）消费

创建一个高质量的 TypeScript 模块非常简单。假设下面是你的包期待的文件夹结构：
```ts
package
├─ package.json
├─ tsconfig.json
├─ src
│  ├─ index.ts
│  ├─ foo.ts
│  └─ ...All your source files (Authored)
└─ lib
  ├─ index.d.ts.map
  ├─ index.d.ts
  ├─ index.js
  ├─ foo.d.ts.map
  ├─ foo.d.ts
  ├─ foo.js
  └─ ... All your compiled files (Generated)
```

- `src/index.ts`：这里你将抛出任何你期待从你的项目被消费的东西。比如`export { Foo } from './foo';`。从这个文件导出让它可以被消费者使用，当某人执行`import { /* Here */ } from 'example';`。
- 在你的`tsconfig.json`
    - 有`compilerOptions`：`"outDir": "lib"` + `"declaration": true` + `"declarationMap" : true` < 这生成`.js`（JavaScript）`.d.ts`（为 TypeSafety 声明）和`.d.ts.map`（启用`declaration .d.ts` => `source .ts` IDE 导航）在 lib 文件夹
    - 有`include: ["src"]`< 这包含`src`文件夹所有的文件
- 在你的`package.json`
    - `include: ["src"]`< 这告诉`lib/index.js`去加载运行时代码
    - `"types": "lib/index"`< 这告诉 TypeScript 为类型检测去加载`lib/index.d.ts`

包例子：
- 为 [TypeStyle]() 执行`npm install typestyle`
- 使用：`import { style } from 'typestyle';`将会完全类型安全

### 管理依赖

#### devDependencies

- 如果你的包依赖其他包，当你在开发它（比如，`prettier`）的时候，你应该安装他们作为一个`devDependency`。这个方式，他们将不会污染你的模块的消费者的`node_modules`（因为`npm i foo`不安装`foo`的`devDependencies`）。
- `typescript`通常是一个`devDependency`，因为你只用它构建你的包。消费者可以使用你的包，不管有没有 TypeScript

- 如果你的包依赖其他 JavaScript 创作的包，你想要使用它，但是你的项目没有类型安全，放置他们的类型（比如，`@types/foo`在`devDependencies`）。JavaScript 类型应该在主 NPM 流之外被管理。JavaScript 生态系统经常破坏类型，没有语义话版本，因此如果你的用户需要类型，他们应该安装可以为他们工作的`@types/foo`版本。如果你想要指导用户去安装这些类型，聂可以将他们放到下面提到的`peerDependencies`。

#### peerDependencies

如果你的包严重依赖一个包（相对于工程使用），比如,`react`，将他们放到`peerDependencies`，就像你使用原生 JS 包。本地测试他们你应该也放他们到`devDependencies`。

现在：
- 当你开发包，你将会得到你的`devDependencies`指定的依赖的版本号。

- 当某人安装你的包，他们将不会得到依赖（因为`npm i foo`不会为`foo`安装`devDependencies`），但是他们将得到一个警告，他们应该为你安装丢失的`devDependencies`。

#### dependencies

如果你的包包裹其他包（意味着内部使用，甚至在编译之后），你应该将他们放到`dependencies`。现在当某人安装你的包的时候。他们将得到你的包 + 任何它的依赖。
