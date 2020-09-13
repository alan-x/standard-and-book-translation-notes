声明合并

### 介绍

TypeScript 中一些独特的概念在类型层次描述了 JavaScript 对线的外型。TypeScript 独有的概念的一个例子是‘声明合并’。理解这个概念将会让你在使用现有 JavaScript 的时候更有优势。它当然也打开通往更加抽象概念的大门。

这篇文章的目的，“声明合并”意味着编译器合并两个使用相同名字的分离的声明到一个定义。这个合并的定义有两个原始定义的特性。任何数量的声明都可以被合并，不仅仅局限于两个声明。

### 基本概念

在 TypeScript 中，一个声明至少在三个组中创建实体：命名空间，类型，或者值。命名空间创建声明创建一个命名空间，包含可以用点符号访问的名字。类型创建声明做这个：创建一个类型，被身影的外型和包裹的给定名字。最后，值创建声明创建值，在输出的 JavaScript 可见。



| 声明类型 | 命名空间 | 类型 | 值 |
| --- | --- | --- | --- |
| 命名空间 | x | | x |
| 类 | | x | x |
| 枚举 | | x | x |
| 接口 | | x |  |
| 类型别名 | | x |  |
| 函数 | |  | x |
| 变量 | |  | x |


理解每一种声明创建了啥对理解当你执行一个声明合并的时候合并了啥有帮助。

### 合并接口

最简单，并且可能最常见的类型合并是接口合并。在最基本的层次，合并机制合并声明的成员到一个单独的接口，使用相同的名字。

```
interface Box {
  height: number;
  width: number;
}

interface Box {
  scale: number;
}

let box: Box = { height: 5, width: 6, scale: 10 };
```

接口的非函数成员应该是唯一的。如果他们不是唯一的，他们必须有相同的类型。比那一起将会报错，如果接口了一个有相同名字的非函数成员，但是类型不同，将会报错。

对于函数成员，每一个相同名字的函数成员被认为是相同函数的重载描述作为笔记，在接口`A`合并后面的接口`A`的场景中，第二个接口将有比第一个更高的优先级。

也就是，在这个例子中：
```
interface Cloner {
  clone(animal: Animal): Animal;
}

interface Cloner {
  clone(animal: Sheep): Sheep;
}

interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
}
```
三个接口将会合并去创建一个单独的声明如下：
```
interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
  clone(animal: Sheep): Sheep;
  clone(animal: Animal): Animal;
}
```

注意每一个组的元素维护相同的顺序，但是组他们按序合并之后的重载。

这个规则的异常是特定签名。如果签名有一个参数，他的类型是一个单独的字符串字面量类型（比如，不是一个字符串字面量联合），则它将会冒泡到合并的重载列表顶部。

比如，下面的接口将会合并在一起：
```
interface Document {
  createElement(tagName: any): Element;
}
interface Document {
  createElement(tagName: "div"): HTMLDivElement;
  createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
  createElement(tagName: string): HTMLElement;
  createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

`Document`合并的声明结果将会如下：
```ts
interface Document {
  createElement(tagName: "canvas"): HTMLCanvasElement;
  createElement(tagName: "div"): HTMLDivElement;
  createElement(tagName: "span"): HTMLSpanElement;
  createElement(tagName: string): HTMLElement;
  createElement(tagName: any): Element;
}
```

### 合并命名空间

和接口类型，相同名字的命名空间将会合并他们的成员。因为命名空间创建一个命名空间和一个值，我们需要理解怎么合并。

为了合并命名空间，声明在每一个命名空间中导出的类型定义是自我合并的，构成一个单独的命名空间，内部是合并的接口。


为了合并命名空间值，在每一个声明站点，如果命名空间已经存在给定的名字，通过使用存在的命名空间并缇娜家第二个命名空间导出的成员到第一个来扩展。

在这个例子中，`Animals`合并声明：
```
namespace Animals {
  export class Zebra {}
}

namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Dog {}
}

```

等同于：
```ts
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }

  export class Zebra {}
  export class Dog {}
}
```

命名空间合并模型是一个有帮助的开始点，但是我们也需要去理解没有导出的成员发生了什么。没有导出的成员只能在原始的（未合并）命名空间中可用。这意味着在合并之后，从其他地方合并的成员必能看见没有导出的成员。

我们可以在这个例子看的更清楚：
```ts
namespace Animal {
  let haveMuscles = true;

