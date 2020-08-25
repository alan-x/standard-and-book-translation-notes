基本类型

为了程序可用，我们需要能够使用一些简单的数据单元：数字，字符串，结构体，布尔值，和其他类似的。在 TypeScript 中，我们支持你在 JavaScript 中期待的类型一致，还有一个额外的美剧类型抛出，帮助处理。

### Boolean
最基本的数据类型是简单的 true/false 值，JavaScriot 和 TypeScript 叫一个`boolean`值。

```
let isDone: boolean = false;
```

### Number

就像在 JavaScript 中，TypeScropt 中所有的数字都是浮点数或者大整数。这些浮点数类型是`number`，大整数类型是`bigint`。除了十六进制和十进制字面量，TypeScript 也支持 ECMAScript 2015 引入的二进制和八进制字面量

```
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
let big: bigint = 100n;
```

JavaScript 中为网页和服务创建程序的基础是处理文本数据。就像在其他语言，我们使用类型`string`去引用这些文本数据类型。就像 JavaScript，TypeScript 也使用双引号（`"`）或者单引号（`'`）去包裹字符串数据。
```
let color: string = "blue";
color = "red";
```

你当然也可以使用模板字符串，可以分割多行并嵌入字符串。这些字符串被反引号/反单引号（`\``）字符串包裹，并且嵌入的表达式格式为`${ expr }`。

```
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${fullName}.

I'll be ${age + 1} years old next month.`;
```

这和这样声明`setence`一样：
```
let sentence: string =
  "Hello, my name is " +
  fullName +
  ".\n\n" +
  "I'll be " +
  (age + 1) +
  " years old next month.";
```

### Array

TypeScript，就像 JavaScript，允许你操作数组值。数组类型有两种写法。第一种，你使用元素的类型跟随在`[]`去标示元素类型。
```
let list: number[] = [1, 2, 3];
```
第二种生成生成数组类型的方式是`Array<elemType>`：

```
let list: Array<number> = [1, 2, 3];
```

### Tuple

元组类型允许你使用给一个固定数量的元素表达一个数组，他的类型是未知的，但是不需要一样。比如，你可能想要标示一个值为一个`string`和一个`number`构成的对：
```
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
Type 'number' is not assignable to type 'string'.
Type 'string' is not assignable to type 'number'.
```

当使用一个已知的索引访问一个元素，正确的类型被获取：

```
// OK
console.log(x[0].substring(1));

console.log(x[1].substring(1));
Property 'substring' does not exist on type 'number'.
```

访问一个元素超出已知的索引集合会以一个错误失败：
```
x[3] = "world";
Tuple type '[string, number]' of length '2' has no element at index '3'.

console.log(x[5].toString());
Object is possibly 'undefined'.
Tuple type '[string, number]' of length '2' has no element at index '5'.
```

### Enum

从 JavaScript 标准数据类型集合添加的有用的类型是`enum`。在像 C# 的语言中，枚举是给一个数字值集合一个更友好的名字的方式。

```
enum Color {
  Red,
  Green,
  Blue,
}
let c: Color = Color.Green;
```

默认情况下，枚举值从`0`开始为他们的成员标示。你可以改变这个行为，通过手动设置他的其中一个成员的值。比如，我们可以从 1 开始前面的例子，而不是 0：
```
enum Color {
  Red = 1,
  Green,
  Blue,
}
let c: Color = Color.Green;
```
或者，甚至手动设置所有的枚举值：
```
enum Color {
  Red = 1,
  Green = 2,
  Blue = 4,
}
let c: Color = Color.Green;
```

枚举的一个方便特性就是你可以从数字值获取到他在枚举中的名字。比如，如果我们有一个值`2`，但是不确定它在`Color`枚举中的映射，我们可以找出它对应的名字：

```
enum Color {
  Red = 1,
  Green,
  Blue,
}
let colorName: string = Color[2];

// Displays 'Green'
console.log(colorName);
```


### Unknow

当我们编写应用的时候，我们可能需要在我们不知道一个变量的类型的时候去描述它。子鹅血值可能来自动态内容 - 比如，从用户 -或者我们可能故意想要从我们的 API 接受所有值。在这些场景中，我们想要去提供一个类型告诉编译器和未来的读者这个变量可以是任何东西，因此我们给了一个`unknow`类型。

```
let notSure: unknown = 4;
notSure = "maybe a string instead";

// OK, definitely a boolean
notSure = false;
```

如果你有一个变量，他的类型是未知类型，你可以通过执行`typeof`检测让它缩小到特定的，相较于检查，或者更高级的守卫将会在之后的章节讨论：
```typescript
declare const maybe: unknown;
// 'maybe' could be a string, object, boolean, undefined, or other types
const aNumber: number = maybe;
Type 'unknown' is not assignable to type 'number'.

if (maybe === true) {
  // TypeScript knows that maybe is a boolean now
  const aBoolean: boolean = maybe;
  // So, it cannot be a string
  const aString: string = maybe;
Type 'boolean' is not assignable to type 'string'.
}

