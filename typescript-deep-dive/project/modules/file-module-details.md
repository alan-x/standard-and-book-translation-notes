[已校对]
# 文件模块细节

TypeScript 内部模块模式包含了大量威力和可用性。这里我们讨论他的威力和一些需要映射真实世界使用的模式。

### 解释： commonjs，amd，es module，其他

首先我们需要阐明（糟糕）模块系统的不一致性。我将只展示我现在推荐的，并移除噪音，比如，不展示所有其他可以工作的方式。

根据`module`选项，TypeScript 可以从相同的 TypeScript 生成不同的 JavaScript。这是你可以忽略的东西（我没有兴趣探索死亡的技术）

- AMD：不要使用，只有浏览器
- SystemJS：是一个好的体验，被 ES 模块替代
- ES 模块：还没准备好。

现在这些只是声明 JavaScript 的选项。使用`module:commonjs`替代这些选项。

你编写 TypeScript 模块的方式也有一点混乱。再一次，这是现在不能做的方式：
-  `import foo = require('foo')`，比如`import/require`。使用 ES 模块语法替代。

很好，解决了这个问题之后，来看看 ES 模块语法。

> 描述：使用`module:commonjs`和使用 ES 模块语法去 导入/导入/ 创作模块

### ES 模块语法

导出一个变量（或者类型）很简单，只要添加`export`前缀就行了，比如：
```ts
// file `foo.ts`
export let someVar = 123;
export type SomeType = {
  foo: string;
};
```
- 导出一个变量或者类型在一个专有的`export`语句，比如
```ts
// file `foo.ts`
let someVar = 123;
type SomeType = {
  foo: string;
};
export {
  someVar,
  SomeType
};
```

- 导出一个变量或者类型在一个专有的`export`预语句并重命名名
```ts
// file `foo.ts`
let someVar = 123;
export { someVar as aDifferentName };
```

- 使用`import`引入一个变量或者类型
```ts
// file `bar.ts`
import { someVar, SomeType } from './foo';
```

- 使用`import`导入一个变量或者类型，并重命名
```ts
// file `bar.ts`
import { someVar as aDifferentName } from './foo';
```

- 使用`import * as`从一个模块导入任何东西到一个名字
```ts
// file `bar.ts`
import * as foo from './foo';
// you can use `foo.someVar` and `foo.SomeType` and anything else that foo might export.
```

- 使用单独的导入语句导入一个文件的副作用
```ts
import 'core-js'; // a common polyfill library
```

- 从其他制作的模块重新导出所有项
```ts
export * from './foo';
```

- 冲其他制作模块重新导出一些项
```ts
export { someVar } from './foo';
```

- 从其他制作模块重新导出一些项目并重命名
```ts
export { someVar as aDifferentName } from './foo';
```

### 默认导出/导入

正如你在之后将会学到的，我不是默认导出的粉丝。不过，这是导出和使用默认导出的语法

- 使用`export default`导出
    - 在一个变量（不需要`let/const/var`）
    - 在一个函数之前
    - 在一个类之前

```ts
// some var
export default someVar = 123;
// OR Some function
export default function someFunction() { }
// OR Some class
export default class SomeClass { }
```

- 使用`import someName from "someModule"`语法导入（你可以重命名导入的为任何你想要的），比如
```ts
import someLocalNameForThisFile from "../foo";
```

### 模块路径

> 我假设`moduleResolution: "Node"`。这是你应该在你的 TypeScript 配置中的选项。这个设置意味着自动为`module:commonjs`。

有两种不同类型的模块。这种不同通过导入语句的路径部分区分（比如，`import foo from 'THIS IS THE PATH SECTION'`）。

- 相对路径模块（路径以`.`开始，比如`./someFile`或者`../../someFolder/someFile`等）
- 其他动态查找模块（比如，`'core-js'`或者`'typestyle'`或者`'react'`甚至`react/core`等）。


主要的不同时模块在文件系统中如何解析。

> 我会使用概念术语 place，我将会在解释查找模式之后解释它。

### 相对路径模块

简单，只是跟随在相对路径

- 如果文件`bar.ts`执行`import * as foo from './foo';`，则 place`foo`必须存在于相同的文件夹。

- 如果文件`bar.ts`执行`import * as foo from '../foo';`，则 pace`foo`必须存在于上一个文件夹


- 如果文件`bar.ts`执行`import * as foo from '../someFolder/foo';`，则上一个文件夹，必须有一个带 place `foo`文件夹`someFolder`。

或者任何其他你可以想到的相对路径:)

### 动态查找

