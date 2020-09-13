`let`和`const`是 JavaScript 中变量声明两个相对新的概念。[就像我们之前提到的]()，`let`在某些方面类似`var`，但是允许用户去避免一些 JavaScript 用户陷入的常见“陷阱”，

`const`是`let`的扩展，它防止被重新赋值。

因为 TypeScript 是 JavaScript 的扩展，语言天生支持`let`和`const`。这里我们将详细说明这些新声明和为什么他们比`var`更适合。

如果你随意使用过 JavaScript，下一个章节可能是刷新你的记忆的更好方式。如果你非常熟悉 JavaScript 中`var`声明的所有怪癖，你可能发现它更容易去越过。

### var 声明

JavaScript 中声明一个变量通常使用`var`关键字完成。
```
var a = 10;
```

正如你可能指出的，我们只是声明了一个名为`a`，值为`10`的变量。

我们也可以在函数内声明一个变量：
```
function f() {
  var message = "Hello, world!";

  return message;
}
```
我们也可以在其他函数内访问这些相同变量：
```
function f() {
  var a = 10;
  return function g() {
    var b = a + 1;
    return b;
  };
}

var g = f();
g(); // returns '11'
```

在前面的例子中，`g`捕获声明于`f`的变量`a`。任何时刻，`g`被调用，`a`的值将会被捆绑到`f`中的`a`，尽管`g`在`f`被调用完成之后`f`才被调用，它可以访问和修改`a`。
```
function f() {
  var a = 1;

  a = 2;
  var b = g();
  a = 3;

  return b;

  function g() {
    return a;
  }
}

f(); // returns '2'
```

### 作用域规则

对于哪些用于其他语言的`var`声明有一些奇怪的作用域规则。看下面的例子：
```
function f(shouldInitialize: boolean) {
  if (shouldInitialize) {
    var x = 10;
  }

  return x;
}

f(true); // returns '10'
f(false); // returns 'undefined'
```

一些读者可能会再看一边这个例子。变量`x`声明在 if 块中，然而，我们可以在这个块的外部访问它。这是因为`var`声明在包含他们的函数、模块、命名空间、或者全局作用域都可以访问 - 这些我们将会在稍后提到 - 无视包含的块。一些人称这个为 var 作用域或者函数作用域。参数也是函数级作用域。

这些作用域规则会导致多种类型的错误。他们加剧的一个问题是多次声明相同变量不是错误的事实：

```
function sumMatrix(matrix: number[][]) {
  var sum = 0;
  for (var i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i];
    for (var i = 0; i < currentRow.length; i++) {
      sum += currentRow[i];
    }
  }

  return sum;
}
```

对于一些有经验的 JavaScript 开发者，可能很容易指出，但是内部`for`循环将会意外的修改变量`i`，因为`i`引用相同的函数作用域变量。就像有经验的开发这现在知道的，类似系列的 bug 从代码审阅中溜走，并可能成为无尽的沮丧来源。

### 变量捕捉怪癖
快速猜一下下面片段的输出：
```
for (var i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100 * i);
}
```

给那些不熟的人，`set`将会尝试在某个毫秒数后去执行一个函数（尽管等到什么东西去停止运行）。


准备好了？看看：
```
10
10
10
10
10
10
10
10
10
10
```

很多 JavaScript 开发者对这个行为非常熟悉。但是如果你很惊艳，你并不孤单。大部分人们期待输出是：
```
0
1
2
3
4
5
6
7
8
9
```

记得我们前面提到的变量捕获？我们传递给`setTimeout`的任何函数表达式实际上都从相同的作用域引用相同的`i`。

花点时间想想这意味着什么。`setTimeout`将会在一些毫秒数之后运行一个函数，但是只有在`for`循环停止执行之后；随着`for`循环停止执行，`i`的值是`10`。因此，每一次函数被调用，它将会答应`10`！。

