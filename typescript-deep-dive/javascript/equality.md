# 相等

### 相等

关于 JavaScript 有一点需要小心，那就是`==`和`===`的不同。因为 javaScript 尝试避免编程错误，`==`尝试在两个变量执行类型转换。比如，转化一个字符串为一个数字，因此你可以如下比较一个数字：
```ts
console.log(5 == "5"); // true   , TS Error
console.log(5 === "5"); // false , TS Error
```

然而，JavaScript 做的选择不总是理想的。比如，在下面的例子，第一个语句是 false，因为`""`和`"0"`都是字符串，很明显不相等。然而，在第二个场景，`0`和空字符串（`""`）都是假值（比如，行为类似`false`），因此在`==`下是相等的。当你使用`===`的时候，这两个语句都是 false。
```ts
console.log("" == "0"); // false
console.log(0 == ""); // true

console.log("" === "0"); // false
console.log(0 === ""); // false
```

> 和`==`vs`===`类似的还有`!=`vs`!==`。

因此，专家提示：总是使用`===`和`!==`，除了 null 检测，我们将在稍后覆盖。

### 结构上相等

如果你想要比较两个对象在结构上相等，`==`/`===`不够，比如：
```ts
console.log({a:123} == {a:123}); // False
console.log({a:123} === {a:123}); // False
```
执行这类检查查看[deep-equal]()npm 包，比如：
```ts
import * as deepEqual from "deep-equal";

console.log(deepEqual({a:123},{a:123})); // True
```

然而，通常你不需要深度检测，你需要的是通过一些`id`检测，比如：
```ts
type IdDisplay = {
  id: string,
  display: string
}
const list: IdDisplay[] = [
  {
    id: 'foo',
    display: 'Foo Select'
  },
  {
    id: 'bar',
    display: 'Bar Select'
  },
]

const fooIndex = list.map(i => i.id).indexOf('foo');
console.log(fooIndex); // 0
```