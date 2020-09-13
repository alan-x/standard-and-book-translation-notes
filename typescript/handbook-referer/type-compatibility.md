TypeScript 中的类型兼容基于结构化的子类型。结构化类型是只基于他们成员关联类型的方式。这和常规相反。考虑下列代码：
```
interface Named {
  name: string;
}

class Person {
  name: string;
}

let p: Named;
// OK, because of structural typing
p = new Person();

```

在类似 C# 或者 Java 的常规类型语言中，相同的代码将会是一个错误，因为`Person`类没有明确的描述它自己是`Named`接口的实现者。

TypeScript 的结构化类型系统基于 JavaScript 代码常规编写方式设计。因为 JavaScript 广泛使用匿名对象，比如函数表达式和对象字面量，javaScript 库中结构化类型系统的类型关系的表达更自然，而不是常规的方式。

### 关于稳健的笔记

TypeScript 类型系统允许某种操作在编译时未知的类型安全。当一个类型系统有这个属性，它被称为“不稳健的”。这个地方，TypeScript 允许不稳健的行为是仔细考虑过的，而在这个文档，我们将解释这些发生在哪里，和他们背后的场景意图。

### 开始

TypeScript 的结构化类型系统的基本规则，`x`兼容`y`，如果`y`和`x`有相同的成员，比如：
```
interface Named {
  name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: "Alice", location: "Seattle" };
x = y;
```

为了检测`y`是否可以赋值于`x`，编译器检测`x`的每一个属性，去`y`找到一个兼容的属性。在这种场景中，`y`必须有一个成员叫做`name`，是一个字符。它有，则赋值是被允许的。

相同的规则也用于赋值，当检测函数调用参数的时候：

```
function greet(n: Named) {
  console.log("Hello, " + n.name);
}
greet(y); // OK
```

注意`y`有一个额外的`location`属性，但是这不会创建一个错误。只有目标类型（这个场景中是`Named`）的成员被考虑，当检查兼容性的时候。

这个比较过程递归处理，探索诶一个成员和子成员的类型。

### 对比两个函数

对比原始类型和对象类型相对比较直接，问题是什么类型的函数应该被认为兼容，有点复杂。从一个两个只有参数列表不同的函数的简单例子开始：
```
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

为了检测`x`是否可以赋值给`y`，我们首先查找参数列表。`x`的每一个参数必须有一个对应的兼容的类型的参数在`y`。注意参数的名字不被考虑，只有他们的类型。在这个场景，`x`的每一个参数都有一个对应的可兼容参数在`y`，因此赋值是被允许的。

第二个赋值是一个错误，因为`y`有一个必须的第二个参数，而`x`没有，因此，赋值是不被允许的。

你可能思考为什么我们允许‘废弃’参数，比如例子中的`y=x`。这种赋值被允许的原因是因为忽略额外参数在 JavaScript 中是非常常见的。比如`Array#forEach`提供三个参数到回调函数和：数组元素，他的索引，和包含的数组。然而，提供一个回调只使用第一个参数非常有用：
```
let items = [1, 2, 3];

// Don't force these extra parameters
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach((item) => console.log(item));
```

现在，来看看返回类型是如何被对待的，使用两个只有返回类型不同的函数：
```
let x = () => ({ name: "Alice" });
let y = () => ({ name: "Alice", location: "Seattle" });

x = y; // OK
y = x; // Error, because x() lacks a location property
```

类型系统强制源函数的返回值是目标类型的返回类型的子类型。

### 函数参数不变性

当对比函数参数的类型的时候，如果源参数可以赋值给目标参数，或者相反，则赋值成功。这是不稳定的，因为一个调用者可能可能给一个函数接受一个更指定的类型，但是使用一个更少的指定类型调用函数。在实践中，这些错误很罕见，并且这允许很多常见 JavaScript 模式。一个简单的例子：
```
enum EventType {
  Mouse,
  Keyboard,
}

interface Event {
  timestamp: number;
}
interface MouseEvent extends Event {
  x: number;
  y: number;
}
interface KeyEvent extends Event {
  keyCode: number;
}

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
  /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) =>
  console.log((e as MouseEvent).x + "," + (e as MouseEvent).y)
);
listenEvent(EventType.Mouse, ((e: MouseEvent) =>
  console.log(e.x + "," + e.y)) as (e: Event) => void);

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```

