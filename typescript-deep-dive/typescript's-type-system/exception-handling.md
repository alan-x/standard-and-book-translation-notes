[已校对]
# 异常处理

JavaScript 有一个`Error`类，你可以用于处理异常。你使用`throw`关键字抛出一个错误。你可以使用`try`/`catch`块对去捕获它，比如：
```ts
try {
  throw new Error('Something bad happened');
}
catch(e) {
  console.log(e);
}
```

### 错误子类型

除了内建的`Error`类，还有一些额外的内建错误类，继承于`Error`，JavaScript 运行时可以抛出：

#### RangeError

创建一个实例，表示一个错误，当数字变量或者参数在它的有效范围之外的时候出现。
```ts
// Call console with too many arguments
console.log.apply(console, new Array(1000000000)); // RangeError: Invalid array length
```

#### ReferenceError

创建一个实例表示一个错误，当关联一个无效的关联的时候出现。
```ts
'use strict';
console.log(notValidVar); // ReferenceError: notValidVar is not defined
```

#### SyntaxError

创建一个实例，表示一个错误，当转化代码不是有效 JavaScript 的时候出现。
```ts
1***3; // SyntaxError: Unexpected token *
```

#### TypeError

创建一个实例，表示一个错误，当一个变量或者参数不是一个有效的类型的时候出现。
```ts
('1.2').toPrecision(1); // TypeError: '1.2'.toPrecision is not a function
```

#### URIError
创建一个实例表示一个错误，当`encodeURI()`或者`decodeURI()`被传递进一个无效的参数的时候
```ts
decodeURI('%'); // URIError: URI malformed
```

### 总是使用`Error`

JavaScript 开发者新手有时候只是抛出一个原生字符串，比如
```ts
try {
  throw 'Something bad happened';
}
catch(e) {
  console.log(e);
}
```

不要这么做。`Error`对象的基础好处是能够通过`stack`属性自动保持对创建和起源的跟踪。

原始字符串导致一个非常痛苦的调试体验并且复杂化日志分析。

### 你不需要`throw`一个错误

传递一个`Error`是没关系的。这在 Node.js 回调风格代码中很方便，它接受回调，这个回调的第一个参数是一个错误对象：
```ts
function myFunction (callback: (e?: Error)) {
  doSomethingAsync(function () {
    if (somethingWrong) {
      callback(new Error('This is my error'))
    } else {
      callback();
    }
  });
}
```

### 异常的例子

`Exceptions should be exceptional`是计算机科学中的常见说法。这是一些为什么这对 JavaScript（和 TypeScript）是真的原因。

#### 不清楚是那里抛出的

考虑下面的代码块：
```ts
try {
  const foo = runTask1();
  const bar = runTask2();
}
catch(e) {
  console.log('Error:', e);
}
```

下一个开发者不知道哪个函数可能抛出错误。重新查看代码的人不能在不阅读 task1/ ask2 和其他可能调用的函数情况下知道。

#### 让优雅处理变得困难

你可以尝试让它优雅，使用明确的捕获每一个可能抛出的东西：
```ts
try {
  const foo = runTask1();
}
catch(e) {
  console.log('Error:', e);
}
try {
  const bar = runTask2();
}
catch(e) {
  console.log('Error:', e);
}
```

但是如果你需要从第一个任务传递东西到第二个，代码会变得混乱：（注意`foo`操作需要`let`+明确需要声明，因为它不能推断出`runTask1`的返回）：
```ts
let foo: number; // Notice use of `let` and explicit type annotation
try {
  foo = runTask1();
}
catch(e) {
  console.log('Error:', e);
}
try {
  const bar = runTask2(foo);
}
catch(e) {
  console.log('Error:', e);
}
```

#### 没有在类型系统中很好的表示

假设函数：
```ts
function validate(value: number) {
  if (value < 0 || value > 100) throw new Error('Invalid value');
}
```

为这些场景使用`Error`是坏主意，因为它不存在于类型定义（是`(value:number) => void`）。创建一个验证方法可能是一个更好的方式：
```ts
function validate(value: number): {error?: string} {
  if (value < 0 || value > 100) return {error:'Invalid value'};
}
```

现在它存在于类型系统。

> 除非你想要去以非常通用的方式去处理错误（简单/捕获所有），不要抛出一个错误。