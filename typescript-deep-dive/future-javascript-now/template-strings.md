[已校对]
# 模板字符串

语法上，这些字符串使用反引号（比如 `）而不是单引号(')或者双引号(")。模板字符串的动机有三个：

- 字符串插值
- 多行字符串
- 标签模板

### 字符串插值

另一个常见使用场景是当你想要生成一些不是静态字符串+一些变量的字符串的时候。为了这个，你需要一些模版逻辑，这也是模板字符串的名字的由来。他们也因此被官方重命名为模板字符串。这里是你之前可能生成一个 html 字符串的方法：
```ts
var lyrics = 'Never gonna give you up';
var html = '<div>' + lyrics + '</div>';
```

现在，使用模板字面量，你可以：
```ts
var lyrics = 'Never gonna give you up';
var html = `<div>${lyrics}</div>`;
```

注意插值内（`${`和`}`）的任何占位符被对待为 JavaScript 表达式并如下求值，比如，你可以做花哨的数学：
```ts
console.log(`1 and 1 make ${1 + 1}`);
```

### 多行字面量

甚至想要放置一个新行在一个 JavaScript 字符串？可能你想要嵌入一些诗歌？你可能需要去避开字面量新行，使用我们罪行还的转义字符`\`，然后放置新行到字符串`\n`在下一行。这显示在下面：
```ts
var lyrics = "Never gonna give you up \
\nNever gonna let you down";
```
使用 TypeScript，你可以使用一个模板字符串：
```ts
var lyrics = `Never gonna give you up
Never gonna let you down`;
```

### 标记模板

你可以放置一个函数（叫做一个`tag`）在模板字符串前面，它可以在模板字符串处理之前处理，加上所有占位表达式的值，并返回一个结果。一些笔记：
- 所有的静态字面量都作为一个数组传递给第一个参数
- 所有的占位表达式的值都作为剩余参数传递。通常你只是使用功能剩余参数去妆化这些到一个数组。

这时候一个例子，我们有一个标签函数（命名为`htmlEscape`），转义占位符的所有 html ：
```ts
var say = "a bird in hand > two in the bush";
var html = htmlEscape `<div> I would just like to say : ${say}</div>`;

// a sample tag function
function htmlEscape(literals: TemplateStringsArray, ...placeholders: string[]) {
    let result = "";

    // interleave the literals with the placeholders
    for (let i = 0; i < placeholders.length; i++) {
        result += literals[i];
        result += placeholders[i]
            .replace(/&/g, '&amp;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#39;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;');
    }

    // add the last literal
    result += literals[literals.length - 1];
    return result;
}
```

> 注意：你可以声明`placeholders`为任何`[]`。不管你怎么解释，TypeScript 将执行类型检测去确保占位符用于调用标签匹配声明。比如，入股你期待处理`string`或者`number`，你可以声明`...placeholders:(string | number)[]`

### 生成的 JS

对于 ES6 编译目标之前，代码非常简单。多行字符串成为转义的字符串。字符串插值成为字符串链接。标签模板成为函数调用。

### 总结

多行字符串和字符串插值在任何语言都是好东西。好在现在你可以在你的 JavaScript（感谢 TypeScript）中使用。标签模板允许你去创建威力强大的字符串工具。