if (typeof maybe === "string") {
  // TypeScript knows that maybe is a string
  const aString: string = maybe;
  // So, it cannot be a boolean
  const aBoolean: boolean = maybe;
Type 'string' is not assignable to type 'boolean'.
}
```

### Any

在某些场景，并不是所有类型信息都可以获取到，或者它的声明将会不适当的作用。这可能出现在非 TypeScript 编写的代码或者一个三方库。在这些场景，我们可能想要类型检查。为了做到这个，我们标记这些类型为`any`类型：
```
declare function getValue(key: string): any;
// OK, return value of 'getValue' is not checked
const str: string = getValue("myString");
```

` any`类型是和现存 JavaScript 合作的有效方式，允许你在编译期进入或者推出。

不像`unkown`，`any`类型的变量允许你去访问任意属性，甚至是不存在的属性。这些属性包括函数，并且 TypeScript 不会检查他们的存在和类型：
```
let looselyTyped: any = 4;
// OK, ifItExists might exist at runtime
looselyTyped.ifItExists();
// OK, toFixed exists (but the compiler doesn't check)
looselyTyped.toFixed();

let strictlyTyped: unknown = 4;
strictlyTyped.toFixed();
Object is of type 'unknown'.
```

`any`将会冒泡贯穿你的对象：
```
let looselyTyped: any = {};
let d = looselyTyped.a.b.c.d;
//  ^ = let d: any
```

最后，记住，`any`所有的便利来自失去安全类型的成本。类型安全是使用 TypeScript 最主要的动机之一，并且你应该尝试去避免在非必要的石斛使用`any`。

### Void

`void`有点像`any`的对立：完全没有任何类型。你可能常常看到这个，当函数没有返回一个值：
```
function warnUser(): void {
  console.log("This is my warning message");
}
```

声明一个变量为`void`类型没啥用，因为你只能赋值`null`（只有当`--strictNullChecks`没有指定，查阅下一个章节）或者`undefined`给他们：
```
let unusable: void = undefined;
// OK if `--strictNullChecks` is not given
unusable = null;
```

### Null 和 Undefined

在 TypeScript 中，`undefined`和`null`都有他们自己的类型，各自叫做`undefined`和`null`。更像 void，他们本身并不是非常有用。

```
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

默认情况下，`null`和`undefined`是其他所有类型的子类型。这意味着你可以赋值`null`和`undefined`给其他类型，比如`number`。

然而，当使用`--strictNullChecks`标志，`null`和`undefined`只能赋值给`unkown`，`any`和他们各自的类型（一个异常是`undefined`可以赋值给`void`）。这帮助避免很多常见错误。在一些情况下，你想要传递一个`string`或者`null`或者`undefined`，你可以使用联合类型`string | null | undefined`。

联合类型是高级话题，我们将在之后的章节覆盖。

作为一个备注：我们鼓励尽可能使用`--strictNullChecks`，但是为了这个手册的目标，我们将假设它关闭。


### Never
`never`类型标示值的类型从来没有出现过。比如，`never`是一个函数表达式的类型或者一个总是抛出异常的箭头函数或者从来不会返回。变量也可以是`never`类型，当任何类型守卫都不是 true 的时候。

`never`类型是每个类型的子类型，也可以赋值给任何类型；然而，没有一种类型是`never`的子类型，也不能赋值给它（除了`never`本身）。甚至`any`也不能赋值给`never`。

一些函数返回`never`的例子：
```ts
// Function returning never must not have a reachable end point
function error(message: string): never {
  throw new Error(message);
}

// Inferred return type is never
function fail() {
  return error("Something failed");
}

// Function returning never must not have a reachable end point
function infiniteLoop(): never {
  while (true) {}
}
```

### Object
`object`表示非原子类型的类型，比如，任何不是`number`，`string`，`boolean`，`symbol`，`null`，或者`undefined`的类型。

使用`object`类型，像`Object.create`的 API 可以更薄的表示，比如：
```ts
declare function create(o: object | null): void;

// OK
create({ prop: 0 });
create(null);

create(42);
Argument of type '42' is not assignable to parameter of type 'object | null'.
create("string");
Argument of type '"string"' is not assignable to parameter of type 'object | null'.
create(false);
Argument of type 'false' is not assignable to parameter of type 'object | null'.
create(undefined);
Argument of type 'undefined' is not assignable to parameter of type 'object | null'.
```

通常，你不需要使用这个。

### 类型断言

有时候你会在一种场景结束，就是你知道的比 TypeScript 的更多。这通常发生在你知道一些实体的类型比它当前的类型更具体。

类型断言是告诉编译器“相信我，我知道我在做什么”的方式。一个类型断言就像其他语言的类型转化，但是不执行特定检查或者重构数据。它没有运行时暗示，存粹是编译器使用。TypeScript 假设你，编程者，已经执行过任何你需要的特定检查。

类型断言有两种方式

一种是`as`-语法：
```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```
另一种方式是“角括号”语法：
```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```
两种方式是相同的。使用一个超过另一个完成是偏好选择；然而，当使用 TypeScript 编写 JSX 的时候，只有`as`-风格断言允许。


### 关于 let 的一个笔记

到目前未知，你可能已经注意到了，我们使用`let`关键字替换你更熟悉的 JavaScript 的`var`关键字。`let`关键字是一个更新的 JavaScript 概念，TypeScript 让他可用。你可以阅读[变量声明]()手册索引，了解更多关于`let`和`const`相对与`var`修复的问题。

### 关于 Number，String，Boolean，Symbol 和 Object

很容易让人想到，`Number`，`Steing`，`Boolean`，`Symbol`，或者`Object`和前面推荐的消协版本相同。这些类型不引用语言原语，也不应该用作一个类型：
```
function reverse(s: String): String {
  return s.split("").reverse().join("");
}

reverse("hello world");
```

相反，使用`number`，`string`，`boolean`，`object`和`symbol`。

```
function reverse(s: string): string {
  return s.split("").reverse().join("");
}

reverse("hello world");
```