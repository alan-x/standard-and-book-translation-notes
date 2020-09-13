模块

从 ECMAScript 2015 开始，JavaScript 有了模块的概念。TypeScript 共享这个概念。

模块在他们自己的范围内执行，不再全局范围；这意味着变量，函数，类，等，声明在一个模块中，不能被模块之外可见，除非他们使用[导出模式]()明确的导入。相反，为了消费变量，函数，类，接口，等。从一个不同的模块导出，它必须使用[导入形式]()导入。

模块是声明式的；模块之间的关系在文件层级的导入和导出语句指定。

模块使用模块加载器去导入另一个模块。在运行时，模块加载器负责定位和执行模块的所有依赖，在执行它之前。JavaScript 中著名的模块是 [CommonJS]()模块的 NodeJS 加载器和 Web 应用中的[AMD]()模块的[EequireJS]()。

在 TypeScript，比如在 ECMAScript 2015，任何文件包含一个顶级的`import`或者`export`被认为是一个模块。相反，一个文件没有任何顶级`import`或者`export`声明被认为是一个脚本，他的内容被认为是在全局范围内可用（因此，模块也是）。

### 导出

### 导出一个声明


任何声明（比如变量，函数，类，类型别名，或者接口）都可以导出，通过添加`export`关键字

StringValidator.ts

```
export interface StringValidator {
  isAcceptable(s: string): boolean;
}
```

ZipCodeValidator.ts
```
import { StringValidator } from "./StringValidator";

export const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
```


### 导出语句

导出语句是非常灵活的，当导出需要被消费者重命名的时候，因此前面的例子可以充血为：
```
class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```


### 重新导出

通常模块继承其他模块，并且部分导出一些他们的特性。一个重新导出不意味着它是本地的，或者导入一个本地变量。

ParseIntBasedZipCodeValidator.ts
```
export class ParseIntBasedZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5 && parseInt(s).toString() === s;
  }
}

// Export original validator but rename it
export { ZipCodeValidator as RegExpBasedZipCodeValidator } from "./ZipCodeValidator"
```

可选的，一个模块可以包裹一个或者多个模块，并绑定他们所有的导出，使用`export * from "module"`语法。

AllValidators.ts
```
export * from "./StringValidator"; // exports 'StringValidator' interface
export * from "./ZipCodeValidator"; // exports 'ZipCodeValidator' and const 'numberRegexp' class
export * from "./ParseIntBasedZipCodeValidator"; //  exports the 'ParseIntBasedZipCodeValidator' class
// and re-exports 'RegExpBasedZipCodeValidator' as alias
// of the 'ZipCodeValidator' class from 'ZipCodeValidator.ts'
// module.

```

### 导入

导入和导出一样简单。导入一个导出的声明通过使用下面的`import`形式之一：

### 从一个模块导入一个单独的导出

```
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();
```
导入也可以被重命名
```
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

### 导入整个模块到一个单独的变量，并使用它去访问模块导出

```
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```


### 只为副作用导入一个模块

尽管不是推荐的实践，一些模块设置一些全局状态，不能被其他模块使用。这些模块可能没有任何导出，或者消费者不对任何导出感兴趣。为了导入这些模块，使用：
```
import "./my-module.js";

```

### 导入类型

在 TypeScript 3.8 之前，你可以使用`import`导入一个类型。使用 TypeScript 3.8，你可以使用`import`语句导入一个类型，或者使用`import type`。
```
// Re-using the same import
import { APIResponseType } from "./api";

// Explicitly use import type
import type { APIResponseType } from "./api";
```

`import type`总是保证从你的 JavaScript 中移除，并且类似 Babel 的工具可以对你的代码作出更好的假设，通过`isolatedModules`编译器标志。你可以通过[3.8 发行笔记]()了解更多。

### 默认导出

每一个模块可以可选的导出一个`default`导出。默认导出使用关键字`default`标记；并且每一个模块只有一个`default`导出。`default`导出使用一个不同的导入形式导入。

`default`导出是非常灵活的，比如，一个类似 jQuery 的库可能有一个默认的`jQuery`或者`$`导出，在名字`$`或者`jQuery`名字下导入。

JQuery.d.ts
```
declare let $: JQuery;
export default $;
```

App.ts
```
import $ from "jquery";

