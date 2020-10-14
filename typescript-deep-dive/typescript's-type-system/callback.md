# 回调

你可以注解回调作为一个类型或者接口的一部分，如下：
```ts
interface ReturnString {
  (): string
}
```

这类接口的实例将会是一个函数，然后返回一个字符串，比如：
```ts
declare const foo: ReturnString;
const bar = foo(); // bar is inferred as a string
```

### 明显的例子

当然，这类回调声明可以按需指定任何参数/可选参数/剩余参数。比如，这是一个复杂例子：
```ts
interface Complex {
  (foo: string, bar?: number, ...others: boolean[]): number;
}
```

一个接口可以提供多种回调声明去指定函数重载。比如：
```ts
interface Overloaded {
    (foo: string): string
    (foo: number): number
}

// example implementation
function stringOrNumber(foo: number): number;
function stringOrNumber(foo: string): string;
function stringOrNumber(foo: any): any {
    if (typeof foo === 'number') {
        return foo * foo;
    } else if (typeof foo === 'string') {
        return `hello ${foo}`;
    }
}

const overloaded: Overloaded = stringOrNumber;

// example usage
const str = overloaded(''); // type of `str` is inferred as `string`
const num = overloaded(123); // type of `num` is inferred as `number`
```

当然，就像任何接口体，你可以使用可调用接口体作为一个变量的类型声明。比如：
```ts
const overloaded: {
  (foo: string): string
  (foo: number): number
} = (foo: any) => foo;
```

### 箭头语法

为了让指定可调用签名简单，TypeScript 也允许箭头类型声明。比如，一个函数接受一个`number`并返回一个`string`可以声明为：
```ts
const simple: (foo: number) => string
    = (foo) => foo.toString();
```

> 只有箭头语法才有的限制：你不能指定重载。对于重载，你必须使用完全体语法。


### 可构造调用的

可调用只是特殊的可以使用前置`new`的可调用类型声明。它简单的意味着你需要使用`new`调用，比如：
```ts
interface CallMeWithNewToGetString {
  new(): string
}
// Usage
declare const Foo: CallMeWithNewToGetString;
const bar = new Foo(); // bar is inferred to be of type string
```