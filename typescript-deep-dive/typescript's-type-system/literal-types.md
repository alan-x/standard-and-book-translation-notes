# 字面量类型

字面量是 JavaScript 原生的明确的值。

### 字符串字面量

你可以使用一个字符串字面量作为一个类型，比如：
```ts
let foo: 'Hello';

```
这里我们创建了一个变量，叫做`foo`，只允许字面量值`'Hello'`被赋值给它。这显示在下面：
```ts
let foo: 'Hello';
foo = 'Bar'; // Error: "Bar" is not assignable to type "Hello"
```
他们自身没啥用，但是何以结合到一个类型联合去创建一个强大的（和有用的）抽象，比如：
```ts
type CardinalDirection =
    | "North"
    | "East"
    | "South"
    | "West";
​
function move(distance: number, direction: CardinalDirection) {
    // ...
}
​
move(1,"North"); // Okay
move(1,"Nurth"); // Error!
```



### 其他字面量类型

TypeScript 还支持`boolean`和`number`字面量乐行，比如：
```ts
type OneToFive = 1 | 2 | 3 | 4 | 5;
type Bools = true | false;
```

### 推断
你常常会得到一些类似`Type string is not assignable to type "foo"`。下面的例子显示了这个
```ts
function iTakeFoo(foo: 'foo') { }
const test = {
  someProp: 'foo'
};
iTakeFoo(test.someProp); // Error: Argument of type string is not assignable to parameter of type 'foo'
```

这是因为`test`推断为类型`{someProp: string}`。这里的修复方式很简单，使用一个简单的类型断言去告诉 TypeScript 字面量你想要如下推断：
```ts
function iTakeFoo(foo: 'foo') { }
const test = {
  someProp: 'foo' as 'foo'
};
iTakeFoo(test.someProp); // Okay!
```
或者使用一个类型声明帮助 TypeScript 在声明的地方推断正确的东西：
```ts
function iTakeFoo(foo: 'foo') { }
type Test = {
  someProp: 'foo',
}
const test: Test = { // Annotate - inferred someProp is always === 'foo'
  someProp: 'foo' 
}; 
iTakeFoo(test.someProp); // Okay!
```


### 用例

对于字符串字面量类型有效的使用场景是：

#### 基于字符串的枚举

[TypeScript 枚举是基于数字的]()。你可以使用字符串字面量和联合类型去模拟基于字符串的么哦句，就像我们在前面的`CardinalDirection`例子。你甚至可以生成一个`Key:Value`构造使用下面的函数：
```ts
/** Utility function to create a K:V from a list of strings */
function strEnum<T extends string>(o: Array<T>): {[K in T]: K} {
  return o.reduce((res, key) => {
    res[key] = key;
    return res;
  }, Object.create(null));
}
```

然后生成字面量类型联合，使用`keyof typeof`。这是一个完整的例子：
```ts
/** Utility function to create a K:V from a list of strings */
function strEnum<T extends string>(o: Array<T>): {[K in T]: K} {
  return o.reduce((res, key) => {
    res[key] = key;
    return res;
  }, Object.create(null));
}
​
/**
  * Sample create a string enum
  */
​
/** Create a K:V */
const Direction = strEnum([
  'North',
  'South',
  'East',
  'West'
])
/** Create a Type */
type Direction = keyof typeof Direction;
​
/** 
  * Sample using a string enum
  */
let sample: Direction;
​
sample = Direction.North; // Okay
sample = 'North'; // Okay
sample = 'AnythingElse'; // ERROR!
```

#### 模型化存在的 JavaScript API

比如[CodeMirror 编辑器有一个可选的`readonly`选项]()可以是`boolean`或者字面量字符串`"nocursor"`（有效的值`true,false,"nocursor"`）。他可以声明为：
```ts
readOnly: boolean | 'nocursor';

```

#### 区分联合
我们将[在这本书的后面]()覆盖