一个常见解决方案是使用 IIFE，一个立即调用函数表达式-在每一个迭代捕获`i`：
```
for (var i = 0; i < 10; i++) {
  // capture the current state of 'i'
  // by invoking a function with its current value
  (function (i) {
    setTimeout(function () {
      console.log(i);
    }, 100 * i);
  })(i);
}

```

这个奇怪的模式其实非常常见。参数列表的`i`是声明在`for`循环的`i`的阴影，但是因为我们以相同的名字命名，我们不需要去太过重新定义循环体。

### let 声明

到现在，你已经发现`var`有一些问题，这也正式是为什么`let`语句被引入。除了使用的关键字，`let`语句和`var`语句的写法相同。
```
let hello = "Hello!";
```

关键的不同不是在语法，而是在语义，，这也是我们将深入的。

### 块级作用域

当一个变量使用`let`声明的时候，它所使用的称为词法作用域或者块级作用域。不像使用`var`声明的变量，它的作用域泄露到包含它的函数，块级作用域变量不能在他们嵌套的块或者`for`循环外可见。
```
function f(input: boolean) {
  let a = 100;

  if (input) {
    // Still okay to reference 'a'
    let b = a + 1;
    return b;
  }

  // Error: 'b' doesn't exist here
  return b;
}

```

这里，我们有两个本地变量`a`和`b`，阿门的范围局限于`f`的体内，`b`的作用域局限于包含的`if`语句块。


`catch`语句中的变量声明也有类似的作用域规则。
```
try {
  throw "oh no!";
} catch (e) {
  console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

块级作用域变量的另一个属性是在他们真实声明之前，无法读取和写入。尽管这些变量贯穿他们的作用域‘存在’，直到他们被声明之前的部分是他们的暂存死区。这是你无法在`let`语句之前访问他们的一种复杂说法，幸运的是，TypeScript 将会让你知道这个。
```
a++; // illegal to use 'a' before it's declared;
let a;
```

需要注意你依旧可以捕获块级作用域变量，在他声明之前，唯一的问题是在声明之前调用函数是非法的。如果目标是 ES2015，一个现代运行时将会抛出一个错误；然而，现在 TypeScript 允许并且不会汇报这个错误。
```
function foo() {
  // okay to capture 'a'
  return a;
}

// illegal call 'foo' before 'a' is declared
// runtimes should throw an error here
foo();

let a;
```

了解更多关于暂存死区的信息，在[Mozilla 开发者网络]()查阅相关内容。

### 重新声明和阴影

使用`var`声明，无论你声明多少次你的变量，你只能得到一个。
```
function f(x) {
  var x;
  var x;

  if (true) {
    var x;
  }
}
```

在前面的例子，`x`的所有声明实际都引用相同的`x`，这是完美可用的。这通常是 bug 的来源。幸运的是，`let`声明没有那么宽容。
```
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same sco
```

变量不需要都是块级作用域，因为 TypeScript 告诉我们这是一个我呢体。
```
function f(x) {
  let x = 100; // error: interferes with parameter declaration
}

function g() {
  let x = 100;
  var x = 100; // error: can't have both declarations of 'x'
}

```


这不是说块级作用域变量用能声明为函数作用域变量。块级作用域变量只需要声明到一个单独的不同的块。
```
function f(condition, x) {
  if (condition) {
    let x = 100;
    return x;
  }

  return x;
}

