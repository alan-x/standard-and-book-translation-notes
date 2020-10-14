# 对象字面量懒实例化

在 JavaScript 代码库中，非常常见的是以下面的方式初始化对象字面量：
```ts
let foo = {};
foo.bar = 123;
foo.bas = "Hello World";
```

当你移动代码到 TypeScript，你将获取下面的错误：
```ts
let foo = {};
foo.bar = 123; // Error: Property 'bar' does not exist on type '{}'
foo.bas = "Hello World"; // Error: Property 'bas' does not exist on type '{}'
```
这是因为从状态`let foo = {}`，TypeScript 推断`foo`的类型(初始化赋值的左手边)是右手边的类型`{}`（也就是，一个没有属性的对象）。因此，它发生了错误，如果你尝试去赋值一个它不知道的属性。

### 理想修复

在 TypeScript 中初始化一个对象的适当方法是在赋值中做这个：
```ts
let foo = {
    bar: 123,
    bas: "Hello World",
};
```
这对代码审阅和代码维护目的也很好。

> 下面描述的快速修复和中间懒初始化模式会错误的忘记初始化一个属性。

### 快速修复

如果你有一个大的 JavaScript 代码库需要升级到 TypeScript，理想修复可能不是一个可实施的方案。在这种场景，你可以小心的使用类型断言去让编译器沉默：
```ts
let foo = {} as any;
foo.bar = 123;
foo.bas = "Hello World";
```

### 中间方案

当然，使用`any`断言是非常坏的，因为某种程度上破坏了 TypeScript 的安全性。中间方案修复是创建一个接口去确保

- 好的文档
- 安全赋值

这显示在下面：
```ts
interface Foo {
    bar: number
    bas: string
}

let foo = {} as Foo;
foo.bar = 123;
foo.bas = "Hello World";
```

这是一个快速例子，显示使用接口可以保护你的真想：
```ts
interface Foo {
    bar: number
    bas: string
}

let foo = {} as Foo;
foo.bar = 123;
foo.bas = "Hello World";

// later in the codebase:
foo.bar = 'Hello Stranger'; // Error: You probably misspelled `bas` as `bar`, cannot assign string to number
```