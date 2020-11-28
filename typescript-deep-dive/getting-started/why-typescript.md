[已校对]
# 为什么是 TypeScript

TypeScript 有两个主要目的：

- 为 JavaScript 提供可选的类型系统
- 从未来的 JavaScript 版本提供计划的特性到当前的 JavaScript 引擎。

这些目标的渴望被下面驱动：

###  TypeScript 类型系统

你可能会思考“为什么要添加类型到 JavaScript？”

类型已经被证明能够增强代码质量和可理解性。大量的团队（Google，Microsoft，Facebook）陆续得出了这个结论。特别是：

- 类型在重构的时候更敏捷。编译器捕捉错误比在运行时发生错误更好。特别是：

- 类型是你拥有的最好形式的文档。函数签名是理论，函数题是实践。

然而，类型有一种不必要的礼节。TypeScript 特别讲究让门槛尽可能低。以下是怎样做到的：

#### 你的 JavaScript 是 TypeScript

TypeScript 为你的 JavaScript 代码提供编译时类型安全。从他的名字并不意外。最好的是类型是完全可选的。你的 JavaScript 代码`.js`文件可以被重命名为一个`.ts`文件，TypeScript 依旧返回有效的和原始 JavaScript 文件相同的`.js`。TypeScript 是故意的和严格的使用可选类型检测的 javaScript 超集

#### TypeScript 可以是明确的

TypeScript 将会尝试尽它所能去推断类型信息，为了给你类型安全，使用最少的生产力消耗，在代码开发期间。比如，在下面的例子中，TypeScript 将会知道下面显示的 foo 是`number` 类型，将会在第二行显示一个错误：
```ts
var foo = 123;
foo = '456'; // Error: cannot assign `string` to `number`

// Is foo a number or a string?
```

这个类型推断的动机很好。如果你做了这个例子中的事情，则，在后面的到吗中，你不能确认`foo`是一个`number`还是一个`string`。这些问题经常在大型多文件代码库出现。我们将在之后深入类型索引。

#### TypeScript 可以是显式的

正如我们前面提到的，TypeScript 将会尽可能安全的推断。然而，你可以使用注解去：

1. 帮助编译器，更重要的是为后面的需要去阅读你的代码的开发者（可能是未来的你）记录东西。
2. 强制编译器看到的是你想要它看到的。你对代码的礼节匹配一个代码的算法分析（编译器完成）。

TypeScript 使用其他可选声明语言的后缀类型声明（比如，ActionScript 和 F#）。
```ts
var foo: number = 123;
```

因此如果你做了一些错的事情，编译器将会报错，比如：
```ts
var foo: number = '123'; // Error: cannot assign a `string` to a `number`
```

我们将会在之后的章节讨论 TypeScript 支持的所有声明语法的详情。

#### TypeScript 是结构化的

在一些语言（特别是名义上类型的）静态类型导致不必要的礼仪，因为尽管你知道代码可以很好的运行，但是语言语义强制你去复制一些东西。这也是为什么像[C# 的 automapper](http://automapper.org/)的东西对 C# 非常必要。在 TypeScript，我们真实想要对 JavaScript 开发者简单，花费最低的认知负荷。考虑下面的例子。函数`iTakePoint2D`将会接受任何包含它希望的所有东西（`x`和`y`）的东西：
```ts
interface Point2D {
    x: number;
    y: number;
}
interface Point3D {
    x: number;
    y: number;
    z: number;
}
var point2D: Point2D = { x: 0, y: 10 }
var point3D: Point3D = { x: 0, y: 10, z: 20 }
function iTakePoint2D(point: Point2D) { /* do something */ }

iTakePoint2D(point2D); // exact match okay
iTakePoint2D(point3D); // extra information okay
iTakePoint2D({ x: 0 }); // Error: missing information `y`
```

#### 类型错误不会阻止 javaScript 生成

为了让你的 JavaScript 代码升级到 TypeScript 代码更简单，就算有编译错误，默认 TypeScript 将会尽可能生成有效的 JavaScript，比如：
```ts
var foo = 123;
foo = '456'; // Error: cannot assign a `string` to a `number`
```
将会生成下面的 js：
```ts
var foo = 123;
foo = '456';
```
所以你可以逐渐升级你的 JavaScript 代码到 TypeScript。这和很多其他语言编译器的工作非常不同，这也是升级为 TypeScript 的另一个原因。


#### 类型可以是环境的

TypeScript 的主要设计目标是去让你在 TypeScript 中安全和简单的使用已存在的 JavaScript 库成为可能。TypeScript 通过声明做到这个。TypeScript 提供一个可伸缩的你想要放置多少影响在你的代码的尺度，放置越多的影响，你将得到越多的类型安全 + 代码交互。注意大部分的 JavaScript 库的定义已经通过[DefinitelyTyped 社区](https://github.com/borisyankov/DefinitelyTyped)为你编写，因此对于大部分目的：

1. 定义文件已经存在
2. 或者至少，已经存在大量定义良好的 TypeScript 声明模版已经存在。

作为一个你将如何编写你自己的声明文件的快速例子，假设有一个平常的[jquery](https://jquery.com/)例子。默认（正如好的 JS 代码期待的）TypeScript 期待你在使用一个变量之前去声明（比如，在某个地方`var`）。
```ts
$('.awesome').show(); // Error: cannot find name `$`
```

作为一个快速修复，你可以告诉 TypeScript 需要叫做`$`的东西：
```ts
declare var $: any;
$('.awesome').show(); // Okay!
```

如果你想要你可以构建这个基础声明并提供更多信息帮助从错误中保护你。
```ts
declare var $: {
    (selector:string): any;
};
$('.awesome').show(); // Okay!
$(123).show(); // Error: selector needs to be a string
```

我们将会在你知道更多关于 TypeScript 的时候讨论为已存在的 JavaScript 创建 TypeScript 声明的细节（比如，像`interface`和`any`之类的东西）。


### 未来的 JavaScript => 现在

TypeScript 为当前的 JavaScript 引擎（只支持 ES5 的那些）提供大量的 ES6 中计划的特性。TypeScript 团队活跃于添加这类特性，这个列表将会随着时间增常，我们将会在它自己的章节覆盖。但是作为样本，这是一个类的例子：
```ts
class Point {
    constructor(public x: number, public y: number) {
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // { x: 10, y: 30 }
```

还有可爱的箭头函数：
```ts
var inc = x => x+1;
```

#### 描述

在这个章节，我们已经提供给你 TypeScript 的动机和设计目的。使用这些方式，我们可以深入挖掘 TypeScript 的细节。