f(false, 0); // returns '0'
f(true, 0); // returns '100'
```

在一个嵌套的作用域引入一个新的名字的行为叫做映射。这是一把双刃剑，意外的影射会引入某些 bug，也能防止某些 bug。比如，想象我们之前使用`;et`变量写的`sumMatrix`。

```
function sumMatrix(matrix: number[][]) {
  let sum = 0;
  for (let i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i];
    for (let i = 0; i < currentRow.length; i++) {
      sum += currentRow[i];
    }
  }

  return sum;
}
```

这个版本的循环将会真实的正确执行描述，因为内部循环的`i`从外部循环影射`i`。

影射应该通常在编写更清晰的代码的时候被避免。尽管有一些场景可以更好的利用它，应应该使用你最好的判断。

### 块级作用域变量捕获

当我们使用`var`声明第一次接触到变量捕捉的想法，我们简单介绍一下变量捕捉是如何发生的。为了更好的理解他，每一次作用域运行的时候，它创建一个变量“环境”。这个环境和他捕获的变量会存在，就算在范围内的任何东东都完成执行。
```
function theCityThatAlwaysSleeps() {
  let getCity;

  if (true) {
    let city = "Seattle";
    getCity = function () {
      return city;
    };
  }

  return getCity();
}
```

因为我们在它的环境中捕获了`city`，我们依旧能够访问它，尽管`if`块完成执行。

会议一下我们之前的`setTimeout`例子，我们最终使用一个 IIFE 去捕获`for`循环每一次迭代的变量的状态。这么做的效果是为你捕获的变量创建了一个变量环境。这有点痛苦，但是幸运的是，你在 TypeScript 中不需要再这么做。

`let`声明作为循环的一部分声明的时候有非常不同的行为。与其只是引入一个新的环境到循环自身，这些声明为每个迭代创建了一个新的作用域。因为这是我们在 IIFE 中做的，我们可以使用`let`声明改变我们旧的`setTimeout`例子。
```
for (let i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100 * i);
}

```
正如期待的，这将会输出：
```
0
1
2
3
4
5
6
7
8
9
```


### const 声明

`const`声明是声明一个额变量的另一种方式。
```
const numLivesForCat = 9;

```

他们和`let`声明和祥，但是，正如他们的名字暗示的，他们的值一档绑定，就不能改变。换句话说，他们呢和`let`有相同的作用域，但是你不能为他们重新赋值。

这不要和他们引用的值是不可变的混淆。
```
const numLivesForCat = 9;
const kitty = {
  name: "Aurora",
  numLives: numLivesForCat,
};

