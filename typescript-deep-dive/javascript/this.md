# this

函数内任何`this`关键字的访问都受控于函数实际被调用的方式。这通常被称为“调用上下文”

这是一个例子：
```ts
function foo() {
  console.log(this);
}

foo(); // logs out the global e.g. `window` in browsers
let bar = {
  foo
}
bar.foo(); // Logs out `bar` as `foo` was called on `bar`
```

因此，小心`this`的使用。如果你想要在类形式中分离`this`和调用栈，可以使用箭头函数，[之后有更多内容]()。