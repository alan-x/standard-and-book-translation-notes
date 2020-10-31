# 扩展操作符

扩展操作符的主要目标是扩展数组或者对象的元素。这最好使用例子来解释。


### apply

一个常见使用场景是扩展一个对象到一个函数参数。之前，你将需要使用`Function.prototype.apply`。
```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo.apply(null, args);
```

现在你可以通过在参数前面使用`...`简化，就像下面展示的：
```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo(...args);
```
这里我们展开了`args`数组到`arguments`位置。

### 解构

我们已经在解构中开到这个使用：
```ts
var [x, y, ...remaining] = [1, 2, 3, 4];
console.log(x, y, remaining); // 1,2,[3,4]
```

这里的动机是去简化，让你捕获数组的剩余元素更简单，当解构的时候。

### 数组赋值

展开操作符允许你去简单的放置一个扩展版本的数组到另一个数组。这在下面的例子展示：
```ts
var list = [1, 2];
list = [...list, 3, 4];
console.log(list); // [1,2,3,4]
```

你可以放置扩展的数组在任何位置，并得到你期待的效果：
```ts
var list = [1, 2];
list = [0, ...list, 4];
console.log(list); // [0,1,2,4]
```

### 对象扩展

你也可以扩展一个对象到其他对象。一个常见使用场景是简化添加一个属性到一个对象而不需要操作原始：
```ts
const point2D = {x: 1, y: 2};
/** Create a new object by using all the point2D props along with z */
const point3D = {...point2D, z: 3};
```

对于对象，你放置展开的顺序很重要。这和`Object.assign`工作的很像，如你期望的执行：先出现的被后来的'覆盖'：
```ts
const point2D = {x: 1, y: 2};
const anotherPoint3D = {x: 5, z: 4, ...point2D};
console.log(anotherPoint3D); // {x: 1, y: 2, z: 4}
const yetAnotherPoint3D = {...point2D, x: 5, z: 4}
console.log(yetAnotherPoint3D); // {x: 5, y: 2, z: 4}
```

另一个常见使用场景是简化浅扩展：
```ts
const foo = {a: 1, b: 2, c: 0};
const bar = {c: 1, d: 2};
/** Merge foo and bar */
const fooBar = {...foo, ...bar};
// fooBar is now {a: 1, b: 2, c: 1, d: 2}
```

### 总结

`apply`是你在 JavaScript 中常用的，因此有一个更好的语法是很好的，你不需要为`this`参数使用丑陋的`null。当然有一个专用的语法去移动数组出来（解构）或者进去（赋值）其他数组，提供一个整洁的语法，当你在特定数组执行数组处理。