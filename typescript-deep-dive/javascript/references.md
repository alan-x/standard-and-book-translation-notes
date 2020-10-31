# 引用

除了字面量，JavaScript 中的任何对象（包括函数，数组，正则等）都是引用。这意味着下面：

### 操作跨域所有的引用

```ts
var foo = {};
var bar = foo; // bar is a reference to the same object

foo.baz = 123;
console.log(bar.baz); // 123
```

### 相等是针对索引
```ts
var foo = {};
var bar = foo; // bar is a reference
var baz = {}; // baz is a *new object* distinct from `foo`

console.log(foo === bar); // true
console.log(foo === baz); // false
```