// Error
kitty = {
  name: "Danielle",
  numLives: numLivesForCat,
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

除非你采取特殊的手段去避免，`const`变量内部状态依旧是可修改的。幸运的是，TypeScript 允许你去指定对象的成员是`readonly`。[接口章节]()有详细的信息。

### let vs const

假设我们有两种不同类类型的声明有相同的作用域语义，我们很自然会问用哪一个。就像大部分着陆问题，回答是：看情况。

应用最小授权原则，所有声明，除了哪些你计划去修改的，都应该使用`const`。原理是，如果一个变量不需要被写入，工作在相同代码库的其他人不应该自动能够写入对象，也需要考虑他们是否真的需要重新赋值这个变量。使用`const`也让代码在推理数据流的时候更加可预测。

使用你最好的判断，如果可行，和你团队的其他成员讨论一下。

本手册主要使用`let`声明

### 解构

TypeScript  拥有的 ECMAScript 2015 的另一个特性是解构。



### 数组解构

解构最简单的形式是数组解构赋值：

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```
这创建了两个变量，叫做`first`和`second`。折合使用索引一样，但是更方便。
```
first = input[0];
second = input[1];
```

解构使用在已经声明的变量也行：
```
// swap variables
[first, second] = [second, first];
```
和函数参数一起使用：
```
function f([first, second]: [number, number]) {
  console.log(first);
  console.log(second);
}
f([1, 2]);
```

你可以使用`...`语法为剩余项创建一个列表：
```
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

当然，因为这是 JavaScript，你可以忽略你不关心你额剩余元素：
```
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```
或者其他元素：
```
let [, second, , fourth] = [1, 2, 3, 4];
console.log(second); // outputs 2
console.log(fourth); // outputs 4
```

### 元组解构

元组可以像数组一样解构；解构变量获得对应元组元素的类型：
```
let tuple: [number, string, boolean] = [7, "hello", true];

let [a, b, c] = tuple; // a: number, b: string, c: boolean

```

解构一个元素超出他的元素的索引范围是一个错误。
```
let [a, b, c, d] = tuple; // Error, no element at index 3
```
随着使用数组，你可以使用`...`解构剩余的元组，获取到一个更短的元组：
```
let [a, ...bc] = tuple; // bc: [string, boolean]
let [a, b, c, ...d] = tuple; // d: [], the empty tuple
```
或者忽略剩余的元素，或者其他元素：
```
let [a] = tuple; // a: number
let [, b] = tuple; // b: string
```

### 对象解构

你也可以解构对象：
```
let o = {
  a: "foo",
  b: 12,
  c: "bar",
};
let { a, b } = o;
```

这从`o.a`和`o.b`创建了两个新变量`a`和`b`。注意你可以跳过`c`，如果你不需要它。

就像数组解构，你可以赋值，而不需要声明：
```
({ a, b } = { a: "baz", b: 101 });

```
注意我们使用括号包裹这个语句。JavaScript 通常转化`{`为块的开始。
你可能创建一个变量去持有对象剩余的项目，使用`...`语法：
```
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;

```

#### 适当重命名
你可以给属性一个不同的名字：
```
let { a: newName1, b: newName2 } = o;

```
这里的语法开始有点混淆。你可以吧`a:newName1`读作`a`为`newName1`。方向是从左到右，如果你编写：
```
let newName1 = o.a;
let newName2 = o.b;
```
令人疑惑的是，冒号不走味类型的指示器。类型，如果你指定它，依旧需要在之后编写主要的解构：
```
let { a, b }: { a: string; b: number } = o;

```

#### 默认值
默认值让你指定一个默认值，防止一个属性是 undefined：
```
function keepWholeObject(wholeObject: { a: string; b?: number }) {
  let { a, b = 1001 } = wholeObject;
}
```

在这个例子中，`b?`意味着`b`是可选的。因此它可能是`undefined`。`keepWholeObject`现在有一个变量`wholeObject`，属性`a`和`b`，就尽管`b`是 udefined。

### 函数解构

解构在函数声明也有用，最简单的场景：
```
type C = { a: string; b?: number };
function f({ a, b }: C): void {
  // ...
}
```
但是为参数指定默认更常见，解构中获取默认值更麻烦。首先，你需要记得在默认值之前使用这个模式：
```
function f({ a = "", b = 0 } = {}): void {
  // ...
}
f();
```
前面的片段是类型推断的一个例子，在手册之后的章节将会解释。

你需要去记住在解构属性中为可选属性设置默认值，而不是在主初始化器。记住`C`使用可选的`b`定义：
```
function f({ a, b = 0 } = { a: "" }): void {
  // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to { a: "" }, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument

```

使用解构要小心。就像前面的例子显示的，就算是很小的解构表达式也是令人疑惑的。特别是深层解构，理解更加困难，就算没有重命名，默认值和类型声明。尝试去保持解构表达式小且简单。你可以编写赋值，解构将自动生成。

### 扩散

扩散操作符是解构的反向操作。它允许你扩散一个数组到另一个数组，或者一个对象到另一个对象，比如：
```
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```

这给 bothPlus 值`[0,1,2,3,4,5]`。扩展创建了一个``first`和`second`的浅赋值。他没有被扩散改变。

Nike 也可以扩散对象：
```
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };

```

现在`search`是`{ food: "rich", price: "$$", ambiance: "noisy" }`。对象扩散比数组扩散更加复杂。类似数组扩散，它从左到右处理，但是结果依旧是一个对象。这意味着扩展对象来的属性覆盖来更早的属性。因此，如果我们定义前面一个例子扩散到最后：
```
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };

```

`defaults`中的属性覆盖`food`属性`food: "rich"`，这是我们这个场景不想要的。

对象扩展也有一堆其他特别的限制。首先，它只包含一个对象的[自有的，可枚举的属性]()。基本上，这意味着你丢失方法，当扩散一个实例到另一个对象：
```
class C {
  p = 12;
  m() {}
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```

其次，TypeScript 编译器不允许从泛型扩散类型参数。这个特性在语言未来的版本可能会出现。