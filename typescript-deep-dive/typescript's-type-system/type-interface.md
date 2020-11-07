[已校对]
# 类型推断

TypeScript 可以基于一些简单的规则推断（然后检测）变量的类型。因为这些规则很简单，你可以把你的精力放在组织安全/不安全的代码（它非常快的发生在我和我的团队成员身上）。

> 类型流动就是我想象中类型信息的流动

### 变量定义

变量的类型通过定义推断。
```ts
let foo = 123; // foo is a `number`
let bar = "Hello"; // bar is a `string`
foo = bar; // Error: cannot assign `string` to a `number`
```

这是一个类型从右向左流动的例子。

### 函数返回类型

返回类型通过返回语句推断，比如，下面的函数推断返回一个`number`。
```ts
function add(a: number, b: number) {
    return a + b;
}
```
这是一个类型从底部流出的例子。

### 赋值

函数参数/返回值的类型也可以通过赋值推断。比如，这里我们说`foo`是一个`Adder`，这让`number`为`a`和`b`的类型。

```ts
type Adder = (a: number, b: number) => number;
let foo: Adder = (a, b) => a + b;
```

这个事实可以通过下面的代码显示，他会如你所愿抛出一个错误：
```ts
type Adder = (a: number, b: number) => number;
let foo: Adder = (a, b) => {
    a = "hello"; // Error: cannot assign `string` to a `number`
    return a + b;
}
```

这是类型从左到右流动的例子。

相同的赋值风格类型推断也能工作，如果你为一个回调参数创建一个函数。
```ts
type Adder = (a: number, b: number) => number;
function iTakeAnAdder(adder: Adder) {
    return adder(1, 2);
}
iTakeAnAdder((a, b) => {
    // a = "hello"; // Would Error: cannot assign `string` to a `number`
    return a + b;
})
```

### 构造

这些简单的规则也工作在构造的时候（对象字面量创建）。比如，在下面的例子中，`foo`被推断为`{a: number, b:number}`。
```ts
let foo = {
    a: 123,
    b: 456
};
// foo.a = "hello"; // Would Error: cannot assign `string` to a `number`
```

简化数组：
```ts
const bar = [1,2,3];
// bar[0] = "hello"; // Would error: cannot assign `string` to a `number`
```

当然还有任何嵌套：
```ts
let foo = {
    bar: [1, 3, 4]
};
// foo.bar[0] = 'hello'; // Would error: cannot assign `string` to a `number`
```

### 解构

当然，他们和解构也能一起用，包括对象：
```ts
let foo = {
    a: 123,
    b: 456
};
let {a} = foo;
// a = "hello"; // Would Error: cannot assign `string` to a `number`
```
和数组：
```ts
const bar = [1, 2];
let [a, b] = bar;
// a = "hello"; // Would Error: cannot assign `string` to a `number`
```
如果函数参数可以被推断，也可以解构属性。比如，这里我们解构参数到他的`a`/`b`成员。
```ts
type Adder = (numbers: { a: number, b: number }) => number;
function iTakeAnAdder(adder: Adder) {
    return adder({ a: 1, b: 2 });
}
iTakeAnAdder(({a, b}) => { // Types of `a` and `b` are inferred
    // a = "hello"; // Would Error: cannot assign `string` to a `number`
    return a + b;
})
```

### 类型守卫

我们已经知道[类型守卫](https://basarat.gitbook.io/typescript/type-system/typeguard)如何帮助我们改变和向下转型类型（特别是在联合的场景）。类型断言是一个块内变量另一种形式的类型推断。

### 警告

#### 小心参数

类型不能流进函数参数，如果他不能从赋值被推断。比如，在下面的例子，编译器知道`foo`的类型，因此它不能推断`a`或`b`的类型。
```ts
const foo = (a,b) => { /* do something */ };
```

然而，如果`foo`有函数参数类型，则可以被推断（下面的例子`a`，`b`都被推断为`number`类型）。
```ts
type TwoNumberFunction = (a: number, b: number) => void;
const foo: TwoNumberFunction = (a, b) => { /* do something */ };
```

#### 小心返回

尽管 TypeScript 通常可以推断函数的返回值，他可能不是你期待的。比如，这里函数`foo`有一个返回值`any`。
```ts
function foo(a: number, b: number) {
    return a + addOne(b);
}
// Some external function in a library someone wrote in JavaScript
function addOne(c) {
    return c + 1;
}
```

这是因为返回值通过简单的类型定义`addOne`暗示（`c`是`any`，因此`addOne`的返回值是`any`，因此`foo`的返回值是`any`）。

> 我发现最简单的总是显式指定函数返回值。之后，这些声明是理论，而函数体是证据。

这是一个可以想象的其他例子，但是好消息是有一个编译器标志可以捕获这个 bug。

### noImplictAny

标志`noImplicitAny`指示编译器如果它无法推断一个变量的类型（因此值剋有有一个暗示的`any`类型）就报错。然后你可以：
- 可以说你希望`type`被明确添加一个`:any`类型声明
- 通过添加一些更正确的声明帮助编译器。


`