[已校对]
# TypeScript 的类型系统

当我们讨论[为什么是 TypeScirpt](https://basarat.gitbook.io/typescript/getting-started/why-typescript)的时候我们介绍了 TypeScript 类型系统的主要特性。下面是讨论的关键点，我们不需要深入解释：

- TypeScript 的类型系统设计为可选择的，因此你的 JavaScript 就是 TypeScript。

- TypeScript 不阻塞 JavaScript 生成，在类型错误存在的时候，允许你去渐进的更新的你的 JS 到 TS。

现在从 TypeScript 类型系统的语法开始吧。这种方式，你可以开始立即在你的代码中使用这些声明，并看到收益。这将会为之后的深入做准备。

### 基础注解

就像前面提到的，类型使用`:TypeAnnotation`语法声明。任何类型声明空间的变量可以用作类型声明。

下面的例子展示了变量，函数参数和函数返回值的类型声明：
```ts
var num: number = 123;
function identity(num: number): number {
    return num;
}
```

#### 原始类型

JavaScript 原始类型在 TypeScript 类型系统表现的很好。这意味着`string`，`number`，`boolean`如下展示：
```ts
var num: number;
var str: string;
var bool: boolean;

num = 123;
num = 123.456;
num = '123'; // Error

str = '123';
str = 123; // Error

bool = true;
bool = false;
bool = 'false'; // Error
```

#### 数组

TypeScript 为数组提供了专门的语法让你声明和标记你的代码更简单。这个语法只是简单的添加`[]`后缀到任何有效的类型声明（比如`:boolean[]`）。它允许你去安全的做任何常规的数组操作，从类似赋值错误的类型到一个成员之类的错误中保护你。展示如下：
```ts
var boolArray: boolean[];

boolArray = [true, false];
console.log(boolArray[0]); // true
console.log(boolArray.length); // 2
boolArray[1] = true;
boolArray = [false, false];

boolArray[0] = 'false'; // Error!
boolArray = 'false'; // Error!
boolArray = [true, 'false']; // Error
```

#### 接口

接口是 TypeScript 组合多个类型声明到一个单一的具名声明的主要方式。考虑下面的例子：
```ts
interface Name {
    first: string;
    second: string;
}

var name: Name;
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```

这里，我们组合了声明`first: string`+`second: string`到一个新的声明`Name`，强制在每个独立的成员上做类型检测。TypeScript 中的接口有很多的能力，我们将使用一整个章节去描述你如何利用这些优点。

#### 内联类型声明

除了创建新的`interface`，你可以使用`:{ /*Structure*/}`内联语法声明任何你想要的。前面的例子使用内联类型再展示一次：
```ts
var name: {
    first: string;
    second: string;
};
name = {
    first: 'John',
    second: 'Doe'
};

name = {           // Error : `second` is missing
    first: 'John'
};
name = {           // Error : `second` is the wrong type
    first: 'John',
    second: 1337
};
```
内联类型对于为一些东西快速提供一个一次性的类型声明很不错。它节约你创建一个（一个潜在的坏处）类型名称的繁琐。然而，如果你发现你多次放置相同的类型声明，最好考虑重构它到一个接口（或者一个在这个章节后面提到的`type alias`）。


### 特殊类型

除了已经提到的原始类型之外，在 TypeScript 中还有少量的类型有特殊的意义。他们是`any`，`null`，`undefined`，`void`。

#### any

`any`类型在 TypeScript 中有特殊的地位。它给你一个类型系统的逃生出口，告诉编译器滚开。`any`和`any`，还有其他类型系统中的任何系统都兼容。这意味着任何东西都可以赋值给它，它也可以被赋值给任何东西。这展示在下面的例子：
```ts
var power: any;

// Takes any and all types
power = '123';
power = 123;

// Is compatible with all types
var num: number;
power = num;
num = power;
```

如果你正在转译 JavaScript 到 TypeScript，你可能会在开始的时候和`any`走的很近。然而，不要太严肃对待这个关系，因为这意味着类型安全的确保取决于你。你只是告诉编译器不要采取任何有意义的静态分析。

#### `null`和`undefined`

他们被如何对待取决于`strictNullChecks`编译器标志（我们将在之后提到这个标志）。当`strictNullCheck:false`，`null`和`undefined`JavaScript 字面量被类型系统有效的对待为`any`类型的东西。这些字面量可以被赋值给任何其他类型。这在下面的例子展示：
```ts
var num: number;
var str: string;

// These literals can be assigned to anything
num = null;
str = undefined;
```

#### `:void`

使用`:void`标示函数没有返回类型：
```ts
function log(message): void {
    console.log(message);
}
```

### 泛型

计算机科学中的很多算法和数据结构不依赖于对象的具体的类型。然而，你依旧想要在大量变量之间强制一个约束。一个简单的玩具例子是一个接受一个列表的项并返回一个反向列表项的函数。这里的约束在传入函数的东西和函数返回的东西之间：
```ts
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// Safety!
reversed[0] = '1';     // Error!
reversed = ['1', '2']; // Error!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

基本上可以这么说，函数`reverse`接受一个某种类型`T`的数组（`items: T[]`）（注意`reverse<T>`的类型参数）和返回值的数组类型`T`（注意`:T[]`）。因为`reverse`函数返回它接受的相同类型的项，TypeScript 知道`reversed`变量也是`number[]`类型，将会赋予你类型安全。同样，如果你传递一个`string[]`数组到反向函数，返回的结果也是一个数组的`sting[]`，并且你也会得到类型安全，就像下面显示的：
```ts
var strArr = ['1', '2'];
var reversedStrs = reverse(strArr);

reversedStrs = [1, 2]; // Error!
```
实际上 JavaScript 数组已经有了一个`.reverse`函数，并且 TypeScript 内部的确使用泛型去定义它的结构：
```ts
interface Array<T> {
 reverse(): T[];
 // ...
}
```
这意味着在任何数组上调用`.reverse`的时候，你将会得到类型安全，就像下面显示的：
```ts
var numArr = [1, 2];
var reversedNums = numArr.reverse();

reversedNums = ['1', '2']; // Error!
```

当我们之后在**外界声明**章节展现`lib.d.ts`的时候，我们将讨论更多关于`Array<T>`接口。

### 联合类型

在 JavaScript 中很常见的是允许一个属性是多个类型之一，比如一个`string`或者一个`number`。这是联合类型（通过在类型声明中标记为`|`，比如`string|number`）有用的地方。一个常见的使用场景是一个函数可以接受一个单独的变量或者一个数组的对象，比如：
```ts
function formatCommandline(command: string[]|string) {
    var line = '';
    if (typeof command === 'string') {
        line = command.trim();
    } else {
        line = command.join(' ').trim();
    }

    // Do stuff with line: string
}
```

### 交叉类型

`extend`是 JavaScript 中非常常见的模式，它接受两个对象并创建一个新的有两个对象特性的。一个**交叉类型**允许允许你安全的去使用这个模式，如下面展现：
```ts
function extend<T, U>(first: T, second: U): T & U {
  return { ...first, ...second };
}

const x = extend({ a: "hello" }, { b: 42 });

// x now has both `a` and `b`
const a = x.a;
const b = x.b;
```

### 元组类型
JavaScript 没有一流的元组支持。人们通常只是使用数组作为一个元组。这正是 TypeScript 类型系统所支持的。元素可以使用`:[typeofmember1, typeofmember2]`声明。一个元组可以有任何数量的成员。元组在下面的例子显示：
```ts
var nameNumber: [string, number];

// Okay
nameNumber = ['Jenny', 8675309];

// Error!
nameNumber = ['Jenny', '867-5309'];
```
将这个和 TypeScript 支持的解构结合，元组几乎是一流的，尽管底层是数组：
```ts
var nameNumber: [string, number];
nameNumber = ['Jenny', 8675309];

var [name, num] = nameNumber;
```


### 类型别名

TypeScript 提供便利的语法，让你为你想要在多个地方使用的类型声明提供名字。别名使用`type SomeName = someValidTypeAnnotation`语法创建。下面是一个例子：
```ts
type StrOrNum = string|number;

// Usage: just like any other notation
var sample: StrOrNum;
sample = 123;
sample = '123';

// Just checking
sample = true; // Error!
```

不像一个`interface`，你可以将一个类型直接别名为任何类型声明（对于联合和交叉类型之类的东西非常有用）。这是一些让你熟悉这个语法的例子：
```ts
type Text = string | { text: string };
type Coordinates = [number, number];
type Callback = (data: string) => void;
```

> 提示：如果你需要类型声明的层级，使用`interface`。他们可以使用`implement`和`extends`。

> 提示：为简单的对象解构使用类型别名（像`Coordinates`），只是给他们一个语义名字。当你想要给一个联合或者交叉类型语义的时候，一个类型别名的方式也行。


### 总结

现在你可以开始声明你的大部分 JavaScript 代码，我们可以进入 TypeScript 的类型系统可用的所有威力的细节。