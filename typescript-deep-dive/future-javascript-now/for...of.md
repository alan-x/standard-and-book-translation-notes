# for...of

一个 JavaScript 开发者的一个常见错误是`for...in`对于一个数组并不遍历数组的项目。而是遍历传入的对象的 key。这显示在下面的例子。这里你将期待`9.2.5`，但是你得到索引`0,1,2`：
```ts
var someArray = [9, 2, 5];
for (var item in someArray) {
    console.log(item); // 0,1,2
}
```

这是`for...of`存在在 TypeScript(和 ES6)原因之一。下面的数组遍历正确记录期待的成员：
```ts
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item); // 9,2,5
}
```
同样的，TypeScript 使用`for...of`遍历一个字符串：
```ts
var hello = "is it me you're looking for?";
for (var char of hello) {
    console.log(char); // is it me you're looking for?
}
```

### JS 生成

对于 ES6 之前的目标，TypeScript 将生成标准的`for (var i = 0; i < list.length; i++)`循环类型。比如，比如这是前面例子生成的：
```ts
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item);
}

// becomes //

for (var _i = 0; _i < someArray.length; _i++) {
    var item = someArray[_i];
    console.log(item);
}
```

你可以看到，使用`for...of`让目标更清晰，并且也减少你要编写的代码数量（和你要声明的变量名字）。

### 限制

如果你目标是 ES6 或者前面，生产的代码假设属性`length`存在于对象，则对象可以通过数字索引，比如`obj[2]`。因此它对于遗留的 JS 引擎只支持`string`和`array`。

如果 TypeScript 可以看到你不实用一个数组或者字符串，他将给你一个清晰的错误“不是一个数组类型或者字符串类型”
```ts
let articleParagraphs = document.querySelectorAll("article > p");
// Error: Nodelist is not an array type or a string type
for (let paragraph of articleParagraphs) {
    paragraph.classList.add("read");
}
```

只对你知道是数组或者字符串的东西使用`for...of`。注意这个限制可能在未来的 TypeScript 版本移除。

### 总结

你可能惊喜于你将会遍历多少次数组的元素。下一次你会发现你自己这么做，给`for...of`一个生路。你可能只让下一个审阅你的代码的人开心。