$("button.continue").html("Next Step...");
```

类和函数声明可以直接作为默认导出。默认导出类和函数声明名字都是可选的。

ZipCodeValidator.ts
```
export default class ZipCodeValidator {
  static numberRegexp = /^[0-9]+$/;
  isAcceptable(s: string) {
    return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
  }
}

```

Test.ts
```
import validator from "./ZipCodeValidator";

let myValidator = new validator();
```

或者
StaticZipCodeValidator.ts
```
const numberRegexp = /^[0-9]+$/;

export default function (s: string) {
  return s.length === 5 && numberRegexp.test(s);
}
```

Test.ts
```
import validate from "./StaticZipCodeValidator";

let strings = ["Hello", "98052", "101"];

// Use function validate
strings.forEach((s) => {
  console.log(`"${s}" ${validate(s) ? "matches" : "does not match"}`);
});

```

默认导出也可以只是值：

OneTwoThree.ts
```
export default "123";

```

Log.ts
```
import num from "./OneTwoThree";

console.log(num); // "123"
```

### 作为 x 导出所有

使用 TypeScript 3.8，你可以使用`export * as ns`作为使用一个名字重新导出其他模块的缩写：
```
export * as utilities from "./utilities";

```

这从其他模块采用所有的依赖并让他成为一个导出的域，你可以像这样导入：
```
import { utilities } from "./index";

```

### 导出 = 和 导入 = require()

CommonJS 和 AMD 通常都有一个`exports`对象的概念，包含从一个模块的所有导出。

他们当然也支持`exports`对象替换为一个自定义单一对象。默认导出意在表现为这个行为的替代；然而，两者是兼容的。TypeScript 支持`export =`去建模传统的 CommonJS 和 AMD 工作流。

`export =`语法为模块导出指定一个单一的对象。这可以是一个类，接口，命名空间，函数，或者枚举。

当使用`export =`导出一个模块，TypeScript 指定`import module = require("module")`必须用于导入一个模块。

ZipCodeValidator.ts
```
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
export = ZipCodeValidator;

```

Test.ts
```ts
import zip = require("./ZipCodeValidator");

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validator = new zip();

// Show whether each string passed each validator
strings.forEach((s) => {
  console.log(
    `"${s}" - ${validator.isAcceptable(s) ? "matches" : "does not match"}`
  );
});
```
### 模块代码生成

取决于在编译的时候指定的模块目标，编译器将会为 NodeJS（[ComonJS]()），require.js([AMD]())，[UMD]()，[SystemJS]()，或者[ECMAScript 2015 原生模块]()（ES6）模块加载系统生成适当的代码。了解更多关于`define`，`require`，和`register`调用在生成的代码做了啥，查阅每一个模块加载器的文档。

这个简单的例子显示名字在引入和导出的时候如何被转化为模块加载代码。

SimpleModule.ts
```
import m = require("mod");
export let t = m.something + 1;
```

AMD / RequireJS SimpleModule.js
```
define(["require", "exports", "./mod"], function (require, exports, mod_1) {
  exports.t = mod_1.something + 1;
});
```

CommonJS / Node SimpleModule.js
```
var mod_1 = require("./mod");
exports.t = mod_1.something + 1;

```

UMD SimpleModule.js
```
(function (factory) {
  if (typeof module === "object" && typeof module.exports === "object") {
    var v = factory(require, exports);
    if (v !== undefined) module.exports = v;
  } else if (typeof define === "function" && define.amd) {
    define(["require", "exports", "./mod"], factory);
  }
})(function (require, exports) {
  var mod_1 = require("./mod");
  exports.t = mod_1.something + 1;
});
```

System SimpleModule.js
```
System.register(["./mod"], function (exports_1) {
  var mod_1;
  var t;
  return {
    setters: [
      function (mod_1_1) {
        mod_1 = mod_1_1;
      },
    ],
    execute: function () {
      exports_1("t", (t = mod_1.something + 1));
    },
  };
});
```
Native ECMAScript 2015 modules SimpleModule.js
```
import { something } from "./mod";
export var t = something + 1;

```

### 简单例子

下面，我们增强了前面使用的校验器实现，只为每一个模块导出一个单独的具名导出

为了编译，你必须在命令行指定一个模块目标。对于 Node.js，使用`--module commonjs`；对于 requirejs，使用`--module amd`。比如：
```
tsc --module commonjs Test.ts

