# lib.d.ts

- [lib.d.ts]()
- [使用例子]()
- [内部视角]()
- [修改原生类型]()
- [使用自定义 lib.d.ts]()
- [编译器`target`对 lib.d.ts 的影响]()
- [`lib`选项]()
- [旧的`JavaScript`引擎的 polyfill]()


### `lib.d.ts`

一个特殊的声明文件`lib.d.ts`伴随着每一次 TypeScript 的安装。这个文件包含各种各样存在于 JavaScript 运行时和 DOM 的常见构造器的外部声明。

- 这个文件自动包含在一个 TypeScript 项目的编译上下文
- 这个文件的目的是让你开始编写类型检查的 JavaScript 代码更简单。

你可以通过指定`--noLib`编译器命令行标志（或者`tsconfig.json`中的`"noLib": true`）去吧这个文件排除出编译上下文。

### 使用例子

正如往常，来看看这个文件的使用例子：
```ts
var foo = 123;
var bar = foo.toString();
```
这个代码类型检测得很好，因为`toString`函数定义在`lib.d.ts`，所有 JavaScript 对象都可用。

如果你使用相同的例子代码，但是使用`noLib`选项，你会得到一个类型检测错误：
```ts
var foo = 123;
var bar = foo.toString(); // ERROR: Property 'toString' does not exist on type 'number'.
```

所以，现在你理解`lib.d.ts`的重要性了，它的内容是啥样的？我们在下面检测它。

### `lib.d.ts`内部

阅读全局东西的文档和类型声明最简单的方式是去输入你知道的代码，比如`Math.floor`，然后使用你的 IDE F12（VSCode 对这个支持的很好）。

来看一个变量声明例子，比如，`window`定义如下：
```ts
declare var window: Window;
```

这只是一个简单的`declare var`，后面跟着变量名（这里是`window`）和一个类型声明（这里是`Window`接口）的接口。这些变量通常指向一些全局接口，比如，这里是一个`Window`接口的简单例子：
```ts
interface Window extends EventTarget, WindowTimers, WindowSessionStorage, WindowLocalStorage, WindowConsole, GlobalEventHandlers, IDBEnvironment, WindowBase64 {
    animationStartTime: number;
    applicationCache: ApplicationCache;
    clientInformation: Navigator;
    closed: boolean;
    crypto: Crypto;
    // so on and so forth...
}
```

可以看到，这个接口有大量的类型信息。在没有 TypeScript 的时候，你需要去吧这些记在脑袋里。现在你可以将这些知识放到编译器，使用类似`intellisense`类似的东西简单访问。

有一个为这些全局对象使用接口的好理由。它允许你去添加额外的属性到这些全局对象，而不需要改变`lib.d.ts`，我们将在之后覆盖这些概念。

### 修改原生类型

因为 TypeScript 中的接口是开放的，这以为着你可以添加成员到声明在`lib.d.ts`中的接口，并且 TypeScript 会捡起这些。注意你需要让这些改变在[全局模块]()，让这些接口可以被`lib.d.ts`关联。我们甚至推荐创建一个特定的文件，叫做`global.d.ts`来做这件事。

这里有一些例子场景，我们添加了一些东西到`Window`，`Math`，`Date`：

#### `Window`例子

只是添加东西到`Window`接口，比如：
```ts
interface Window {
    helloWorld(): void;
}
```
这允许你以类型安全的方式去使用它：
```ts
// Add it at runtime
window.helloWorld = () => console.log('hello world');
// Call it
window.helloWorld();
// Misuse it and you get an error:
window.helloWorld('gracius'); // Error: Supplied parameters do not match the signature of the call target
```

#### `Math`例子

全局变量`Math`定义在`lib.d.ts`（子啊一次，使用你的开发工具去导航到定义）：
```ts
/** An intrinsic object that provides basic mathematics functionality and constants. */
declare var Math: Math;
```

比如，变量`Math`是`Math`接口的实例。`Match`接口定义为：
```ts
interface Math {
    E: number;
    LN10: number;
    // others ...
}
```

这意味着如果你想要去添加东西到`Math`全局变量，你只需要添加它到`Math`全局接口，比如，假设[seedrandom 项目]()，添加了一个`seedrandom`函数到全局`Math`对象，这些以简单声明为：
```ts
interface Math {
    seedrandom(seed?: string);
}
```
然后你可以使用它：
```ts
Math.seedrandom();
// or
Math.seedrandom("Any string you want!");
```

#### `Date`例子

如果你看了`lib.d.ts`中`Date`变量的定义，你会发现：
```ts
declare var Date: DateConstructor;
```
接口`DateConstructor`和你之前看到的`Math`和`Window`很像，它包含你可以用的全局变量`Date`的成员，比如`Date.now()`。除了这些成员，它还包含构造器签名，允许你去创建`Date`实例（比如，`new Date()`）。一个`DateConstructor`接口的片段显示在下面：
```ts
interface DateConstructor {
    new (): Date;
    // ... other construct signatures

    now(): number;
    // ... other member functions
}
```

考虑项目[date.js]()。DateJS 添加成员到`Date`全局变量和`Date`实例。因此这个库的 TypeScript 定义可能像（[顺便，社区已经为这个场景编写了]()）：
```ts
/** DateJS Public Static Methods */
interface DateConstructor {
    /** Gets a date that is set to the current date. The time is set to the start of the day (00:00 or 12:00 AM) */
    today(): Date;
    // ... so on and so forth
}

/** DateJS Public Instance Methods */
interface Date {
    /** Adds the specified number of milliseconds to this instance. */
    addMilliseconds(milliseconds: number): Date;
    // ... so on and so forth
}
```

