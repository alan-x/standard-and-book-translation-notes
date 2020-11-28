[已校对]
# const

`const`是一个非常受欢迎的增强，由 ES6 / TypeScript 提供，它允许你让变量不可变。从文档和运行时的角度来讲，这形式很好。使用 const，只需要使用`const`替代`var`：
```ts
const foo = 123;
```

> 语法（IMHO）比其他强制用户输入一些类似`let constant foo`的语言更好，比如，变量 + 行为 指定器。

`const`对于可读性和维护性来说是一个好的实践，避免使用魔法字面量，比如：
```ts
// Low readability
if (x > 10) {
}

// Better!
const maxRows = 10;
if (x > maxRows) {
}

```

### const 声明必须初始化

下面是一个编译器错误：
```ts
const foo; // ERROR: const declarations must be initialized
```

### 左侧赋值不能是一个常量

常量是在创建之后是不可变的，因此，如果你尝试去赋值他们到一个新的变量，他会有一个运行时错误：
```ts
const foo = 123;
foo = 456; // ERROR: Left-hand side of an assignment expression cannot be a constant
```

### 块范围

一个`const`是块范围的，就像我们在`let`看到的：
```ts
const foo = 123;
if (true) {
    const foo = 456; // Allowed as its a new variable limited to this `if` block
}
```

### 深度

一个`const`也能和对象字面量一起用，就保护变量引用链接而言：
```ts
const foo = { bar: 123 };
foo = { bar: 456 }; // ERROR : Left hand side of an assignment expression cannot be a constant
```

然而，它依旧允许对象的子属性被操作，就像下面展示的：
```ts
const foo = { bar: 123 };
foo.bar = 456; // Allowed!
console.log(foo); // { bar: 456 }
```

### 首选 const
总是使用`const`，除非你计划去懒初始化一个变量，或者执行一个重新赋值（为这些场景使用`let`）。