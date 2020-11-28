[已校对]
# 声明空间

在 TypeScript 中有两种声明类型：变量声明空间和类型声明空间。这些概念将会在下面探索。

### 类型声明空间

类型声明空间包含可以用用于类型声明的东西，比如，下面是一些类型声明：
```ts
class Foo {};
interface Bar {};
type Bas = {};
```

这意味着你可以使用`Foo`，`Bar`，`Bas`，等作为类型声明，比如：
```ts
var foo: Foo;
var bar: Bar;
var bas: Bas;
```

注意，尽管你有`interface Bar`，你不能使用它作为变量，因为它不贡献给变量声明空间。这显示在下面：
```ts
interface Bar {};
var bar = Bar; // ERROR: "cannot find name 'Bar'"
```
它说`cannot find name`的原因是因为名字`Bar`没有定义在声明空间。这带领我们去下一个主题“变量声明空间”

### 变量声明空间

变量声明空间包含你可以用作一个变量的东西。我们说`class Foo`贡献了类型`Foo`到类型声明空间。猜猜看？它也贡献了一个变量`Foo`到变量声明空间，如下所示：
```ts
class Foo {};
var someVar = Foo;
var someOtherVar = 123;
```

这很棒，因为有时候，你想要传递类作为变量。记住：

- 我们可以使用类似`interface`的只在类型声明空间的东西作为一个变量。

同样的，你使用`var`声明的东西，只在变量声明空间，并且不能用作类型声明：
```ts
var foo = 123;
var bar: foo; // ERROR: "cannot find name 'foo'"
```