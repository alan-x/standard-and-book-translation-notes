### 全局库

全局库是可以在全局范围访问的库（比如，没有使用任何形式的`import`）。很多库简单暴露一个或者多个全局变量使用。比如，如果你使用[jQuery]()，`$`变量可以可以通过简单的引用它使用：
```ts
$(() => {
  console.log("hello!");
});
```

你通常可以在文档的指南中看到一个全局库如何在 HTML 脚本标签中使用：
```
<script src="http://a.great.cdn.for/someLib.js"></script>

```

 如今大部分流行的全局-可访问库实际上使用 UMD 编写（查看下面）。UML 库文档和全局库文档很难区分。在编写全局声明文件之前，确保库不是 UMD。

### 从代码定义一个全局库

全局库代码通常非常简单。一个全局的“Hello，world”库可能看起来像这样：
```ts
function createGreeting(s) {
  return "Hello, " + s;
}
```

或者像这样：
```ts
window.createGreeting = function (s) {
  return "Hello, " + s;
};
```
当查看一个全局库代码的时候，呢通常会看到：

- 顶级`var`语句或者`function`声明
- 一个或者多个`window.someName`赋值
- 假设像`document`或者`window`之类的 DOM 原生存在

你不会看到：

- 检查，使用，类似`require`或者`define`的模块加载器
- CommonJS/Node.js 风格的导入形式`var fs = require("fs");`
- 描述如何`require`或者导入一个库的文档。


### 全局库的例子

因为通常很简单就能转化一个全局库到一个 UMD 库，很少流行的库依旧以全局风格编写。然而，库很小或者需要 DOM（或者没有依赖） 的可能依旧是全局的。

### 全局库模板

你可以看看下面的 DTS 例子：
```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ If this library is callable (e.g. can be invoked as myLib(3)),
 *~ include those call signatures here.
 *~ Otherwise, delete this section.
 */
declare function myLib(a: string): string;
declare function myLib(a: number): number;

/*~ If you want the name of this library to be a valid type name,
 *~ you can do so here.
 *~
 *~ For example, this allows us to write 'var x: myLib';
 *~ Be sure this actually makes sense! If it doesn't, just
 *~ delete this declaration and add types inside the namespace below.
 */
interface myLib {
  name: string;
  length: number;
  extras?: string[];
}

/*~ If your library has properties exposed on a global variable,
 *~ place them here.
 *~ You should also place types (interfaces and type alias) here.
 */
declare namespace myLib {
  //~ We can write 'myLib.timeout = 50;'
  let timeout: number;

  //~ We can access 'myLib.version', but not change it
  const version: string;

  //~ There's some class we can create via 'let c = new myLib.Cat(42)'
  //~ Or reference e.g. 'function f(c: myLib.Cat) { ... }
  class Cat {
    constructor(n: number);

    //~ We can read 'c.age' from a 'Cat' instance
    readonly age: number;

    //~ We can invoke 'c.purr()' from a 'Cat' instance
    purr(): void;
  }

  //~ We can declare a variable as
  //~   'var s: myLib.CatSettings = { weight: 5, name: "Maru" };'
  interface CatSettings {
    weight: number;
    name: string;
    tailLength?: number;
  }

  //~ We can write 'const v: myLib.VetID = 42;'
  //~  or 'const v: myLib.VetID = "bob";'
  type VetID = string | number;

  //~ We can invoke 'myLib.checkCat(c)' or 'myLib.checkCat(c, v);'
  function checkCat(c: Cat, s?: VetID);
}
```