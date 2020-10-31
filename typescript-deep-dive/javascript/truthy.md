# 真值
JavaScript 有一个`truthy`的概念。比如，执行为`true`的东西将在某个点（比如`if`条件和布尔`&&``||`操作符）。下面的东西在 JavaScript 是真值。一个例子是非`0`的任何东西，比如：
```ts
if (123) { // Will be treated like `true`
  console.log('Any number other than 0 is truthy');
}
```

不是真值的东西都叫做`falsy`。

这是一个便捷表格，你可以索引：


| 变量类型 | 当他是假值 | 当它是真值 |
| --- | --- | --- |
| `boolean` | `false` | `true` |
| `string` | `''`（空字符串） | 任何其他字符串 |
| `number` | `0``NaN` | 任何其他数字 |
| `null` | 总是 | 永远不会 |
| `undefined` | 总是 | 永不 |
| 任何其他对象，包括`{}`| 永不 | 总是 |


### 变得明确

> `!!`模式

通常它帮助去明确目标去对待值为`boolean`并转化它到一个真正的布尔（`true`|`false`中的一个）。你可以简单转化值到一个真正的布尔，通过前置`!!`，比如`!!foo`。他只是使用`!`两次。第一个`!`转化变量（这里是`foo`）为一个布尔值，但是点到逻辑（真值-`!`>`false`，假值-`!`>`true`）。

在很多地方使用这个模式很常见，比如：
```ts
// Direct variables
const hasName = !!name;

// As members of objects
const someObj = {
  hasName: !!name
}

// e.g. in ReactJS JSX
{!!someName && <div>{someName}</div>}
```