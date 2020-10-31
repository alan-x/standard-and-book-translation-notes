# 解构

TypeScript 支持下面形式的解构（在解构后的字面量命名，破坏了解构）：

1. 对象解构
2. 数组解构

很容易认为解构是构造的反向。JavaScript 中的构造是对象字面量：
```ts
var foo = {
    bar: {
        bas: 123
    }
};
```

没有帅气的构造内置到 JavaScript，凭空创建一个对象的确会非常麻烦。解构带来相同级别的遍历从一个解构获取数据。

### 对象解构

解构很有用，因为它允许你在单独的行执行，否则需要很多行。假设下面的场景：
```ts
var rect = { x: 0, y: 10, width: 15, height: 20 };

// Destructuring assignment
var {x, y, width, height} = rect;
console.log(x, y, width, height); // 0,10,15,20

rect.x = 10;
({x, y, width, height} = rect); // assign to existing variables using outer parentheses
console.log(x, y, width, height); // 10,10,15,20
```

这里，在缺少解构的时候，你将需要从`rect`一个一个选出来`x,y,width,height`。

去赋值一个提取的变量到一个新变量名，你可以如下做：
```ts
// structure
const obj = {"some property": "some value"};

// destructure
const {"some property": someProperty} = obj;
console.log(someProperty === "some value"); // true
```

此外，你可以使用解构在解构之外获取深层数据。这显示在下面的例子：
```ts
var foo = { bar: { bas: 123 } };
var {bar: {bas}} = foo; // Effectively `var bas = foo.bar.bas;`
```

### 对象解构和剩余

你可以从一个对象选择任意数量的元素并获取对象的剩余元素，使用对象解构和剩余。
```ts
var {w, x, ...remaining} = {w: 1, x: 2, y: 3, z: 4};
console.log(w, x, remaining); // 1, 2, {y:3,z:4}
```
一个常见的使用场景是忽略某些属性。比如：
```ts
// Example function
function goto(point2D: {x: number, y: number}) {
  // Imagine some code that might break
  // if you pass in an object
  // with more items than desired
}
// Some point you get from somewhere
const point3D = {x: 1, y: 2, z: 3};
/** A nifty use of rest to remove extra properties */
const { z, ...point2D } = point3D;
goto(point2D);
```

### 数组解构

一个常见编程问题：“如何交换两个变量而不是用第三个？”。TypeScript 解决方案：
```ts
var x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2,1
```

注意数组解构影响编译器执行`[0], [1], ...`。不能保证这些值都存在。

### 数组解构和剩余

你可以从数组选择任何数量的元素，并获取一个数组的剩余元素，使用数组解构和剩余。
```ts
var [x, y, ...remaining] = [1, 2, 3, 4];
console.log(x, y, remaining); // 1, 2, [3,4]
```

### 数组解构和忽略

你可以忽略任意索引，通过让它的位置为空，比如`,,`在赋值的左手边。比如：
```ts
var [x, , ...remaining] = [1, 2, 3, 4];
console.log(x, remaining); // 1, [3,4]
```

### JS 生成

为非 ES6 目标生成的 JavaScript 调用创建临时变量，就像你将需要自己执行没有原生语言支持的解构，比如
```ts
var x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2,1

// becomes //

var x = 1, y = 2;
_a = [y,x], x = _a[0], y = _a[1];
console.log(x, y);
var _a;
```

### 总结

解构可以让你的的代码更加可读和可维护，通过减少函数并让目标清晰。数组解构可以允许你去使用数组，就像他们是元组。