当导入路径不是相对的，查找通过[node 风格解析](https://nodejs.org/api/modules.html#modules_all_together)查找。这里我只给出一个简单的例子：

- 你有`import * as foo from 'foo'`，下面是顺序查找的 place
    - `./node_modules/foo`
    - `../node_modules/foo`
    - `../../node_modules/foo`
    - 直到文件系统根目录

- 你有`import * as foo from 'something/foo'`，下面是顺序查找的 place
    - `./node_modules/something/foo`
    - `../node_modules/something/foo`
    - `../../node_modules/something/foo`
    - 直到文件系统根目录

### place 是啥

当我说检查 place 的时候，我说的是下面的东西在这个地方被检查。比如，对于 place `foo`：

- 如果 place 是一个文件，比如，`foo.ts`，欢呼！
- 否则如果 place 是一个文件夹，并且存在文件`foo/index.ts`，欢呼！
- 否则如果 place 是一个文件并且有一个`foo/package.json`，并且 package.json 中的`types`键指定的文件存在，则欢呼！
- 否则如果 place 是一个文件，并且有一个`package.json`，并且 package.json 中`main`键指定的文件存在，则欢呼！

对于文件，我实际指的是`.ts`/`.d.ts`和`.js`。

就是这了，现在你是模块查找专家了（不是一个小特性）。


### 推翻只为类型的动态查找

你可以使用`declare module 'somePath'`声明一个全局模块，然后导入将会魔法般的解析道这个路径。

比如：
```ts
// global.d.ts
declare module 'foo' {
  // Some variable declarations
  export var bar: number; /*sample*/
}
```

然后：
```ts
// anyOtherTsFileInYourProject.ts
import * as foo from 'foo';
// TypeScript assumes (without doing any lookup) that
// foo is {bar:number}
```

### `import/require`只导入类型

下面的语句：
```ts
import foo = require('foo');
```
实际只做了两个东西：
- 导入 foo 模块的类型信息
- 指定运行时依赖 foo 模块

你可以选择只家在类型信息，而没有运行时依赖。在继续之前，你可能想要回顾这本书的[声明空间]()章节。

如果你没有在变量声明空间使用导入的名字，则导入会被完全从生成的 JavaScript 移除。这最好用例子解释。一旦你理解这个，我们将为你展示使用例子。

### 例子1
```ts
import foo = require('foo');
```
将会生成 JavaScript：
```ts
import foo = require('foo');
var bar: foo;
```
这是对的。一个空的文件，因为 foo 没有被使用。
### 例子2
```ts
import foo = require('foo');
var bar: foo;
```
将会生成 JavaScript：
```ts
var bar;
```
这是因为`foo`（或者它的任何属性，比如`ff.bas`）没有被作为变量使用。
### 例子3

```ts
import foo = require('foo');
var bar = foo;
```
将会生成 JavaScript（假设 commonjs）：
```ts
var foo = require('foo');
var bar = foo;
```
这是因为`foo`被用作变量

### 使用例子：懒加载

类型索引需要预先完成。这意味着如果你想要使用来自文件`bar`的`foo`，你需要：
```ts
import foo = require('foo');
var bar: foo.SomeType;
```

然而，你可能想要只在运行时的某些条件下家在文件`foo`。对于这些场景，你应该只在类型声明的时候使用`import`名字，而不是一个变量。这移除任何 TyoeScript 注入的预先运行时时代码。然后手动导入实际模块使用你的模块加载器指定的代码。

作为例子，假设下面基于`commonjs`的代码，我们只加载了模块`'foo'`在某个函数调用：
```ts
import foo = require('foo');

export function loadFoo() {
    // This is lazy loading `foo` and using the original module *only* as a type annotation
    var _foo: typeof foo = require('foo');
    // Now use `_foo` as a variable instead of `foo`.
}
```
一个类型的例子在`amd`（使用 requirejs）可能是：
```ts
import foo = require('foo');

export function loadFoo() {
    // This is lazy loading `foo` and using the original module *only* as a type annotation
    require(['foo'], (_foo: typeof foo) => {
        // Now use `_foo` as a variable instead of `foo`.
    });
}
```

这个模式通常用于：

- 在 web 应用中，你在某些路由下才加载某些 JavaScript
- 在 node 应用中，你只加载某些模块，如果需要加速应用启动。

### 使用例子：打破循环依赖

和懒加载使用场景类似，某些模块加载器（）对循环依赖处理的不是很好。在这类例子，某个方向的懒加载代码和在另一个方向的模块预先加载是很有用的。

### 使用例子：确保导入

有时候你只是为了副作用加载一个文件（比如，模块可能注册它自己到一些库，类似[CodeMirror 插件]()等）。然而，如果你只是执行一个`import/require`，编译的 JavaScript 将不会包含模块内的依赖，你的模块加载器（比如，webpack）可能完整忽略导入。在这类场景，你可以使用一个`ensureImport`变量去确保编译的 JavaScript 接受一个模块的依赖：
```ts
import foo = require('./foo');
import bar = require('./bar');
import bas = require('./bas');
const ensureImport: any =
    foo
    && bar
    && bas;
```