```
被编译之后，每一个模块将编程分离的`.js`文件，就像索引标记一样。编译器将会遵循`import`语句去编译依赖文件。

Validation.ts
```
export interface StringValidator {
  isAcceptable(s: string): boolean;
}
```
LettersOnlyValidator.ts
```
import { StringValidator } from "./Validation";

const lettersRegexp = /^[A-Za-z]+$/;

export class LettersOnlyValidator implements StringValidator {
  isAcceptable(s: string) {
    return lettersRegexp.test(s);
  }
}
```
ZipCodeValidator.ts
```
import { StringValidator } from "./Validation";

const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
```
Test.ts
```
import { StringValidator } from "./Validation";
import { ZipCodeValidator } from "./ZipCodeValidator";
import { LettersOnlyValidator } from "./LettersOnlyValidator";

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
strings.forEach((s) => {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
});
```

### 可选模块加载和其他高级的加载场景

在某些场景，你可能想要只在一些条件下加载一个模块。在 TypeScript 中，我们可以使用下面显示的模式去是想这个，其他高级家加载场景去直接调用模块加载器，而不丢失类型安全。

编译器检测每个模块是否在生成的 JavaScript 被使用。如果一个模块标识符只用于类型声明的一部分，并且没有作为表达式，则这个模块没有`require`。这个为使用的省略是一个好的性能优化，也允许对这些模块的可选加载。

这个模式的核心思想是`import id = require("...")`语句让我们访问模块暴露的类型。模块加载器是动态加载（通过`requirejs`）的，就像下面的`if`块。这利用了引用省略优化，因此模块只在需要的时候加载。为了这个模式可以工作，通过一个`import`定义的 表示只能用在类型定位很重要（从来没有在一个点将会生成到 JavaScript）。

为了维持类型安全，我们可以使用`typeof`关键字。`keyof`关键字在类型定位使用的时候，产生一个值的类型，在这种场景下，是模块的类型。

Dynamic Module Loading in Node.js
```
declare function require(moduleName: string): any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
  let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
  let validator = new ZipCodeValidator();
  if (validator.isAcceptable("...")) {
    /* ... */
  }
}
```

Sample: Dynamic Module Loading in require.js
```
declare function require(
  moduleNames: string[],
  onLoad: (...args: any[]) => void
): void;

import * as Zip from "./ZipCodeValidator";

if (needZipValidation) {
  require(["./ZipCodeValidator"], (ZipCodeValidator: typeof Zip) => {
    let validator = new ZipCodeValidator.ZipCodeValidator();
    if (validator.isAcceptable("...")) {
      /* ... */
    }
  });
}

```

### 和其他 JavaScript 库一起使用
为了描述不是使用 TypeScript 编写的库的外形，我们需要声明库暴露的 API。

当调用没有定一个实现“环境”的声明的时候。通常，这些定义在`.d.ts`文件。如果你对 C/C++ 很熟悉，你可以认为这就像`.h` 文件。来看看一些小例子。

### 环境模块

在 Node.js，大部分任务通过加载一个或者多个模块完成。我们可以在每一个模块的`.d.ts`文件定义顶级导出声明，但是编写大门到一个巨大的`.d.ts`文件更方便。为了做到这个，我们使用一个和环境命名空间类似的结构，但是我们使用`module`关键字和在之后的导入可用模块的引用名。比如：

node.d.ts (simplified excerpt)
```
declare module "url" {
  export interface Url {
    protocol?: string;
    hostname?: string;
    pathname?: string;
  }

  export function parse(
    urlStr: string,
    parseQueryString?,
    slashesDenoteHost?
  ): Url;
}

declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```
现在我们可以`/// <reference> node.d.ts`然后使用`import url = require("url");`或者`import * as URL from "url"`加载模块。

### 缩写环境模块

如果你不想要花时间去编写声明，在使用一个新的模块之前，你可以使用一个缩写声明去快速开始。
declarations.d.ts

```
declare module "hot-new-module";

```
所有从缩写模块的导入将有一个`any`类型
```
import x, { y } from "hot-new-module";
x(y);
```

### 通配符模块声明

一些模块加载器，比如[SystemJS]()和[AMD]()允许非 JavaScript 内容被导入。这些通常用一个前缀或者后缀去只是特定的加载语法。通配符模块加载可以用于覆盖这些场景。