这允许你以下面类型安全的方式做一些东西：
```ts
var today = Date.today();
var todayAfter1second = today.addMilliseconds(1000);
```

#### `string`例子

如果你查看`lib.d.ts`内部的 string，你会发现和我们看到的`Date`（`String`全局变量，`StringConstructor`接口，`String`接口）很像。有一个东西要记得，`String`接口也暗示着字符串字面量，如下代码例子展示：
```ts
interface String {
    endsWith(suffix: string): boolean;
}

String.prototype.endsWith = function(suffix: string): boolean {
    var str: string = this;
    return str && str.indexOf(suffix, str.length - suffix.length) !== -1;
}

console.log('foo bar'.endsWith('bas')); // false
console.log('foo bas'.endsWith('bas')); // true
```

类似的变量和接口也存在于其他东西，像`Number`，`Boolean`，`RegExp`之类的拥有静态和实例成员的。这些接口影响也这些类型的直接实例。

### `string`例子回顾

我们推荐为可维护性创建一个全局的`global.d.ts`，然而，你可以分离他们到一个文件木块的全局命名空间，如果你想要。这通过使用`declare global { /*global namespace here*/ }`做到。比如，前面的例子可以这样：
```ts
// Ensure this is treated as a module.
export {};

declare global {
    interface String {
        endsWith(suffix: string): boolean;
    }
}

String.prototype.endsWith = function(suffix: string): boolean {
    var str: string = this;
    return str && str.indexOf(suffix, str.length - suffix.length) !== -1;
}

console.log('foo bar'.endsWith('bas')); // false
console.log('foo bas'.endsWith('bas')); // true
```

### 使用你自己的自定义 lib.d.ts

就像我们前面提到的，使用`--noLib`布尔值编译器标志会导致 TypeScript 排除自动白汗的`lib.d.ts`。有许多的理由解释为什么这是一个飞铲更有用的特性，这里是常见的几个：

- 你运行在一个自定义 JavaScript 环境，和基于标准浏览器运行时环境有非常大的不同。
- 你想要在你的代码中严格控制全局变量。比如，lib.d.ts 定义`item`作为全局变量，而你不想要这泄露到你的代码。

一旦你排除了默认的`lib.d.ts`，你可以包含一个类似的具名文件到你的编译上下文，TypeScript 将会使用它做类型检测。

> 注意：小心`--noLib`。一旦你在 noLib 环境，如果你选择去分享你的项目给其他人，他们也会强制进入 noLib 环境（或者你的 lib 环境）。甚至更糟糕，如果你将他们的代码引入到你的项目，你肯跟需要移植它到你的库代码。

### 编译器 target 对`lib,d,ts`的影响

设置编译器目标到`ts6`导致`lib,d,ts`包含额外的类似`Promise`之类更现代（es6）的外界声明。编译器目标的魔法效果改变了代码的外界，是一些人期待的，而对于其他人，这是一个问题，因为它合并代码生成和代码环境。

然而，如果你想要更细粒度的控制你的环境，你应该使用`--lib`选项，我们将会在后面讨论。

#### lib 选项

有时候（很多时候）你想要解绑编译目标（生成的 JavaScript 版本）和环境库支持。一个常见例子是`Promise`，比如，现在（在 June 2016）你大部分喜欢`--target es5`但是依旧使用最新的特性，比如`Promise`。为了支持这个，你可以采取对`lib`的明确控制，使用`lib`编译器选项。

> 注意：使用`--lib`从`--target`解绑任何 lib 魔法，使用`lib`编译器选项。

你可以在命令行或者`tsconfig.json`（推荐）提供这个选项：

**命令行**：
```ts
tsc --target es5 --lib dom,es6
```

**tsconfig,json**
```ts
"compilerOptions": {
    "lib": ["dom", "es6"]
}
```

lib 可以如下分类：

- JavaScript 扩展特性：
    - es5
    - es6
    - es2015
    - es7
    - es2016
    - es2017
    - esnext
- 运行时环境
    - dom
    - dom.iterable
    - webworker
    - scripthost
- ESNext 特征选项（甚至比扩展特性都小）
    - es2015.core
    - es2015.collection
    - es2015.generator
    - es2015.iterable
    - es2015.promise
    - es2015.proxy
    - es2015.reflect
    - es2015.symbol
    - es2015.symbol.wellknown
    - es2016.array.include
    - es2017.object
    - es2017.sharedmemory
    - esnext.asynciterable

> 注意：`--lib`选项提供极为精细的控制。因此你可以从扩展中选择一些项目 + 环境分类。如果 --lib 没有指定一个默认的库，它就会被注入：
- 对于 --target es5 => es5, dom, scripthost
- 对于 --target es6 => es6, dom, dom.iterable, scripthost


我个人的推荐：
```ts
"compilerOptions": {
    "target": "es5",
    "lib": ["es6", "dom"]
}
```

ES5 包含 Symbol 的例子：

Symbol API 没有被包含，当 targe 是 es5 的时候。实际上我们接收到一个错误类似：[ts]Cannot find name 'Symbol'。我们可以使用"target":"es5"绑定“lib”在 TypeScript 去提供 API：
```ts
"compilerOptions": {
    "target": "es5",
    "lib": ["es5", "dom", "scripthost", "es2015.symbol"]
}
```

### 旧的 JavaScript 引擎的 Polyfille

> [Egghead PRO 关于这个主题的视频]()

使用现代的`lib`，只有很少的运行时特性可以使用，比如`Map`/`Set`，甚至`Promise`(这个列表将会随着时间改变)。为了使用这些，你需要去使用`core-js`。简单安装：
```ts
npm install core-js --save-dev
```
并添加一个导入到你的应用入口：
```ts
import "core-js";
```

他会为你填充这些运行时特性。