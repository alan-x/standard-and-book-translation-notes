[校对]
# 剩余参数

剩余参数（最后一个参数记为`...argumentName`）允许你在你的函数快速接受多个参数，并类似数组的得到他们。这展示在下面的例子。
```ts
function iTakeItAll(first, second, ...allOthers) {
    console.log(allOthers);
}
iTakeItAll('foo', 'bar'); // []
iTakeItAll('foo', 'bar', 'bas', 'qux'); // ['bas','qux']
```

剩余参数可以被用于任何函数`function`/`()=>`/`class member`