```
declare module "*!text" {
  const content: string;
  export default content;
}
// Some do it the other way around.
declare module "json!*" {
  const value: any;
  export default value;
}
```
现在，你可以引入匹配`*!text`或者`json!*`的东西了。
```
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

### UMD 模块

一些库设计用于多个模块加载器，或者没有模块加载（全局变量）。比如[UMD]模块。这些库可以通过导入或者全局变量访问，比如：
math-lib.d.ts

```
export function isPrime(x: number): boolean;
export as namespace mathLib;
```
库可以用于模块内部的 import：
```
import { isPrime } from "math-lib";
isPrime(2);
mathLib.isPrime(2); // ERROR: can't use the global definition from inside a module

```

也可以用于全局变量，但是只在一个脚本内部。（一个脚本是一个没有 import 或 export 的文件）
```
mathLib.isPrime(2);

```


### 构建模块指南

### 尽可能在最高级导出

你的模块的消费者在使用你导出的东西的时候应该尽可能少的摩擦。添加太多级别的嵌套很麻烦，思考清楚关于怎样构建你的东西。

从你的模块导出一个命名空间是添加太多层级的嵌套的一个例子。尽管命名空间有时候有他们的用处，他们添加了一个额额外的简洁层级，当使用模块的时候。这很快会成为用户的一个痛点，它通常是非必须的。

导出类上的静态方法也有类似的问题 - 类本身添加了一个嵌套层级。除非它增加表达或者为了更清晰的使用方式，考虑简单导出一个帮助函数。

如果你只导出一个单一的类或函数，使用默认导出

就像“在顶级导出”减少你的模块的消费者摩擦，引入一个默认导出也是。如果一个模块的主要目的是寄宿一个指定导出，则你应该考虑将它导出为默认导出。这让导入和实际使用导入更简单。比如：

MyClass.ts
```
export default class SomeType {
  constructor() { ... }
}

```
MyFunc.ts
```
export default function getThing() {
  return "thing";
}

```
Consumer.ts
```
import t from "./MyClass";
import f from "./MyFunc";
let x = new t();
console.log(f());
```

这对消费者很理想。他们可以任意命名的类型（这里是`t`）并且不需要去做任何过分的点操作去寻找你的对象。

如果你正在导出多个对象，将他们全部放在顶级。

MyThings.ts
```
export class SomeType {
  /* ... */
}
export function someFunc() {
  /* ... */
}
```

相反，当导入的时候：

显示列出导入的名称

 
Consumer.ts
```
import { SomeType, someFunc } from "./MyThings";
let x = new SomeType();
let y = someFunc();
```

使用命名空间导入模式，如果你导入大量的东西

MyLargeModule.ts
```
export class Dog { ... }
export class Cat { ... }
export class Tree { ... }
export class Flower { ... }
```

Consumer.ts
```
import * as myLargeModule from "./MyLargeModule.ts";
let x = new myLargeModule.Dog();
```

### 重新导出去继承

经常你需要去扩展一个模块的功能。一个常见的 JS 模式是去扩展原始对象，类似 jQuery 扩展的做法。就像我们前面提到的，不想全局命名空间对象一样合并。推荐的解决方案是不要去操作原始对象，而是导出一个新的实例，提供新的功能。

考虑一个简单的计算器实现定义在模块`Calculator.ts`。模块也导出一个帮助函数去测试计算器功能，通过传递一个列表的输入字符串并在最后输出结果。
Calculator.ts
```ts
export class Calculator {
  private current = 0;
  private memory = 0;
  private operator: string;

  protected processDigit(digit: string, currentValue: number) {
    if (digit >= "0" && digit <= "9") {
      return currentValue * 10 + (digit.charCodeAt(0) - "0".charCodeAt(0));
    }
  }

  protected processOperator(operator: string) {
    if (["+", "-", "*", "/"].indexOf(operator) >= 0) {
      return operator;
    }
  }

  protected evaluateOperator(
    operator: string,
    left: number,
    right: number
  ): number {
    switch (this.operator) {
      case "+":
        return left + right;
      case "-":
        return left - right;
      case "*":
        return left * right;
      case "/":
        return left / right;
    }
  }

