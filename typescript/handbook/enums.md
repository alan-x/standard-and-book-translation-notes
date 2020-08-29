枚举

枚举是少量 TypeScript 不是一个 JavaScript 类型级别扩展的特性。

枚举允许一个开发者去定义一个集合的具名常量。使用枚举，让记录意图更简单，或者创建一个集合的不同场景。TypeScript 提供数字化和基于字符串的枚举。


### 数字化枚举

我们首先从数字化枚举开始，如果你从其他语言来，这会更加熟悉。一个枚举可以使用`enum`关键字定义。

```
enum Direction {
  Up = 1,
  Down,
  Left,
  Right
}
```
前面，我们有数字化枚举，`Up`初始化为`1`，后面的所有成员自动从这个点递增。换句话说，`Direction.Up`的值是`1`，`Down`是`2`，`Left`是`3`，`Right`是`4`。

如果我们想要，我们可以完全不初始化：
```
enum Direction {
  Up,
  Down,
  Left,
  Right
}
```

这里，`Up`的值是`0`，`Down`的值是`1`，等。这自增的行为对于我们可能不关心成员值的场景很有用，只关心每一个值和枚举中的其他值是分离的。

使用枚举很简单：只访问任何成员就像枚举本身的一个属性，并使用枚举的名字声明类型：
```
enum UserResponse {
  No = 0,
  Yes = 1
}

function respond(recipient: string, message: UserResponse): void {
  // ...
}

respond("Princess Caroline", UserResponse.Yes);
```

数字化枚举可以混合进[计算和常量成员（看下面）]()。简短的说，枚举要么先不需要被初始化，要么必须在数字化枚举使用数字化常量或者其他常量枚举成员初始化。换句话说，下面是不被允许的：
```
enum E {
  A = getSomeValue(),
  B
Enum member must have initializer.
}
```

### 字符串枚举

字符串枚举是类似的概念，但是有一些不明显的[运行时差异]()，正如下面记录。在一个字符串枚举，每一个成员必须使用字符串字面量常量初始化，或者其他字符串枚举成员。


```
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
```

尽管字符串枚举没有自动递增的行为，字符串枚举的好处是“序列化”。换句话说，如果你需要调试，还需要读取一个数字化枚举的运行时值，值通常是不透明的 - 它对自身的意义传递的不是很好（尽管[反向映射]()通常有帮助），字符串枚举允许你去给一个有意义的和可读的值，当你的代码运行，枚举成员自身有独立的名字。

### 异构枚举值

技术上的枚举值可以混合字符串和数字化成员，但是你为啥这么做不太清楚：
```
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES"
}
```

除非你准备尝试以一种聪明的方式好好利用 JavaScript 的运行时行为，推荐你不要这么做。


### 计算的和常量的成员呢

- 每一个枚举成员都有一个值和他关联，可以是常量，也可以是计算的。一个枚举成员被认为是常量，如果：
```
// E.X is constant:
enum E {
X
}
```

- 它没有一个初始化并且前面的枚举成员是一个数字的常量。在这种场景下，当前的枚举成员将会是前面的枚举成员加一。
```
// All enum members in 'E1' and 'E2' are constant.

enum E1 {
X,
Y,
Z
}

enum E2 {
A = 1,
B,
C
}
```

- 枚举成员使用常量枚举表达式。一个常量枚举表达式是 TypeScript 表达式的子集，可以在编译时完全求值。一个表达式是一个常量枚举表达式，如果：

1. 一个字面量枚举表达式（基本上是一个字符串字面量或者数字字面量）
2. 前面定义的常量枚举成员的索引（可以来自其他枚举）
3. 带括号的常量枚举表达式
4. 应用常量枚举表达式`+`，`-`，`~`一元操作符中的一个
5. `+`，`-`，`*`，`/`，`%`，`<<`，`>>`，`>>>`，`&`，`|`，`^`二进制操作符常量枚举表达式作为操作数。

常量枚举表达式计算为`NaN`或者`Infinity`是一个编译时错误。

在所有掐场景中，枚举值成员被认为是计算的。
```
enum FileAccess {
  // constant members
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
  // computed member
  G = "123".length
}

```

### 联合枚举和枚举成员类型

