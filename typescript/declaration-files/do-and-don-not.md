### 常见类型

#### Number，String，Boolean，Symbol，和 Object

永远不要使用`Number`，`String`，`Boolean`，`Symbol`，或者`Object`这些引用非原生装箱对象的类型，他们在 JavaScript 几乎没有被适合的使用过。

```ts
/* WRONG */
function reverse(s: String): String;
```

使用`number`，`string`，`boolean`和`symbol`。
```ts
/* WRONG */
function reverse(s: String): String;
```

不使用`Object`，而是使用非原生的`object`类型（TypeScript 2.2 加入的）。


### 泛型

永远不要有不使用类型参数的泛型类型。在[TypeScript FAQ 页面]()了解更多。

### any

不要使用`any`作为类型，除非你正在将 JavaScript 项目升级为 TypeScript。编译器实际上对待`any`为“为这个东西关闭类型检测”。这和在每一次使用这个变量的时候放置一个`@ts-ignore`注释一样。这在你第一次升级一个 JavaScript 项目到 TypeScript 的时候非常有用，你可以为这些还没升级的地方为`any`，但是在一个完整的 TypeScript 项目，你使用这个将会禁止你的程序的任何部分。

为了放置你不知道你想要的是什么类型，或者当你想要接受任何东西的时候，因为你将会透传它而不是和他交互，你可以使用[unknown]()。


### 回调类型

### 回调的返回类型

不要为值将会被忽略的回调使用`any`作为返回值：
```
/* WRONG */
function fn(x: () => any) {
  x();
}
```

使用`void`作为值会被忽略的回调的返回值。
```
/* OK */
function fn(x: () => void) {
  x();
}

```

为啥？使用`void`是更安全的，因为它防止你意外使用`x`的返回值。

```
/* OK */
function fn(x: () => void) {
  x();
}

```

### 回调中的可选的参数

不要在回调中使用可选，除非你真的想要：
```
/* WRONG */
interface Fetcher {
  getObject(done: (data: any, elapsedTime?: number) => void): void;
}
```

这有一个非常特殊的意义：`done`回调可能会使用一个或者两个参数调用。作者可能想要标示回调不关心`elapsedTime`参数，但是不需要去让参数可选来达到这个--提供一个接受更少参数的回调总是合法的，

编写回调参数为非可选：
```
/* OK */
interface Fetcher {
  getObject(done: (data: any, elapsedTime: number) => void): void;
}

```

重载和回调

不要分开编写只有回调不同的重载：
```
/* WRONG */
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number
): void;
```

编写单一的重载，使用最大的属性：
```
/* OK */
declare function beforeAll(
  action: (done: DoneFn) => void,
  timeout?: number
): void;

```
为什么：对于回调，缺少参数总是合法的，没有必要去编写短的重载。提供一个更短的回调允许不正确的类型的函数去传递，因为他们命中第一个重载。

函数重载

### 顺序
不要将更通用的重载放到更明确的重载之前：
```
/* WRONG */
declare function fn(x: any): any;
declare function fn(x: HTMLElement): number;
declare function fn(x: HTMLDivElement): string;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: any, wat?
```

将更通用的签名放在更明确的签名之后：
```
/* WRONG */
declare function fn(x: any): any;
declare function fn(x: HTMLElement): number;
declare function fn(x: HTMLDivElement): string;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: any, wat?
```

为什么：TypeScript 选择第一个命中的重载，当解析函数调用的时候。当一个更早的重载比后一个“更通用”，后一个会被隐藏并且不能被调用。

### 使用可选的参数

不要编写只有最后一个参数不同的重载

```
/* WRONG */
interface Example {
  diff(one: string): number;
  diff(one: string, two: string): number;
  diff(one: string, two: string, three: boolean): number;
}
```
尽可能使用可选的参数：
```
/* OK */
interface Example {
  diff(one: string, two?: string, three?: boolean): number;
}

```
注意这个合并应该只出现在所有的重载有相同的返回值的时候。

为什么：这有两个很重要的原因。

TypeScript 解析签名兼容性是通过检查目标的签名是否可以使用源的参数去调用，额外的参数也是允许的。这个代码，比如，只有当签名正确使用可选的参数编写的时候暴露一个 bug：
```ts
function fn(x: (a: string, b: number, c: number) => void) {}
var x: Example;
// When written with overloads, OK -- used first overload
// When written with optionals, correctly an error
fn(x.diff);
```

第二个原因是当一个消费者使用 TypeScript 的“严格空检测”特性的时候。因为没有指定的参数在 JavaScript 中出现为`undefined`，为可选参数传递一个明确的`undefined`给一个函数也是很常见的，比如，在空严格下也应该可以：
```
var x: Example;
// When written with overloads, incorrectly an error because of passing 'undefined' to 'string'
// When written with optionals, correctly OK
x.diff("something", true ? undefined : "hour");

```

### 使用联合类型
不要编写只有一个参数位不同的重载：
```
/* WRONG */
interface Moment {
  utcOffset(): number;
  utcOffset(b: number): Moment;
  utcOffset(b: string): Moment;
}
```
尽可能使用联合类型：
```ts
/* OK */
interface Moment {
  utcOffset(): number;
  utcOffset(b: number | string): Moment;
}
```

注意我们没有让`b`可选，因为签名的返回值不同。

为什么：这对于“传递”一个值到你的函数的人很重要
```ts
function fn(x: string): void;
function fn(x: number): void;
function fn(x: number | string) {
  // When written with separate overloads, incorrectly an error
  // When written with union types, correctly OK
  return moment().utcOffset(x);
}
```