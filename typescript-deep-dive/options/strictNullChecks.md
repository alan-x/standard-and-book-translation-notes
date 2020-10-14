# strictNullChecks

默认`null`和`undefined`可以赋值给 TypeScript 中的所有类型，比如：
```ts
let foo: number = 123;
foo = null; // Okay
foo = undefined; // Okay
```

这是从大部分人编写 JavaScript 的方式中建模出来的。然后，就像所有东西，TypeScript 允许你去明确什么可以和不可以赋值给一个`null`或者`undefined`：

在严格空检测模式，`null`和`undefined`是不同的：

```ts
let foo = undefined;
foo = null; // NOT Okay
```
假设我们有一个`Member`接口：
```ts
interface Member {
  name: string,
  age?: number
}
```
不是每一个`Member`将会提供潭门的名字，因此`age`是可选的属性，意味着`age`可能或者可能不是`undefined`。

`undefined`是所有邪恶的根源。它通常导致运行时错误。很容易写出在运行时抛出`Error`的代码：
```ts
getMember()
  .then(member: Member => {
    const stringifyAge = member.age.toString() // Cannot read property 'toString' of undefined
  })
```

但是在严格空检测模式，错误将会在编译时被捕获：
```ts
getMember()
  .then(member: Member => {
    const stringifyAge = member.age.toString() // Object is possibly 'undefined'
  })
```

### 非空断言操作符

一个新的`!`后缀表达式操作符可能用于断言它在上下文中是非空的和非 undefined，在类型检测无法得出结论的地方。比如：
```ts
// Compiled with --strictNullChecks
function validateEntity(e?: Entity) {
    // Throw exception if e is null or invalid entity
}

function processEntity(e?: Entity) {
    validateEntity(e);
    let a = e.name;  // TS ERROR: e may be null.
    let b = e!.name;  // OKAY. We are asserting that e is non-null.
}
```

> 注意这只是一个断言，就像类型断言，你要自己负责确保值是非空的。一个非空断言就是告诉编译器“我知道这不是空的，因此让我使用它，就像它不是空的”。

### 确定赋值断言操作符

TypeScript 也会抱怨关于类中的属性没有被初始化：
```ts
class C {
  foo: number; // OKAY as assigned in constructor
  bar: string = "hello"; // OKAY as has property initializer
  baz: boolean; // TS ERROR: Property 'baz' has no initializer and is not assigned directly in the constructor.
  constructor() {
    this.foo = 42;
  }
}
```

你可以使用确定赋值断言后缀到属性名字去告诉 TYpeScript 你已经在一些地方初始化了，而不是构造器，比如：
```ts
class C {
  foo!: number;
  // ^
  // Notice this exclamation point!
  // This is the "definite assignment assertion" modifier.

  constructor() {
    this.initialize();
  }
  initialize() {
    this.foo = 0;
  }
}
```

你可以和简单的变量声明使用这个断言：
```ts
let a: number[]; // No assertion
let b!: number[]; // Assert

initialize();

a.push(4); // TS ERROR: variable used before assignment
b.push(4); // OKAY: because of the assertion

function initialize() {
  a = [0, 1, 2, 3];
  b = [0, 1, 2, 3];
}
```

> 就像所有的断言，你告诉编译器相信你。编译器将不会抱怨，就算代码不总是赋值属性。