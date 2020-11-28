[已校对]
# 函数参数

如果你有一个函数，接受很多参数，或者相同类型的参数，则你可能想要去考虑改变函数去接受一个对象。

假设下面的函数：
```ts
function foo(flagA: boolean, flagB: boolean) {
  // your awesome function body 
}
```
这类函数定义很容易正确调用，比如`foo(flagB, flagA)`，你将不会从编译器获取任何帮助。

相反，转化为一个函数接受一个对象：
```ts
function foo(config: {flagA: boolean, flagB: boolean}) {
  const {flagA, flagB} = config;
  // your awesome function body 
}
```

现在函数调用将会像`foo({flagA, flagB})`，这样更容易发现错误和和代码审阅。

> 注意：如果你的函数足够简单，你不期待更多打扰，自由忽略这个建议。