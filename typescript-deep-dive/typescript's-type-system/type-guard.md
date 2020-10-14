# 类型守卫

- [类型守卫]()
- [用户定义的类型守卫]()

### 类型守卫

类型守卫允许你在一个条件化的块中向下转型一个对象的类型。

### typeof

TypeScript能够意识到 JavaScript `instanceof`和`typeof`操作符的使用。如果你在一个条件的块中使用这些，TypeScript 将会理解变量的类型在条件块中不同。这是一个例子，TypeScript 意识到一个特殊的函数不存在于`string`，并指出这是一个用户输入错误：
```ts
function doSomething(x: number | string) {
    if (typeof x === 'string') { // Within the block TypeScript knows that `x` must be a string
        console.log(x.subtr(1)); // Error, 'subtr' does not exist on `string`
        console.log(x.substr(1)); // OK
    }
    x.substr(1); // Error: There is no guarantee that `x` is a `string`
}
```

#### instanceof

这是一个关于类和`instanceof`的例子：
```ts
class Foo {
    foo = 123;
    common = '123';
}

class Bar {
    bar = 123;
    common = '123';
}

function doStuff(arg: Foo | Bar) {
    if (arg instanceof Foo) {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    if (arg instanceof Bar) {
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }

    console.log(arg.common); // OK
    console.log(arg.foo); // Error!
    console.log(arg.bar); // Error!
}

doStuff(new Foo());
doStuff(new Bar());
```

TypeScript 甚至理解`else`，当`if`转型到它知道的类型，则在 else 中，它的定义不是这个类型。这是一个例子：
```ts
class Foo {
    foo = 123;
}

class Bar {
    bar = 123;
}

function doStuff(arg: Foo | Bar) {
    if (arg instanceof Foo) {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    else {  // MUST BE Bar!
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }
}

doStuff(new Foo());
doStuff(new Bar());
```

#### in

`in`操作符对一个对象的一个属性的存在做了一个安全检测，可以用作类型守卫。比如：
```ts
interface A {
  x: number;
}
interface B {
  y: string;
}

function doStuff(q: A | B) {
  if ('x' in q) {
    // q: A
  }
  else {
    // q: B
  }
}
```

### 字面量类型守卫

你可以使用`===`/`==`/`!==`/`!=`去区分直面量值：
```ts
type TriState = 'yes' | 'no' | 'unknown';

function logOutState(state:TriState) {
  if (state == 'yes') {
    console.log('User selected yes');
  } else if (state == 'no') {
    console.log('User selected no');
  } else {
    console.log('User has not made a selection yet');
  }
}
```

这甚至可以在联合的字面量类型中也能工作。你可以检查共享的属性名去识别联合，比如：
```ts
type Foo = {
  kind: 'foo', // Literal type 
  foo: number
}
type Bar = {
  kind: 'bar', // Literal type 
  bar: number
}

function doStuff(arg: Foo | Bar) {
    if (arg.kind === 'foo') {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    else {  // MUST BE Bar!
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }
}
```

#### 使用`strictNullChecks` 的 null 和 undeinfed

TypeScript 足够聪明的去使用`== null`/`!= null`检查排除`null`和`undefined`。比如：
```ts
function foo(a?: number | null) {
  if (a == null) return;

  // a is number now.
}
```


#### 用户定义的类型守卫

JavaScript 没有足够丰富的内建运行时自省支持。当你值使用普通 Javascript 对象（使用你控制的结构化输入），你甚至不能访问`instanceof`或者`typeof`。对于这些场景，一可以创建用户定义的类型守卫函数。他们只是函数，返回`someArgumentName is SomeType`。这里是一个例子：
```ts
/**
 * Just some interfaces
 */
interface Foo {
    foo: number;
    common: string;
}

interface Bar {
    bar: number;
    common: string;
}

/**
 * User Defined Type Guard!
 */
function isFoo(arg: any): arg is Foo {
    return arg.foo !== undefined;
}

/**
 * Sample usage of the User Defined Type Guard
 */
function doStuff(arg: Foo | Bar) {
    if (isFoo(arg)) {
        console.log(arg.foo); // OK
        console.log(arg.bar); // Error!
    }
    else {
        console.log(arg.foo); // Error!
        console.log(arg.bar); // OK
    }
}

doStuff({ foo: 123, common: '123' });
doStuff({ bar: 123, common: '123' });
```

#### 类型守卫和回调

TypeScript 不假设类型守卫在回调用保持活跃，因为保持这种假设是危险的。比如：
```ts
// Example Setup
declare var foo:{bar?: {baz: string}};
function immediate(callback: ()=>void) {
  callback();
}


// Type Guard
if (foo.bar) {
  console.log(foo.bar.baz); // Okay
  functionDoingSomeStuff(() => {
    console.log(foo.bar.baz); // TS error: Object is possibly 'undefined'"
  });
}
```
修复很简单，就像存储推断的安全值在一个本地变量，自动确保它不被外部改变，TypeScript 可以轻易理解这个：
```ts
// Type Guard
if (foo.bar) {
  console.log(foo.bar.baz); // Okay
  const bar = foo.bar;
  functionDoingSomeStuff(() => {
    console.log(bar.baz); // Okay
  });
}
```