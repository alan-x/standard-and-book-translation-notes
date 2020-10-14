# 只读

### readonly

TypeScript 的类型系统允许你去标记独立的属性为`readonly`。这允许你去以函数式的方式工作（不期待的可变性是很坏的）。
```ts
function foo(config: {
    readonly bar: number,
    readonly bas: number
}) {
    // ..
}

let config = { bar: 123, bas: 123 };
foo(config);
// You can be sure that `config` isn't changed 🌹
```

当然你也可以在`interface`和`type`使用`readonly`定义也行，比如：
```ts
type Foo = {
    readonly bar: number;
    readonly bas: number;
}

// Initialization is okay
let foo: Foo = { bar: 123, bas: 456 };

// Mutation is not
foo.bar = 456; // Error: Left-hand side of assignment expression cannot be a constant or a read-only property
```
你甚至可以声明一个类属性是`readonly`。你可以在声明的时候初始化他们，或者在构造器中，如下显示：
```ts
class Foo {
    readonly bar = 1; // OK
    readonly baz: string;
    constructor() {
        this.baz = "hello"; // OK
    }
}
```
### Readonly

有一个类型`Readonly`，接受一个类型`T`，并标志它的所有属性为`readonly`，使用映射类型。这是实际中使用它的一个 demo：
```ts
type Foo = {
  bar: number;
  bas: number;
}

type FooReadonly = Readonly<Foo>; 

let foo: Foo = {bar: 123, bas: 456};
let fooReadonly: FooReadonly = {bar: 123, bas: 456};

foo.bar = 456; // Okay
fooReadonly.bar = 456; // ERROR: bar is readonly
```

### 各种使用场景

#### ReactJS

一个喜欢不可变的库是 ReactJS，你可以标记你的`Props`和`State`为不可变，比如：
```ts
interface Props {
    readonly foo: number;
}
interface State {
    readonly bar: number;
}
export class Something extends React.Component<Props,State> {
  someMethod() {
    // You can rest assured no one is going to do
    this.props.foo = 123; // ERROR: (props are immutable)
    this.state.baz = 456; // ERROR: (one should use this.setState)  
  }
}
```

然而，你不需要这么做，因为 React 的类型定义已经标记这些为`readonly`（通过前面提到的内部的包裹的使用`Readonly`传递进泛型的）。

```ts
export class Something extends React.Component<{ foo: number }, { baz: number }> {
  // You can rest assured no one is going to do
  someMethod() {
    this.props.foo = 123; // ERROR: (props are immutable)
    this.state.baz = 456; // ERROR: (one should use this.setState)  
  }
}
```

#### 连续不可变

你可以标记索引签名为 readonly：
```ts
/**
 * Declaration
 */
interface Foo {
    readonly[x: number]: number;
}

/**
 * Usage
 */
let foo: Foo = { 0: 123, 2: 345 };
console.log(foo[0]);   // Okay (reading)
foo[0] = 456;          // Error (mutating): Readonly
```

如果你想要以不可变的风格使用原生 JavaScript 数字。实际上，TypeScript 使用一个`ReadonlyArray<T>`接口允许你这么做：
```ts
let foo: ReadonlyArray<number> = [1, 2, 3];
console.log(foo[0]);   // Okay
foo.push(4);           // Error: `push` does not exist on ReadonlyArray as it mutates the array
foo = foo.concat([4]); // Okay: create a copy
```

#### 自动推断

在某些场景，编译器可以自动推断一个指定的值为 readonly，比如，在一个雷内，如果你有一个属性只有 getter 但是没有 setter，它被假设为 readonly，比如：
```ts
class Person {
    firstName: string = "John";
    lastName: string = "Doe";
    get fullName() {
        return this.firstName + this.lastName;
    }
}

const person = new Person();
console.log(person.fullName); // John Doe
person.fullName = "Dear Reader"; // Error! fullName is readonly
```

### 和 const 的不同

`const`

1. 是为了变量索引
2. 变量不能被重新赋值给其他任何东西

`readonly`是

1. 为了属性
2. 属性可以被修改，因为别名

例子1:
```ts
const foo = 123; // variable reference
var bar: {
    readonly bar: number; // for property
}
```
例子2:
```ts
let foo: {
    readonly bar: number;
} = {
        bar: 123
    };

function iMutateFoo(foo: { bar: number }) {
    foo.bar = 456;
}

iMutateFoo(foo); // The foo argument is aliased by the foo parameter
console.log(foo.bar); // 456!
```

基本上，`readonly`确保一个属性不能被我修改，但是如果你将它赋值给其他人，那无法保证他们（为了类型兼容原因）可以修改他。当然，如果`iMutateFoo`说他们不操作`foo.bar`，编译器将正确标记它我为一个错误，如下所示：
```ts
interface Foo {
    readonly bar: number;
}
let foo: Foo = {
    bar: 123
};

function iTakeFoo(foo: Foo) {
    foo.bar = 456; // Error! bar is readonly
}

iTakeFoo(foo); // The foo argument is aliased by the foo parameter
```