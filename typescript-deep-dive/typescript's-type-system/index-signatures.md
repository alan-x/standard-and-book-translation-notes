[已校对]
# 索引签名

JavaScript（因此 TypeScript） 中的一个`Object`可以使用一个字符串去访问持有任何其他 JavaScript 对象。

这是一个快速例子：
```ts
let foo: any = {};
foo['Hello'] = 'World';
console.log(foo['Hello']); // World
```
我们在键`"Hello"`下存储一个字符串`"World"`。记住我们说他可以存储任何 JavaScript 对象，因此存储一个类实例去显示这个概念：
```ts
class Foo {
  constructor(public message: string){};
  log(){
    console.log(this.message)
  }
}

let foo: any = {};
foo['Hello'] = new Foo('World');
foo['Hello'].log(); // World
```

记住我们说他可以使用一个字符串访问。如果你传递任何其他对象到索引签名，JavaScript 实际上在它之上调用`.toString`，在获取结果之前。这显示在下面：
```ts
let obj = {
  toString(){
    console.log('toString called')
    return 'Hello'
  }
}

let foo: any = {};
foo[obj] = 'World'; // toString called
console.log(foo[obj]); // toString called, World
console.log(foo['Hello']); // World
```

注意，`toString`将会被调用当`obj`应在索引为止的时候。

数组有点不同，对于`number`索引，JavaScript VM 将尝试优化（这取决于一些东西，比如他实际是一个数组或者构造他的项存储是否匹配）。因此`number`应该被认为是一个有效的访问器，在他自己的右边（和`string`区别）。这是一个简单的数组例子：
```ts
let foo = ['World'];
console.log(foo[0]); // World
```

因此这就是 JavaScript。现在来看看 TypeScript 如何优雅处理这个概念。

### TypeScript 索引签名

首先，因为 JavaScript 在任何对象索引签名都隐式调用`toString`，TypeScript 将会给你一个错误去防止新手射中自己的脚（我在 stackoverflow 看到用户总是在使用 JavaScript 的时候射中自己的脚）。

```ts
let obj = {
  toString(){
    return 'Hello'
  }
}

let foo: any = {};

// ERROR: the index signature must be string, number ...
foo[obj] = 'World';

// FIX: TypeScript forces you to be explicit
foo[obj.toString()] = 'World';
```

强制明确的原因是因为一个对象上的默认`toString`实现非常糟糕，比如，在 v8 它总是返回`[object Object]`：
```ts
let obj = {message:'Hello'}
let foo: any = {};

// ERROR: the index signature must be string, number ...
foo[obj] = 'World';

// Here is where you actually stored it!
console.log(foo["[object Object]"]); // World
```
当然`number`也是支持的，因为

1. 它需要优秀的 Array/Tuple 支持
2. 甚至在如果你为一个`obj`使用它，默认`toString`实现很棒（不是`[object Object]`）

第二点显示在下面：
```ts
console.log((1).toString()); // 1
console.log((2).toString()); // 2
```

因此第一课：

> TypeScript 索引签名必须是`string`或者`number`

快速笔记：`symbols`也是有效的，并且 TypeScript 支持。但是我们先不讲这个，刚起步。

#### 声明一个索引签名

因此我们可以使用`any`去告诉 TypeScript 让我们做任何我们想要的。我们可以实际指定一个明确的索引签名。比如，假设你想要确保存储在一个对象的任何东西使用一个字符串的都兼容结构`{message: string}`。这可以通过使用`{ [index:string] : {message: string} }`声明实现。这显示在下面：
```ts
let foo:{ [index:string] : {message: string} } = {};

/**
 * Must store stuff that conforms to the structure
 */
/** Ok */
foo['a'] = { message: 'some message' };
/** Error: must contain a `message` of type string. You have a typo in `message` */
foo['a'] = { messages: 'some message' };

/**
 * Stuff that is read is also type checked
 */
/** Ok */
foo['a'].message;
/** Error: messages does not exist. You have a typo in `message` */
foo['a'].messages;
```

> TIP:索引签名的名字，比如`{ [index:string] : {message: string} }`中的`index`对于 TypeScript 没有签名只有可读性，比如，如果这是用户命名你可以使用`{ [username:string] : {message: string} }`去帮助下一个查看代码的开发者（）这可能发生在你身上。

