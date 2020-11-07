[已校对]
# 移动类型

TypeScript 的类型系统威力非常强大，允许以其他语言不可能的方式移动和分割类型。

这是因为 TypeScript 设计允许你去和类似 JavaScript 之类的高动态语言无缝工作，这里我们覆盖了 TypeScript 中移动类型相关的一些陷阱。

这里的核心东西：你改变了一个东西，而其他的所有东西会自动更新，如果一些东西被破坏，你会得到一个漂亮的错误，比如一个设计良好的约束系统。

### 赋值类型 + 值

如果你想要移动一个类，你可能尝试去这么做：
```ts
class Foo { }
var Bar = Foo;
var bar: Bar; // ERROR: cannot find name 'Bar'
```

这是一个错误，因为`var`只赋值了`Foo`到变量声明空间，并且你不能使用`Bar`作为类型声明。适合的方式是使用`import`关键字。注意你只能以这种方式使用`import`关键字，如果你使用命名空间或者模块（更多在后面）：
```ts
namespace importing {
    export class Foo { }
}

import Bar = importing.Foo;
var bar: Bar; // Okay
```

这个`import`陷阱只为即是类型和又是变量的定西有效。

### 捕获变量的类型

使用`typeof`操作符，实际你可以使用一个变量在一个类型声明。这允许你去告诉编译器一个变量和其他有相同的类型。这里是一个类型显示在下面：
```ts
var foo = 123;
var bar: typeof foo; // `bar` has the same type as `foo` (here `number`)
bar = 456; // Okay
bar = '789'; // ERROR: Type `string` is not `assignable` to type `number`
```

### 捕获一个类成员的类型

你可以遍历任何非空对象类型去约束一个属性的类型：
```ts
class Foo {
  foo: number; // some member whose type we want to capture
}

let bar: Foo['foo']; // `bar` has type `number`
```
或者，和捕获变量类型类似，你只是为类型捕获目的声明一个变量：
```ts
// Purely to capture type
declare let _foo: Foo;

// Same as before
let bar: typeof _foo.foo; // `bar` has type `number`
```

### 捕获魔法字符串的类型

很多 JavaScript 库和框架都是用原生 JavaScript 字符串。你可以使用`const`变量去捕获他们的类型，比如：
```ts
// Capture both the *type* _and_ *value* of magic string:
const foo = "Hello World";

// Use the captured type:
let bar: typeof foo;

// bar can only ever be assigned to `Hello World`
bar = "Hello World"; // Okay!
bar = "anything else "; // Error!
```

在这个例子中，`bar`有字面量类型`"Hello World"`。我们会在[字面量类型章节](https://basarat.gitbook.io/typescript/type-system/literal-types)覆盖更多。

### 捕获键的名字

`keyof`操作符让你捕获类型的关键名字。比如，你可以使用它去捕获变量的关键字名字，通过使用`typeof`第一次扫描：
```ts
const colors = {
  red: 'reddish',
  blue: 'bluish'
}
type Colors = keyof typeof colors;

let color: Colors; // same as let color: "red" | "blue"
color = 'red'; // okay
color = 'blue'; // okay
color = 'anythingElse'; // Error: Type '"anythingElse"' is not assignable to type '"red" | "blue"'
```

这允许你去有类似字符串枚举 —— 常量非常简单，就像你在前面的例子看到的。