有一个常量枚举成员的特殊子集不是计算的：字面量枚举成员。一个字面量枚举成员是一个常量枚举成员，没有初始化值，或者值被初始化为：
- 任何字符串字面量（比如，`"foo"`，`"bar"`，`"baz"`）
- 任何数字字面量（比如，`1`，`100`）
- 一个一元减号应用的任何数字字面量（比如，`-1`，`-100`）

当枚举值中的所有成员都有字面量枚举值，一些特殊的语义发挥作用。

首先，枚举成员也是类型！比如，我们可以说某些成员只能有枚举成员的值：
```
enum ShapeKind {
  Circle,
  Square
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}

let c: Circle = {
  kind: ShapeKind.Square,
Type 'ShapeKind.Square' is not assignable to type 'ShapeKind.Circle'.
  radius: 100
};
```

另一个改变是枚举类型他们自己的作用成为一个每一个枚举成员的集合。尽管我们没有讨论[联合类型]()，关于联合枚举，所有你需要知道的是，类型系统可以利用他知道存在在枚举本身的值的集合。因为这个，TypeScript 可以在比较值不正确的时候捕获 bug。

```
enum E {
  Foo,
  Bar
}

function f(x: E) {
  if (x !== E.Foo || x !== E.Bar) {
This condition will always return 'true' since the types 'E.Foo' and 'E.Bar' have no overlap.
    //
  }
}
```
在那个例子，我们首先检查`x`不是`E.Foo`。如果检查成功，则我们的`||`将短路，那么‘if’的内容将会执行。然而，如果检查失败，则`x`只能是`E.Foo`，因此了解它是否等于`E.Bar`没有意义。

### 运行时的枚举值

```
enum E {
  X,
  Y,
  Z
}
```

可以完全传递给函数

```
enum E {
  X,
  Y,
  Z
}

function f(obj: { X: number }) {
  return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
```

### 编译时枚举值

尽管枚举是存在于运行时的真实对象，`keyof`关键字和你预期在常见对象上的工作不同。作为替代，使用`keyof typeof`去获取一个表示所有枚举键类型的字符串表示。

```
enum LogLevel {
  ERROR,
  WARN,
  INFO,
  DEBUG
}

/**
 * This is equivalent to:
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;

function printImportant(key: LogLevelStrings, message: string) {
  const num = LogLevel[key];
  if (num <= LogLevel.WARN) {
    console.log("Log level key is:", key);
    console.log("Log level value is:", num);
    console.log("Log level message is:", message);
  }
}
printImportant("ERROR", "This is a message");
```

### 反向映射

为了创建一个成员有属性名的对象，数字枚举成员也有一个从枚举值到枚举名的反向映射。比如，在这个例子：
```
enum Enum {
  A
}

let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

TypeScript 向下编译为以下 JavaScript：
```
"use strict";
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

在这个生成的代码中，一个枚举被编译为一个对象，存储正向（`name`->`value`）和反向（`value`->`name`）映射。索引其他枚举成员总是表现为属性访问并且从不内联。

记住，字符串枚举成员没有反向映射生成。

const 枚举

在大部分场景中，枚举是非常又想的解决放啊。然而，有时候需要更严格。为了避免在生成代码和访问枚举值付出额外的消耗，可以使用`const`枚举。const 枚举使用`const`修饰符到我们的枚举。
```
const enum Enum {
  A = 1,
  B = A * 2
}
```

const 枚举只能使用常量枚举表达式，而且像常规枚举，在编译时完全移除。const 枚举成员在使用时候完全内联。这是可能的，因为 const 枚举不能有计算属性
```
const enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [
  Directions.Up,
  Directions.Down,
  Directions.Left,
  Directions.Right
];
```
在生成代码中将会成为：
```
"use strict";
let directions = [
    0 /* Up */,
    1 /* Down */,
    2 /* Left */,
    3 /* Right */
];
```

### 环境枚举

环境枚举用来描述已经存在的枚举类型的外形/
```
declare enum Enum {
  A = 1,
  B,
  C = 2
}
```
环境和非环境枚举一个重要的不同是，在常规枚举，没有初始化的成员将会被认为是常量，如果它前面的枚举成员被认为是常量。相反，一个环境（和非常量）枚举成员没有初始化，总是被认为是计算的。