#### 所有的签名必须符合`string`索引签名

一旦你有一个`string`索引签名，所有显式的成员必须兼容索引签名。这显示在下面：
```ts
/** Okay */
interface Foo {
  x: number;
  y: number;
}
/** Error */
interface Bar {
  x: number;
  y: string; // ERROR: Property `y` must be of type number
}
```

这提供安全性，因此任何字符串访问返回相同的结果：
```ts
interface Foo {
  x: number;
}
let foo: Foo = {x:1,y:2};

// Directly
foo['x']; // number

// Indirectly
let x = 'x'
foo[x]; // number
```

#### 使用一个限制集合的字符串字面量

一个索引签名需要索引字符串是字符串字面量的成员，通过使用映射的类型，比如：
```ts
type Index = 'a' | 'b' | 'c'
type FromIndex = { [k in Index]?: number }

const good: FromIndex = {b:1, c:2}

// Error:
// Type '{ b: number; c: number; d: number; }' is not assignable to type 'FromIndex'.
// Object literal may only specify known properties, and 'd' does not exist in type 'FromIndex'.
const bad: FromIndex = {b:1, c:2, d:3};
```

这通常和`keyof typeof`一起使用去捕获词汇类型，这描述在下面的页面。

词汇表的规范可以优雅的延迟：
```ts
type FromSomeIndex<K extends string> = { [key in K]: number }
```

#### 拥有`string`和`number`索引

这不是一个常见使用场景，尽管如此，TypeScript 编译器也支持它。

然而，它有一个显示，那就是`string`索引比`number`更严格。这是为了允许类似下面的输入：
```ts
interface ArrStr {
  [key: string]: string | number; // Must accommodate all members

  [index: number]: string; // Can be a subset of string indexer

  // Just an example member
  length: number;
}
```

#### 设计模式：嵌套索引签名

> 当添加索引签名的时候要考虑的 API 

JS 社区中常见的场景是你将看到 API 拒绝字符串索引，比如一个 JS 库中 CSS 常见的模式：
```ts
interface NestedCSS {
  color?: string;
  [selector: string]: string | NestedCSS | undefined;
}

const example: NestedCSS = {
  color: 'red',
  '.subclass': {
    color: 'blue'
  }
}
```

不要混合字符串索引和有效的值，比如，一个输入错误的 padding 将会维持未捕获：
```ts
const failsSilently: NestedCSS = {
  colour: 'red', // No error as `colour` is a valid string selector
}
```

分离嵌套到它自己的属性，比如在你一个名字比如`nest`（或者`children`或者`subnodes`等）：
```ts
interface NestedCSS {
  color?: string;
  nest?: {
    [selector: string]: NestedCSS;
  }
}

const example: NestedCSS = {
  color: 'red',
  nest: {
    '.subclass': {
      color: 'blue'
    }
  }
}

const failsSilently: NestedCSS = {
  colour: 'red', // TS Error: unknown property `colour`
}
```

#### 从索引签名排除某种属性

有时候你需要去绑定属性到一个索引签名。这不是一个建议，你应该使用前面提到的嵌套索引签名模式。

然而，如果你构建存在的 JavaScript，你可以使用交叉类型绕过它。下面显示一个错误的例子，你将会在不实用交叉集合的遭遇到：
```ts
type FieldState = {
  value: string
}

type FormState = {
  isValid: boolean  // Error: Does not conform to the index signature
}
```

这是一个使用交叉类型的方法：
```ts
type FieldState = {
  value: string
}

type FormState =
  { isValid: boolean }
  & { [fieldName: string]: FieldState }
```
注意尽管你可以声明他去建模存在的 JavaScript，你不能使用 TypeScript 创建这么一个对象:
```ts
type FieldState = {
  value: string
}

type FormState =
  { isValid: boolean }
  & { [fieldName: string]: FieldState }


// Use it for some JavaScript object you are getting from somewhere 
declare const foo:FormState; 

const isValidBool = foo.isValid;
const somethingFieldState = foo['something'];

// Using it to create a TypeScript object will not work
const bar: FormState = { // Error `isValid` not assignable to `FieldState
  isValid: false
}
```