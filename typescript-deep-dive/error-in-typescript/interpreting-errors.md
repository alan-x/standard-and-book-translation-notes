# 解释错误

因此 TypeScript 是非常注重面向开发者帮助的编程语言，当发生错误的时候，他的错误信息尝试成为最大的帮助。这可能导致有一点信息超载，对于不熟悉编译器的用户来说不是很有帮助。

来看看 IDE 中的一个例子，分析阅读一个错误信息的过程。
```ts
type SomethingComplex = {
  foo: number,
  bar: string
}
function takeSomethingComplex(arg: SomethingComplex) {
}
function getBar(): string {
  return 'some bar';
}

//////////////////////////////////
// Example error production
//////////////////////////////////
const fail = {
  foo: 123,
  bar: getBar
};

takeSomethingComplex(fail); // TS ERROR HAPPENS HERE
```

这个例子显示了一个常见程序员的错误，他们调用一个函数失败了（`bar: getBar`应该是`bar: getBar()`）。幸运的是，这个错误立马被 TypeScript 捕获，因为它不满足类型要求。

### 错误类别

存在两种类别的 TypeScript 错误信息（简洁的和详细的）。

#### 简洁的

简洁错误信息的目的是提供常规编译器错误数字和信息的描述。简洁信息的例子看起来像这样：
```ts
TS2345: Argument of type '{ foo: number; bar: () => string; }' is not assignable to parameter of type 'SomethingComplex'.
```

这相当自解释。然而，它不提供关于为什么这个错误发生的深入解析。这也是详细错误信息的目的。

#### 详细的

详细版本的例子看起来像这样：
```ts
[ts]
Argument of type '{ foo: number; bar: () => string; }' is not assignable to parameter of type 'SomethingComplex'.
  Types of property 'bar' are incompatible.
    Type '() => string' is not assignable to type 'string'.
```

详细错误信息的目标是知道用户到为什么一些错误（这个场景中是类型不兼容）发生的原因。第一行和简洁版本一样，后边跟着一连串。你应该阅读这个串，就像一些列开发者问题`WHY？`的响应，比如：
```ts
ERROR: Argument of type '{ foo: number; bar: () => string; }' is not assignable to parameter of type 'SomethingComplex'.

WHY? 
CAUSE ERROR: Types of property 'bar' are incompatible.

WHY? 
CAUSE ERROR: Type '() => string' is not assignable to type 'string'.
```

所以根本原因是，
- 对于属性`bar`
- 存在一个函数`() => string`，但是它期待一个`string`

这应该帮助开发者为`bar`属性修复这个 bug（他们忘记去调用`()`函数）。

### 他是如何显示在 IDE 的提示的

IDE 通常在提示工具条的`succinct`版本之后显示`detailed`，就像下面：
![](https://raw.githubusercontent.com/basarat/typescript-book/master/images/errors/interpreting-errors/ide.png)

- 你通常在你的脑袋里组织`WHY?`链去阅读`detailed`
- 你使用简约版本，如果你想要搜索类似的错误（使用`TSXXXX`错误码或者错误信息的一部分）