  export function animalsHaveMuscles() {
    return haveMuscles;
  }
}

namespace Animal {
  export function doAnimalsHaveMuscles() {
    return haveMuscles; // Error, because haveMuscles is not accessible here
  }
}
```


因为`haveMuscles`没有导出，只有共享相同的为合并的命名空间的`animalsHaveMuscles`函数可以看见这个符号。`doAnimalsHaveMuscles`函数，尽管也是合并的`Animal`命名空间的一部分，看不见这个未合并的成员。

### 合并带有类，函数，和枚举的命名空间


命名空间足够灵活去合并其他类型的声明。为了这么做，命名空间声明必须跟随在它将会合并的声明之后。结果的声明拥有所有声明类型的的属性。TypeScript 使用这个能力去建模一些 JavaScript 中的模式，就像其他编程语言。

### 合并类的命名空间

这给用户一个描述内部类的方式：
```
class Album {
  label: Album.AlbumLabel;
}
namespace Album {
  export class AlbumLabel {}
}
```

合并的成员的可见行规则和“合并命名空间”章节描述的一样，因此我们必须导出`AlbumLabel`类，让合并的类去看到它。最后的结果是，一个类在另一个类内被管理。你可以使用命名空间去添加更多的静态成员到一个存在的类。
‘

除了内在类的模式，你可能对 JavaScript 创建一个函数然后通过添加属性到函数扩展函数的实践非常熟悉。TypeScript 使用声明合并去构建类型安全的类似的定义。

```
function buildLabel(name: string): string {
  return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
  export let suffix = "";
  export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith"));
```

同样的命名空间可以用于使用静态成员扩展枚举：
```ts
enum Color {
  red = 1,
  green = 2,
  blue = 4,
}

namespace Color {
  export function mixColor(colorName: string) {
    if (colorName == "yellow") {
      return Color.red + Color.green;
    } else if (colorName == "white") {
      return Color.red + Color.green + Color.blue;
    } else if (colorName == "magenta") {
      return Color.red + Color.blue;
    } else if (colorName == "cyan") {
      return Color.green + Color.blue;
    }
  }
}
```

### 不允许的合并

不是所有的合并在 TypeScript 中被允许。当前，类不能和其他类或者变量合并。了解关于类合并的 信息，查阅[TypeScript 中的混入]()章节。


### 模块扩展

尽管 JavaScript 模块不支持合并，你可以修补存在的对象，通过引入然后更新他们。看看一个简单的 Observable 例子：
```
// observable.ts
export class Observable<T> {
  // ... implementation left as an exercise for the reader ...
}

// map.ts
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
  // ... another exercise for the reader
};
```

这在 TypeScript 中也能工作的很好，但是编译器不知道关于`Observable.prototype.map`。你可以使用模块扩展去告诉编译器关于这一点：
```ts
// observable.ts
export class Observable<T> {
  // ... implementation left as an exercise for the reader ...
}

// map.ts
import { Observable } from "./observable";
declare module "./observable" {
  interface Observable<T> {
    map<U>(f: (x: T) => U): Observable<U>;
  }
}
Observable.prototype.map = function (f) {
  // ... another exercise for the reader
};

// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map((x) => x.toFixed());
```

模块名字和`import`/`export`中的模块说明符以相同的方式被解析。查阅[模块]()了解更多信息。然后扩展中的声明 就像如果他们和原始一样被声明在相同的文件。

然而，只有两点要记住：

1. 你不能在扩展中声明新的顶级声明 -- 只匹配存在的声明。
2. 默认导出也可以被扩展，只有具名导出（因为你需要去扩展一个导出，通过它的导出名字，`default`是一个保留字 - 查阅[#14080]()了解更多）

### 全局扩展

你可以从一个模块的内部添加声明到一个全局范围：
```
// observable.ts
export class Observable<T> {
  // ... still no implementation ...
}

declare global {
  interface Array<T> {
    toObservable(): Observable<T>;
  }
}

Array.prototype.toObservable = function () {
  // ...
};
```
全局扩展和模块扩展一样，有相同和行为和限制。