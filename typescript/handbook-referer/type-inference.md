在 TypeScript，有很多地方类型接口用于提供类型信息，当没有明确的类型宣告的时候。比如，在这个代码：
```
let x = 3;
//  ^ = let x: number
```

`x`变量的类型推断为`number`。这种类型的推断发生在初始化变量和成员，设置参数默认值，和决定函数返回类型的时候。

在大部分场景，类型推断很直接，在下面的章节，我们将探索一些类型推断的细节。

### 最常见的类型

当一个类型推断发生在多个表达式，这些表达式好的类型用于计算一个“最常见类型”。比如：
```
let x = [0, 1, null];
//  ^ = let x: (number | null)[]
```

为了推断前面例子的`x`类型，我们必须思考每一个数组元素的类型。这里我们给数组挂怒数组类型的两个选择：`number`和`null`。最常见类型算认为每一个候选人类型，并选择兼容其他候选类型的类型。

因为最常见类型从提供的候选类型选择，有时候类型共享相同的结构，但是没有一个类型是所有候选类型的父类型。比如：
```
let zoo = [new Rhino(), new Elephant(), new Snake()];
//    ^ = let zoo: (Rhino | Elephant | Snake)[]
```

理想中，我们可能想要`zoo`被推断为`Animal[]`，但是因为数组中没有对象是严格的`Animal`，我们没办法推断数组元素类型。为了修正这个，明确提供类型，当没有一个类型是所有其他候选类型的父类型：
```
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
//    ^ = let zoo: Animal[]
```
当没有最常见类型发现的时候，结果推断是联合数组类型，`(Rhino | Elephant | Snake)[]`

### 上下文类型

类型推断也工作在 TypeScript 某些场景的的“另一个方向”。这也被称作“上下文类型”。上下文类型出现在一个额表达式的类型通过他的为止被暗示的时候。比如：
```
window.onmousedown = function (mouseEvent) {
  console.log(mouseEvent.button); //<- OK
  console.log(mouseEvent.kangaroo); //<- Error!
};

```

这里，TypeScript 类型检查器使用`Window.onmousedown`函数的类型去推断函数表达式赋值右手边的类型。当它这么做的时候，它可以推断`mouseEvent`参数的类型，它包含`button`属性，但是没有`kangaroo`属性。

TypeScript 在其他上下文也足够聪明去推断类型：
```
window.onscroll = function (uiEvent) {
  console.log(uiEvent.button); //<- Error!
};

```

基于前面的函数被赋值给`Window.onscroll`的事实，TypeScript 知道`uiEvent`是一个[UIEvent]()，并不是类似前面例子的[MouseEvent]()。`UIEvent`对象不包含一个`button`属性，因此 TypeScript 将抛出一个错误。

如果函数不再一个上下文的类型为止，函数的参数将会是`any`类型。没有错误将会被提出（除非你使用`--noImplicitAny`选项）：
```
const handler = function (uiEvent) {
  console.log(uiEvent.button); //<- OK
};

```

我们也可以明确提供类型信息给函数的参数去覆盖 any 上下文类型：
```
window.onscroll = function (uiEvent: any) {
  console.log(uiEvent.button); //<- Now, no error is given
};
```

然而，这个代码将会记录`undefined`，因为`uiEvent`没有叫做`button`的属性。

上下文类型应用在很多场景，常见的例子包括函数调用参数，赋值的右侧，类型断言，对象恒源和数字字面量，和返回语句。上下文类型也以最常见类型表现为一个候选者类型。比如：
```
function createZoo(): Animal[] {
  return [new Rhino(), new Elephant(), new Snake()];
}

```
在这个例子，最常见类型有一个集合的四种候选：`Animal`，`Rhino`，`Elephant`，`Snake`。这些，`Animal`可以通过最常见类型算法被选择。