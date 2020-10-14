# noImplocotAny

有一些东西不能被推断或者推断他们可能导致不期待的错误。一个很好的例子是函数参数。如果你不声明他们，什么是应该有效和不应该有效是不清晰的：
```ts
function log(someArg) {
  sendDataToServer(someArg);
}

// What arg is valid and what isn't?
log(123);
log('hello world');
```

如果你不声明一些函数参数，TypeScript 假设`any`并继续前进。对于这种场景，这基本上关闭了类型检测，这可能是 JavaScript 开发者期待的。但是这可能捕获想要高安全的人。因此，有一个选项`noImplicitAny`，当打开的时候，将会标志无法推断类型的场景。
```ts
function log(someArg) { // Error : someArg has an implicit `any` type
  sendDataToServer(someArg);
}
```
当然，你可以继续前进并注释：
```ts
function log(someArg: number) {
  sendDataToServer(someArg);
}
```

当然，如果你不需要安全性，可以明确标记它为`any`：
```ts
function log(someArg: any) {
  sendDataToServer(someArg);
}
```