你可以让 TypeScript 报错，当这个发生的时候，通过编译器标志`strictFunctionTypes`。

### 可选参数和剩余参数

当比较函数兼容性的时候，可选的和必须的参数是可交换的。源类型额外的可选参数不是一个错误，目标类型可选参数在原类型没有对应的参数是一个错误。

当一个函数有剩余参数，被认为是一个无限的可选参数。

从类型系统的观点来看，这是不稳定的，但是从运行时的观点来看，可选参数的想法通常不是 well-enforced，因为在这个点传递`undefined`对于大部分函数都是相同的。

动机例子是常见的模式，一个函数接受一个回调并在一些预知（对于开发者）但是不知道参数数量的参数的回调：

```
function invokeLater(args: any[], callback: (...args: any[]) => void) {
  /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ", " + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ", " + y));
```

### 函数重载

当一个函数有重载，源类型中的每一个重载必须匹配，通过在目标类型的兼容签名。这确保慕白哦函数可以在所有相同的场景调用，就像源函数。

### 枚举

枚举和数字是兼容的，数字和枚举是兼容的，不同枚举类型的枚举值被认为是兼容的。比如，
```
enum Status {
  Ready,
  Waiting,
}
enum Color {
  Red,
  Blue,
  Green,
}

let status = Status.Ready;
status = Color.Green; // Error
```

### 类

类工作的和对象字面量类型和接口很像，只有一个例外：他们有一个静态和一个实例类型。当比较两个类类型的对象的石斛，只有实例的成员被比较。静态成员和构造器不影响兼容性。
```
class Animal {
  feet: number;
  constructor(name: string, numFeet: number) {}
}

class Size {
  feet: number;
  constructor(numFeet: number) {}
}

let a: Animal;
let s: Size;

a = s; // OK
s = a; // OK
```

### 类中私有和受保护的成员

一个类中私有和受保护的成员影响他们的兼容性。当一个类的一个实例检查兼容性的时候如果目标类型包含私有成员，则源类型必须也包含一个源自同一类的私有类型。

同样的，这也应用于一个受保护的成员的实例。这允许一个类可以被兼容赋值给他的父类，但是不使用从一个不同继承体系的只有相同外型的类。

### 泛型

因为 TypeScript 是一个结构化类型系统，类型参数只影响结果类型，当作为成员类型的一部分消费的时候。比如，
```
interface Empty<T> {}
let x: Empty<number>;
let y: Empty<string>;

x = y; // OK, because y matches structure of x

```

在前面，`x`和`y`是兼容的，因为他们的结构以一种不同的方式不使用类型参数。改变这个例子，通过添加一个成员到`Empty<T>`显示这是如何工作的：
```
interface NotEmpty<T> {
  data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y; // Error, because x and y are not compatible

```

在这种方式，有类型参数的泛型表现的就像就像一个非泛型。

没有指定类型参数的泛型，兼容性是通过指定所有未指定的类型参数未`any`去检测。这导致类型被检测为兼容的，就像非泛型场景。

比如：
```
let identity = function <T>(x: T): T {
  // ...
};

let reverse = function <U>(y: U): U {
  // ...
};

identity = reverse; // OK, because (x: any) => any matches (y: any) => any

```

### 高级话题



### 子类型和赋值

到目前为止，我们使用“兼容”，这不是定义在语言规范的术语。在 TypeScript。有两种类型的兼容性：子类型和赋值。这些不同只存在在赋值继承子类型兼容规则允许赋值给或者从`any`，和给和从`enum`使用对应的数字值。

语言的不同地方使用两种兼容机制中的一种，取决于场景。为了特俗的目的，类型兼容是通过赋值兼容指定，就算在`implements`和`extends`语句中也是。