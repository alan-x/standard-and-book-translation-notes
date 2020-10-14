### 对比 JavaScript 和一个 DTS 例子

### 常见 CommonJS 模式

一个使用 CommonJS 模式的模块使用`module.exports`去描述导出的值。比如，这是一个导出一个函数和一个数字常量的模块：
```ts
const maxInterval = 12;

function getArrayLength(arr) {
  return arr.length;
}

module.exports = {
  getArrayLength,
  maxInterval,
};
```

这可以通过下面的`.d.ts`描述：
```ts
export function getArrayLength(arr: any[]): number;
export const maxInterval: 12;
```

TypeScript 演练场可以为你显示和`.d.ts`相同的 JavaScript 代码。你可以[在这里自己尝试]()。

`.d.ts`语法故意和[ES 模块]()语法和祥。ES 模块在 2019 被 TC39 批准，虽然它已经可以通过转化器使用很久了，然而，如果你有一个使用 ES 模块的 JavaScript 代码库：
```ts
export function getArrayLength(arr) {
  return arr.length;
}
```

这和下面的`.d.ts`相同：
```
export function getArrayLength(arr: any[]): number;

```

#### 默认导出

在 CommonJS，你可以导出任何值作为默认导出，比如，这是一个正则表达式模块：
```ts
module.exports = /hello( world)?/;

```
可以通过下面 .d.ts 描述：
```ts
declare const helloWorld: RegExp;
export default helloWorld;
```

或者一个数字：
```ts
module.exports = 3.142;

```
```ts
declare const pi: number;
export default pi;
```

在 CommonJS 中，一个常见导出风格是导出一个函数。因为函数也是一个对象，额外的域也可以添加到这个导出。

可以使用如下描述：
```ts
export default function getArrayLength(arr: any[]): number;
export const maxInterval: 12;
```

注意，在你的 .d.ts  文件使用`export default`需要[esModuleInterop: true]()可用。如果你无法在你的项目使用`esModuleInterop: true`，比如当你提交一个 PR 到 Definitely Typed 的时候，你需要使用`export=`语法替代。这个旧的语法很难用，但是可以在任何地方工作。这是前面例子如何使用`export=`的例子：
```ts
declare function getArrayLength(arr: any[]): number;
declare namespace getArrayLength {
  declare const maxInterval: 12;
}

export = getArrayLength;
```

查阅[Module: Function]()了解更多是如何工作的，还有[Module 索引]()页面。


### 处理多个消费导入

在现代消费代码中，有很多种方式去导入一个模块：
```ts
const fastify = require("fastify");
const { fastify } = require("fastify");
import fastify = require("fastify");
import * as Fastify from "fastify";
import { fastify, FastifyInstance } from "fastify";
import fastify from "fastify";
import fastify, { FastifyInstance } from "fastify";
```

转化这些所有的例子需要 JavaScript 代码去实际支持所有的模式。为了支持这些模式，一个 CommonJS 模块需要看起来像这样：
```ts
class FastifyInstance {}

function fastify() {
  return new FastifyInstance();
}

fastify.FastifyInstance = FastifyInstance;

// Allows for { fastify }
fastify.fastify = fastify;
// Allows for strict ES Module support
fastify.default = fastify;
// Sets the default export
module.exports = fastify;
```

### 模块中的类型
你可能想要为 JavaScript 代码提供一个比存在的类型
```ts
function getArrayMetadata(arr) {
  return {
    length: getArrayLength(arr),
    firstObject: arr[0],
  };
}

module.exports = {
  getArrayMetadata,
};

```
可以描述为：
```ts
export type ArrayMetadata = {
  length: number;
  firstObject: any | undefined;
};
export function getArrayMetadata(arr: any[]): ArrayMetadata;
```

这个例子是[使用泛型]()去提供丰富类型信息的例子：
```ts
export type ArrayMetadata<ArrType> = {
  length: number;
  firstObject: ArrType | undefined;
};

export function getArrayMetadata<ArrType>(
  arr: ArrType[]
): ArrayMetadata<ArrType>;
```

现在数组的类型冒泡到了`ArrayMetadata`类型。

导出的类型可以被模块的消费者重新使用，TypeScript 代码或者[JSDoc 导入]()的`import`或者`import type`。

#### 模块代码中的命名空间

尝试描述 JavaScript 运行时关系是很棘手的。当 ES 模块类似语法没有提供足够的工具去描述导出的时候，你可以使用`namespace`。

比如，你可能有足够复杂的类型去描述你选择命名空间在你的`.d.ts`描述：
```ts
// This represents the JavaScript class which would be available at runtime
export class API {
  constructor(baseURL: string);
  getInfo(opts: API.InfoRequest): API.InfoResponse;
}

// This namespace is merged with the API class and allows for consumers, and this file
// to have types which are nested away in their own sections.
declare namespace API {
  export interface InfoRequest {
    id: string;
  }

  export interface InfoResponse {
    width: number;
    height: number;
  }
}
```

理解命名空间在`.d.ts`文件中如何工作，请阅读[.d.ts 深入]()。

#### 可选的全局使用

你可以使用`export as namespace`去声明你的模块将会在 UMD 上下文的全局空间可用：
```
export as namespace moduleName;

```

### 索引例子

为了给出一个关于怎样让这些所有的东西都聚合在一起的想法，这里有一个`.d.ts`索引去开始常见一个新的模块
```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ If this module is a UMD module that exposes a global variable 'myLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myLib;

/*~ If this module exports functions, declare them like so.
 */
export function myFunction(a: string): string;
export function myOtherFunction(a: number): number;

/*~ You can declare types that are available via importing the module */
export interface SomeType {
  name: string;
  length: number;
  extras?: string[];
}

/*~ You can declare properties of the module using const, let, or var */
export const myField: number;
```

#### 库文件布局

你的声明文件的布局应该和库的布局类似。

一个库可以由多个模块构成，比如
```ts
myLib
  +---- index.js
  +---- foo.js
  +---- bar
         +---- index.js
         +---- baz.js
```
这些可以导入为：
```ts
var a = require("myLib");
var b = require("myLib/foo");
var c = require("myLib/bar");
var d = require("myLib/bar/baz");

```
你的声明文件应该是：
```ts
@types/myLib
  +---- index.d.ts
  +---- foo.d.ts
  +---- bar
         +---- index.d.ts
         +---- baz.d.ts
```

#### 测试你的类型

如果你计划提交这些改变到 DefinitelyTyped ，让其他人也可以使用，推荐你：

1. 创建一个新的文件夹在`node_modules/@types/[libname]`
2. 创建一个`index.d.ts`在这个文件见，然乎复制例子进去
3. 看看模块的使用在什么地方坏了，并尝试去修复 index.d.ts
4. 当你开心的时候，克隆[DefinitelyTyped/DefinitelyTyped]()并遵循 README 的指令

否则

1. 创建新的文件在你的源码树根目录：`[libname].d.ts`
2. 添加`declare module "[libname]" { }`
3. 在声明的模块的花括号中添加添加模版，再看看什么地方的使用断了。