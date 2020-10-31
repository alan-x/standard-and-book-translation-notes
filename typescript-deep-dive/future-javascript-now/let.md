# let

JavaScript 中的`var`变量是函数范围的。这和很多其他语言（C#/Java 等），变量是块范围的。如果你引入块范围思想到 JavaScript，你讲期待下面打印`123`，然而它将会是`456`：
```ts
var foo = 123;
if (true) {
    var foo = 456;
}
console.log(foo); // 456
```

这是因为`{`没有创建一个新的变量范围。变量`foo`在 fi 块内和块外是一样的。这是 JavaScript 编程中常见的错误圆头。这也是为什么 TypeScript（和 ES6）引入`let`关键字去允许你去使用块范围去定义变量。也就是说，如果你使用`let`，而不是`var`，你讲得到一个真正的唯一的元素，和你可能定义在范围外面的不再链接。相同的例子使用`let`展示：
```ts
let foo = 123;
if (true) {
    let foo = 456;
}
console.log(foo); // 123
```

`let`会从错误中拯救你的另一个地方是循环：
```ts
var index = 0;
var array = [1, 2, 3];
for (let index = 0; index < array.length; index++) {
    console.log(array[index]);
}
console.log(index); // 0
```

我们真诚的发现，对于新的和存在的多语言开发者，尽可能使用`let`将会导致更少的意外。

### 函数创建一个新的范围

因为我们提到它，我们展示函数在 JavaScript 创建新变量范围。考虑下面：
```ts
var foo = 123;
function test() {
    var foo = 456;
}
test();
console.log(foo); // 123
```

这个行为就像你期待的。没有这个，它将非常难去使用 JavaScript 编写代码。

### 生成的 JS

TypeScript 生成的 JS 只是简单重命名`let`变量，如果一个类似的名字已经存在在周围范围。比如，下面是使用`var`替代`let`生成的：
```
if (true) {
    let foo = 123;
}

// becomes //

if (true) {
    var foo = 123;
}
```

然而，如果变量名已经被周围的傻姑娘喜爱问使用，则一个新的变量名被如下生成（注意`foo_1`）：
```ts
var foo = '123';
if (true) {
    let foo = 123;
}

// becomes //

var foo = '123';
if (true) {
    var foo_1 = 123; // Renamed
}
```

### Switch

你可以包裹你的`case`体在`{}`中去重用变量名，取决于不同的`case`语句，就像如下展示：
```ts
switch (name) {
    case 'x': {
        let x = 5;
        // ...
        break;
    }
    case 'y': {
        let x = 10;
        // ...
        break;
    }
}
```

### 闭包中的 let

对于一个 JavaScript 开发者一个常见开发面临的问题是这个简单文件的输出是什么：
```ts
var funcs = [];
// create a bunch of functions
for (var i = 0; i < 3; i++) {
    funcs.push(function() {
        console.log(i);
    })
}
// call them
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```

有些人可能期待他是`0,1,2`。意外的是对于所有这些函数他是`3`。原因是所有三个函数使用外部范围的变量`i`。并且在我们执行他们的时候（在第二个循环），`i`的值将会是`3`（这是第一个循环的终止条件）；

一个修复是常见一个新的变量，在一个循环指定循环迭代。就像我们前面学到的，我们可以创建一个新的变量范围，通过创建一个函数并立即执行它（比如，类的 IIFE 模式`(function() { /* body */ })();`）就像下面展示的：
```ts
var funcs = [];
// create a bunch of functions
for (var i = 0; i < 3; i++) {
    (function() {
        var local = i;
        funcs.push(function() {
            console.log(local);
        })
    })();
}
// call them
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```

这里函数关闭了（因此叫做`closure`）本地变量（通常叫做`local`）并使用这替代循环变量`i`。

> 注意闭包有性能影响（他们需要去存储周围状态）

在一个循环中的 ES6 `let`关键字有前面的例子相同的行为：
```ts
var funcs = [];
// create a bunch of functions
for (let i = 0; i < 3; i++) { // Note the use of let
    funcs.push(function() {
        console.log(i);
    })
}
// call them
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```
使用一个`let`替代`var`为每一个循环迭代创建一个唯一的`i`。

### 总结

`let`对于大部分代码是非常有用的。他可以很好的增强你的代码的可读性，并减少编程错误的个机会。