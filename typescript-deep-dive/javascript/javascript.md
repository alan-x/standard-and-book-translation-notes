[已校对]
# JavaScript

过去（还会继续）JavaScript 的一些语法有很多的争议。TypeScript 和他们不一样，因为你的 JavaScript 是 TypeScript。这是一张图：

![JavaScript is TypeScript](https://raw.githubusercontent.com/basarat/typescript-book/master/images/venn.png)


然而，这意味着你需要学习 JavaScript（好消息是你只需要学习 JavaScript）。TypeScript 只是标准化了你为 JavaScript 提供文档的所有方式。

- 只是提供你一个新的语法不帮助捕获 bug - 但是可能帮助你编写更清晰/更少的 bug（比如，CoffieScript）。
- 创建一个新的语言抽象，让你远离运行时和社区 - 但是可能帮助你更简单的落地，如果你已经很熟悉（比如，Dart-更接近 Java/C# 开发者）。

TypeScript 只是有文档的 JavaScript

> JSNext 可以接受交互 - JS 下一个版本的每一个提案不是都实际加入到浏览器。TypeScript 只添加到达[stage 3](https://tc39.es/process-document/)的提案。


### 让 JavaScript 更好

TypeScript 将会尝试从无法工作的 JavaScript 中保护你（因此，你不需要记住这个东西）。
```ts
[] + []; // JavaScript will give you "" (which makes little sense), TypeScript will error

//
// other things that are nonsensical in JavaScript
// - don't give a runtime error (making debugging hard)
// - but TypeScript will give a compile time error (making debugging unnecessary)
//
{} + []; // JS : 0, TS Error
[] + {}; // JS : "[object Object]", TS Error
{} + {}; // JS : NaN or [object Object][object Object] depending upon browser, TS Error
"hello" - 1; // JS : NaN, TS Error

function add(a,b) {
  return
    a + b; // JS : undefined, TS Error 'unreachable code detected'
}
```

基本上，TypeScript 是对齐的 JavaScript。只是做的比其他没有类型信息的 linter 更高。


### 你依旧需要学习 JavaScript 

也就是说 TypeScript 非常务实，实际上你在写 JavaScript。因此关于 JavaScript，有些东西你依旧需要知道，为了不让自己措手不及。在下面讨论他们

> 注意：TypeScript 是 JavaScript 的超级集合。只是多了可以被编译器/DIE 使用的文档。