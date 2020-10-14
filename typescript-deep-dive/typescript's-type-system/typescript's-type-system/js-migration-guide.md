# JS 升级指南

假设：
- 你知道 JavaScript
- 你知道项目中国呢使用的模式和构建工具（比如，webpack）

排除这些假设，通常流程由以下步骤组成：

- 添加`tsconfig.json`。
- 将你的文件扩展名从`.js`改为`.ts`。使用`any`压制错误。
- 使用 TypeScript 编写新的代码，并且尽可能少的使用`any`。
- 返回旧的代码并开始添加类型声明和修复标示的 bug。
- 使用为三方 JavaScript 代码使用环境定义。

让我们深入讨论一些点。

注意所有的 JavaScript 都是有效的 TypeScript。也就是说，如果你给 TypeScript 编译器一些 JavaScript -> TypeScript 编译器生成的 JavaScript 将会表现的和原始 JavaScript 文件完全相同。这意味着将后缀从`.js`改变为`.ts`将不会有害的影响你的代码库。

### 压制错误

TypeScript 将立即开始对你代码的类型检测，并且你的原始的 JavaScript 代码不会像你以为的那么整齐，因此将会得到类型错误。很多错误你都可以使用`any`覆盖，比如：
```ts
var foo = 123;
var bar = 'hey';

bar = foo; // ERROR: cannot assign a number to a string
```
尽管**错误是有效的**（并且大部分场景，类型推断信息将会比代码库不同部分的原始作者想象得更好），你的注意力将可能变为使用 TypeScript 编写新的代码，当渐进升级旧的代码库的时候。这里你可以使用类型断言覆盖错误：
```ts
var foo = 123;
var bar = 'hey';

bar = foo as any; // Okay!
```
在其他地方，你可能想要声明一些东西为`any`，比如：
```ts
function foo() {
    return 1;
}
var bar = 'hey';
bar = foo(); // ERROR: cannot assign a number to a string
```

覆盖为：
```ts
function foo(): any { // Added `any`
    return 1;
}
var bar = 'hey';
bar = foo(); // Okay!
```

> 笔记：覆盖错误是危险的，但是它允许你去注意你的新的 TypeScript 中的错误。你可能想要在 ** 之后留下`// TODO:`评论。


### 第三方 JavaScript

你可以改变你的 JavaScript 为 TypeScript，但是你不能改变整个世界去使用 TypeScript。这是 TypeScript 外部定义支持的地方。在开始的时候我们推荐你创建一个`vendor.d.ts`(`.d.ts`扩展指定这是一个声明文件的事实)并开始添加脏东西到它。另一种方式是为 jquery 库创建一个专门的文件，比如`jquery.d.ts`。

> 注意：为接近前 90% 的 JavaScript 库维护良好的和强类型的定义存在于叫做[DefinitelyTyped]()的 OSS 仓库。我们推荐在创建你自己的定义之前在那里查找这里我们展示的。然而，这快速和脏的方式是较少你和 TypeScript 初始摩擦至关重要的知识。

考虑`jquery`的用例，你可以快速简单的创建一个小的定义：
```ts
declare var $: any;
```

有时候你想要在一些东西（比如，`jQuery`）添加一个显式的声明，并且你需要一些东西在类型定义空间，你可以使用`type`关键字简单的做到：
```ts
declare type JQuery = any;
declare var $: JQuery;
```

这提供一个 简单的深入升级路径。

再一次，一个高质量的`jquery.d.ts`存在于[Definitely Typed]()。但是你限制知道怎样快速克服任何 JavaScript -> TypeScript 摩擦，当使用第三方 JavaScript 的时候。我们将在之后详细查看外部声明。

### 第三方 NPM 模块

和全局变量声明类似，你可以非常简单的声明一个全局模块。比如，对于`jquery`，如果你想要使用它作为一个模块（[https://www.npmjs.com/package/jquery]()），你可以自己编写下面的代码：
```ts
declare module "jquery";
```
然后你可以在需要的时候在你的文件导入他：
```ts
import * as $ from "jquery";
```

> 再一次，一个高质量的`jquery.d.ts`存在于[DefinitelyTyped]()，提供一个更高质量的 jquery 模块声明。但是它可能不存在你的库，因此现在你有一个快速低摩擦的方式继续升级🌹。

### 外部非 js 资源

你甚至可以允许导入任何类似`.css`的文件（如果你使用一些类似 webpack 风格的加载器或者 css 模块），使用一个简单的`*`风格声明（理想的在一个`[global.d.ts 文件]()`）：
```ts
declare module "*.css";
```
现在人们可以`import * as foo from "./some/file.css";`

同样，如果你使用 html 模板（比如 angular），你可以：
```ts
declare module "*.html";
```

### 更多

如果你想要更安静的升级，因为你不能让你的团队为移动到 TypeScript 买单，[TypeScript 有一个关于静默升级而不需要说服你的团队的博客文章]()。