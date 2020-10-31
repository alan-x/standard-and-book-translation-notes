# Null 和 Undefined

> [yourube 上这个主题的免费视频]()

JavaScript（和通过 TypeScript 扩展的）有两个底层类型：`null`和`undefined`。他们是为了不同的东西：

- 没有被初始化的东西：`undefined`
- 现在不可用的东西：`null`

### 检测两者

实际上你需要去处理两者。有趣的是，在 JavaScript 中，使用`==`，`null`和`undefined`是相等的：
```ts
// Both null and undefined are only `==` to themselves and each other:
console.log(null == null); // true (of course)
console.log(undefined == undefined); // true (of course)
console.log(null == undefined); // true


// You don't have to worry about falsy values making through this check
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```

推荐`==null`去检测`undefined`或者`null`。你通常不想要去区别两者：
```ts
function foo(arg: string | null | undefined) {
  if (arg != null) {
    // arg must be a string as `!=` rules out both null and undefined. 
  }
}
```

> 你也可以执行`== undefined`，但是`== null`更方便/简短。

一个异常，根级别的`undefined`值我们将在下面讨论

### 检测根级别的 undefined

还记得我说应该怎样使用`==null`吗？当然你记得（因为我刚说过）。不要为根级别的东西使用它。在严格模式，如果你使用`foo`并且`foo`是 undefined，你将得到一个`ReferenceError`异常，整个调用栈都会展开。

> 你应该使用严格模式...实际上，TS 编译器将会为你插入它，如果你使用模块，更多细节将会在这本书的后面提到，你不需要明确知道它。

因此，使用`typeof`检测一个变量是否定义在一个全局级别：
```ts
if (typeof someglobal !== 'undefined') {
  // someglobal is now safe to use
  console.log(someglobal);
}
```

### 明确限制`undefined`使用

因为 TypeScript 给你机会去分别记录你的结构和文档，而不是类似的东西：
```ts
function foo(){
  // if Something
  return {a:1,b:2};
  // else
  return {a:1,b:undefined};
}
```
你应该使用类型声明：
```ts
function foo():{a:number,b?:number}{
  // if Something
  return {a:1,b:2};
  // else
  return {a:1};
}
```

### Node 风格回调

Node 风格回调函数（比如，`(err,somethingElse)=>{ /* something */ }`）被调用的时候，如果没有错误，`error`会被设置为`null`。你通常只为这个使用一个真值检测：
```ts
fs.readFile('someFile', 'utf8', (err,data) => {
  if (err) {
    // do something
  } else {
    // no error
  }
});
```
当创建你自己的 API 的时候，为这个场景统一使用`null`是可以的。真诚的说，对于你自己的 API，你应该看看 Promise，你这个场景，你其实不需要缺省错误值（你使用`.then`vs`.catch`处理他们）。

### 不要使用`undefined`表示有效性

比如有一个糟糕的函数：
```ts
function toInt(str: string) {
  return str ? parseInt(str) : undefined;
}
```

像这样写更好：
```ts
function toInt(str: string): { valid: boolean, int?: number } {
  const int = parseInt(str);
  if (isNaN(int)) {
    return { valid: false };
  }
  else {
    return { valid: true, int };
  }
}
```

### JSON 和序列化

JSON 标准支持`null`编码，但是不支持`undefined`。当 JSON 编码一个对象，它的一个属性是`null`的时候，这个属性将会和它的 null 值被包含，然而，一个属性如果是`undefined`，将会被完全排除。
```ts
JSON.stringify({willStay: null, willBeGone: undefined}); // {"willStay":null}
```

作为一个结果，基于 JSON 的数据库可能支持`null`值，但是不支持`undefined`值。因为设置为`null`的属性被编码，你可以清晰传递这个目的，通过设置他的值为`null`，在便哈和传输这个对象到远程存储之前。

设置属性值为 undefined 可以节约存储和传输损耗，因为属性名将不会被编码。然而，这可以使清除值和缺省值的语义复杂化。

### 最后的想法

TypeScript 团队不使用`null`：[TypeScript 编码指南]()并且它不会到这任何问题。Douglas Crockford 认为[`null`是一个很坏的主意]()并且我们应该只使用`undefined`。

然而，NodeJS 风格的代码库为 Error 参数使用`null`作为标准，因为它表示`Something is currently unavailable`。我个人不关心两个的区别，因为大部分项目使用不同见解的库，都可以使用`== null`排除。