  private evaluate() {
    if (this.operator) {
      this.memory = this.evaluateOperator(
        this.operator,
        this.memory,
        this.current
      );
    } else {
      this.memory = this.current;
    }
    this.current = 0;
  }

  public handleChar(char: string) {
    if (char === "=") {
      this.evaluate();
      return;
    } else {
      let value = this.processDigit(char, this.current);
      if (value !== undefined) {
        this.current = value;
        return;
      } else {
        let value = this.processOperator(char);
        if (value !== undefined) {
          this.evaluate();
          this.operator = value;
          return;
        }
      }
    }
    throw new Error(`Unsupported input: '${char}'`);
  }

  public getResult() {
    return this.memory;
  }
}

export function test(c: Calculator, input: string) {
  for (let i = 0; i < input.length; i++) {
    c.handleChar(input[i]);
  }

  console.log(`result of '${input}' is '${c.getResult()}'`);
}

```
这是一个使用导出的`test`函数的计算器的简单测试。

TestCalculator.ts
```
import { Calculator, test } from "./Calculator";

let c = new Calculator();
test(c, "1+2*33/11="); // prints 9

```

现在去扩展这个添加对基于不是 10 的数字的输入的支持，来创建`ProgrammerCalculator.ts`。

ProgrammerCalculator.ts

```
import { Calculator } from "./Calculator";

class ProgrammerCalculator extends Calculator {
  static digits = [
    "0",
    "1",
    "2",
    "3",
    "4",
    "5",
    "6",
    "7",
    "8",
    "9",
    "A",
    "B",
    "C",
    "D",
    "E",
    "F",
  ];

  constructor(public base: number) {
    super();
    const maxBase = ProgrammerCalculator.digits.length;
    if (base <= 0 || base > maxBase) {
      throw new Error(`base has to be within 0 to ${maxBase} inclusive.`);
    }
  }

  protected processDigit(digit: string, currentValue: number) {
    if (ProgrammerCalculator.digits.indexOf(digit) >= 0) {
      return (
        currentValue * this.base + ProgrammerCalculator.digits.indexOf(digit)
      );
    }
  }
}

// Export the new extended calculator as Calculator
export { ProgrammerCalculator as Calculator };

// Also, export the helper function
export { test } from "./Calculator";
```

新的模块`ProgrammerCalculator`导出一个 API 外形类似原始的`Calculator`模块，但是不扩展任何原始模块的对象。这是我们 ProgrammerCalculator 类的测试：

TestProgrammerCalculator.ts

```
import { Calculator, test } from "./ProgrammerCalculator";

let c = new Calculator(2);
test(c, "001+010="); // prints 3
```

不在模块中使用命名空间



### 不要在模块中使用命名空间

当第一次转移到基于模块的组织，一个常见趋势是去包裹导出带一个额外的命名空间层。模块有他们自己的命名空间，只导出模块外部可见的声明。记住这一点，当使用模块的时候，命名空间提供的价值非常小。

在组织前面，命名空间在组合逻辑相关的对象和类型到一个全局范围是非常灵活的。比如，C#，你将在 System.Collections 找到所有的集合类型。通过组织我们的类型到层级的命名空间，换句话说，我们为这些类型的用户提供一个好的“发现”经验。模块，换句话说，已经存在在一个文件系统，必须。你必须解析他们，通过路径和文件名，因此，有一个逻辑组织模式给我们使用，我们可以有一个 /collections/generic/ 文件夹，里面是一个列表的模块

命名空间对于避免全案全局范围的命名冲突很重要。比如，你可能有`My.Application.Customer.AddForm`和`My.Application.Order.AddForm` -- 两种相同名字的类型类型，但是命名空间不同。这，然而，使用模块不是一个问题。在一个模块内，没有合理的理由让两个对象有相同的名字。从消费观点来看，任何给定模块的消费者需要去选择一个名字去应用模块，因此，意外的命名冲突是不可能的。

查阅[命名空间和模块]()了解更多关于模块和命名空间的讨论。

### 红标志

下面的所有都是模块构建的红线标志。重复检查你没有尝试去命名空间你的哇哦不模块。如果有任何一个应用到你的文件：

- 一个只有顶级声明为`export namespace Foo { ... }`的文件（移动`Foo`并移动任何东西到顶级）。

- 多个文件有相同的`export namespace Foo {`在顶级（不要认为将会绑定到一个`Foo`）。