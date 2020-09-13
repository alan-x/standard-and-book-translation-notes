
一个关于术语的笔记：在 TypeScript 1.5 中，术语改变了。“内部模块”现在是“命名空间”。“外部模块”现在是“模块”，为了和[ECMAScript 2015]()对其，（也就是`module X{`和`namespace X {`）相同。

这个文章描述在 TypeScript 中使用命名空间组织你的代码的方式。就像我们在我们的笔记提到的术语那样，“内部模块”现在是“命名空间”。此外，当声明内部模块的时候，任何`module`可以用的地方，`namespace`官架子都可以用于代替。这避免拒绝新用户使用类型的命名的术语去重载他们。

### 第一步

先编写一段程序，我们将贯穿这个页面使用这个例子。我们变了了一个小的集合的字符串验证器，就像你可能为检查一个用户在网页上的输入或者检查一个外部提供的数据文件一样。

### 单个文件中的验证器
```ts
interface StringValidator {
  isAcceptable(s: string): boolean;
}

let lettersRegexp = /^[A-Za-z]+$/;
let numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
  isAcceptable(s: string) {
    return lettersRegexp.test(s);
  }
}

class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    let isMatch = validators[name].isAcceptable(s);
    console.log(`'${s}' ${isMatch ? "matches" : "does not match"} '${name}'.`);
  }
}
```

### 命名空间

随着我们添加更多的验证器，我们希望有一些类型的组织模式，这样我们可以保持对我们类型的跟踪，而不需要担心和其他对象的命名冲突。与其将大量不同名字放在全局命名空间，不如包裹我们的对象到一个命名空间。

在这个例子中，我们将移除所有的验证器相关的额入口到一个命名空间，叫做`Validation`。因为我们想要接口和类可以被命名空间外面访问，我们使用`export`导出他们。相反，变量`lettersRegexp`和`numberRegexp`是实现细节，因此，他们留下了没有导出的和命名空间外部不可访问的代码。在文件末尾的测试代码中，我们现在需要去限定类型的名字，当在外部命名空间使用的时候，比如，`LettersOnlyValidator`。

### 命名空间化的验证器
```
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }

  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;

  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }

  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
}

```

### 跨文件分离

随着应用增长，我们将分离代码到多个文件，让它更简单的维护。

### 多文件命名空间

这里，我们分离我们的`Validation`命名空间到多个文件。就算文件是分离的，他们可以贡献到相同的命名空间可以假设他们所有都定义在一个地方。因为文件之间相互依赖，我们添加引用标注去告诉编译器关于文件之间的关系。我们的测试代码不需要改名。

Validation.ts
```
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
}
```
LettersOnlyValidator.ts
```
/// <reference path="Validation.ts" />
namespace Validation {
  const lettersRegexp = /^[A-Za-z]+$/;
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
}
```
ZipCodeValidator.ts
```
/// <reference path="Validation.ts" />
namespace Validation {
  const numberRegexp = /^[0-9]+$/;
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
```
Test.ts

```
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
}
```

一旦多个文件被调用，我们需要确保编译的代码都被加载。有两个方式做到泽哥。

首先，我们可以使用`--outFile`表去去编译所有的输入文件到一个单独的 JavaScript 输出文件去合并输出。

编译器将会自动排序输出文件，基于存在在文件的索引标志。你可以单独指定每一个文件：
```
tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

作为替代，我们可以使用单独文件编译（默认）去为每一个输入文件发射一个 JavaScript 文件。如果多个 JS 文件生成，我们将需要使用`<script>`标签在我们的网页上去加载每一个生成的文件，以适合的顺序，比如：

MyTestPage.html (excerpt)
```
<script src="Validation.js" type="text/javascript" />
<script src="LettersOnlyValidator.js" type="text/javascript" />
<script src="ZipCodeValidator.js" type="text/javascript" />
<script src="Test.js" type="text/javascript" />

```

### 别名

简化使用命名空间的另一种方式是使用`import q = x.y.z`去创建常用对象的缩写名字。不要和加载模块的`import x = require("name")`语法混淆，这个语法简单创建一个指定标示的别名。你可以使用这些导入（通常称为别名），为任何类型的标识符，包括从模块导入的对象。

```ts
namespace Shapes {
  export namespace Polygons {
    export class Triangle {}
    export class Square {}
  }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'

```
注意我们没有使用`require`关键字；相反，我们直接从我们导入的福报选择一个合格的名字。这和使用`var`很像，但是对于类型和命名空间也有用。重要的是，对于值，`import`是一个从原始标识不同的索引，因此改变为使用`var`将不会影响原始的变量。


### 操作其他 JavaScript 库

为了描述非 TypeScript 编写的库的外型，我们需要去声明库暴露的 API。因为大部分 JavaScript 库只暴露一些顶级对象，命名空间是标示他们的好方式。

我们称呼没有定义一个实现的声明叫做“环境”。通常这些定义在一个`.d.ts`文件，如果你对 C/C++ 非常熟悉，你可以认为这些就是`.h`文件，来看看一些例子。

### 环境命名空间

流行库 D3 定义了他的懂你在全局对象叫做`d3`。因为这个库通过`<script>`标签加载（而不是模块加载），它的声明使用命名空间去定义它的外型。为了让 TypeScript 编译器找到这个外型，我们使用一个环境命名空间声明。比如，我可以开始这样编写：

D3.d.ts (simplified excerpt)

```ts
declare namespace D3 {
  export interface Selectors {
    select: {
      (selector: string): Selection;
      (element: EventTarget): Selection;
    };
  }

  export interface Event {
    x: number;
    y: number;
  }

  export interface Base extends Selectors {
    event: Event;
  }
}

declare var d3: D3.Base;
```
