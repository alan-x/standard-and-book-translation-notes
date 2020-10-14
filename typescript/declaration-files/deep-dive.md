### 声明文件理论：深入

构造模块去给出你想要的 API 可能很难。比如，你可能想要一个模块可以使用`new`或者不使用`new`去被调用使用不同的类型，在一个层级中有一系列具名导出，同时也有一些属性在模块对象上。

通过阅读这个指南，你将有一个工具去编写复杂声明文件，暴露一个友好的 API 表层。这个指南聚焦于模块（或者 UMD）库，因为这里的选择更不同。

### 关键概念

你可以完全理解怎样创建任何声明的外型，通过理解 TypeScript 如何工作的关键概念。。

### 类型

如果你正在阅读这个指南，你可能粗略的了解 TypeScript 中的类型是啥。为了更加明确，一个类型是：

- 一个类型别名声明（`type sn = number | string;`）
- 一个接口声明（`interface I { x: number[]; }`）
- 一个类声明（`class C { }`）
- 一个枚举声明（`enum E { A, B, C }`）
- 一个`import`声明引用一个类型

这些声明的每一种声明形式创建一个新的类型名字/

#### 值

随着类型的使用，你可能已经理解值是什么。值是运行时名字，我们可以在表达式中引用。比如`let x = 5;`创建一个值叫做`x`。

再一次，明确的说，下面的东西创建值：

- `let`，`const`，`var`声明
- 一个包含值的`namespace`或者`module`声明
- 一个`enum`声明
- 一个`class`声明
- 一个引用一个值的`import`声明
- 一个`function`声明

#### 命名空间

类型可以存在于命名空间。比如，如果我们有一个声明`let x: A.B.C`，我们说`C`来自`A.B`命名空间。

这种区别很微妙也很重要 - 这里，`A.B`不需要一个类型或者值。


### 简单组合：一个名字，多种意义

给定一个名字`A`，我们可能发现`A`的三种意义：一个类型，一个值，或者一个命名空间。名字被如何交互取决于它使用的上下文。比如，在声明`let m: A.A = A;`中，`A`首先用作一个命名空间，然后是一个类型名字，然后是一个值。这意味着可能指向完全不同的声明。

这可能看起来很迷惑，但是实际上非常方便，只要我们不过分的重载。来看看关于这个绑定行为有用的方面。

#### 内置绑定

敏锐的读者将会注意到，比如，`class`出现在类型和值列表。声明`class C { }`创建两个东西：一个类型`C`，引用类的实例，和一个值`C`，引用类的构造器函数。枚举声明行为类似。

#### 用户绑定

假设我们编写了一个模块文件`foo.d.ts`:
```
export var SomeVar: { a: SomeType };
export interface SomeType {
  count: number;
}
```

然后消费它：
```
import * as foo from "./foo";
let x: foo.SomeType = foo.SomeVar.a;
console.log(x.count);
```

这工作的很好，但是我们可能想象到`SomeType`和`SomeVar`非常相近让你希望他们有相同的名字。我们可以使用绑定咋相同的名字`Bar`去标志这些不同的对象：
```ts
export var Bar: { a: Bar };
export interface Bar {
  count: number;
}
```

这存在一个非常好的机会去结构消费代码：
```ts
import { Bar } from "./foo";
let x: Bar = Bar.a;
console.log(x.count);
```

再一次，我们使用`Bar`作为类型和值。注意我们不需要声明`Bar`的值为`Bar`类型 -- 他们是独立的。


### 高级组合

一些类型的声明可以跨越多个声明绑定。比如，`class C {}`和`interface C {}`，可以共存，并贡献属性到`C`类型。

这是合法的，只要它不创建冲突。一个通用的规则是值总是和相同名字的其他值冲突，除非他们声明为`namespace`，类型将会冲突，如果他们声明一个类型别名（`type s = string`），命名空间永远不会冲突。

现在看看这怎么用。

#### 使用一个接口添加
我们可以使用其他的`interface`声明添加额外的成员到一个`interface`:
```
interface Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```
这对类也有效：
```
class Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

注意不能使用一个接口添加一个类型别名（`type s = string;`）

#### 使用命名空间添加

一个`namespace`声明可以以任何方式用于添加一个新的类型，值，和命名空间，不会造成冲突。

比如，我们可以添加一个惊天成员到一个类：
```
class C {}
// ... elsewhere ...
namespace C {
  export let x: number;
}
let y = C.x; // OK
```

注意在这个例子中，我们添加一个值到`C`的静态侧（他的构造器函数）。这是因为我们添加一个值，所有值的容器是其他值（类型被命名空间包含，命名空间被其他命名空间包含）。

我们可以添加一个命名空间类型到一个类：
```
class C {}
// ... elsewhere ...
namespace C {
  export interface D {}
}
let y: C.D; // OK
```

在这个例子，直到我们写了`namspace`声明之后，才有了命名空间`C`。`C`命名空间没有和类创建的类型`C`冲突。

最后，使用`namspace`声明我们可以执行很多不同的合并。这是一个特别真实的例子，但是显示了很多有趣的行为：
```ts
namespace X {
  export interface Y {}
  export class Z {}
}

// ... elsewhere ...
namespace X {
  export var Y: number;
  export namespace Z {
    export class C {}
  }
}
type X = string;
```

在这个例子中，第一块创建了下面的名称意义：

- 一个值`X`（因为`namespace`声明包含一个值，`Z`）

- 一个命名空间`X`(因为`namespace`声明包含一个类型`Y`)

- `X`命名空间中的类型`Y`

- `X`命名空间中的`Z`（类的实例外型）

- `X`值的一个属性`Z`。

第二块创建了下面的名称含义：

- `X`值的一个属性`Y`
- 一个命名空间`Z`
- `X`值的一个属性值`Z`
- `X.Z`命名空间的一个类型`C`
- `X.Z`值的一个属性值`C`
- 类型`X`


