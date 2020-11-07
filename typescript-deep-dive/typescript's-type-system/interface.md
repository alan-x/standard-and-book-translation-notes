[已校对]
# 接口

### 接口

接口对 JS 运行时没有影响，TypeScript 接口对声明变量结构很有用。

下面的两个是相同的声明，第一个使用内联声明，第二个使用接口：
```ts
// Sample A
declare var myPoint: { x: number; y: number; };

// Sample B
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;
```

然而，例子B 的美丽在于如果一个人基于`myPoint`创作了一个库，添加了一些新的成员，他们可以简单的添加到`myPoint`存在的声明。
```ts
// Lib a.d.ts
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;

// Lib b.d.ts
interface Point {
    z: number;
}

// Your code
var myPoint.z; // Allowed!
```

这是因为 TypeScript 中的接口是开放的。这是 TypeScript 一个非常的原则，它允许你去使用接口模拟 JavaScript 的扩展性。

### 类可以实现接口

如果你想要使用一个类，并且这个类型必须遵循一个对象结构，这个对象结构被某个人定义在一个`interface`，你可以使用`implements`关键字去保证兼容性：

```ts
interface Point {
    x: number; y: number;
}

class MyPoint implements Point {
    x: number; y: number; // Same as Point
}
```

基本上，在`implements`的存在的时候，外部的`Point`接口的任何改变都会导致你的代码库的一个编译错误，你可以简单的保证他们同步：
```ts
interface Point {
    x: number; y: number;
    z: number; // New member
}

class MyPoint implements Point { // ERROR : missing member `z`
    x: number; y: number;
}
```

注意`implements`约束类实例的结构，比如：
```ts
var foo: Point = new MyPoint();
```

类似`foo: Point = MyPoint`之类的不是同一个东西。

### 提示

#### 不是每一个接口都能简单实现

接口设计用于声明存在于 JavaScript 中的任何任意的疯狂结构。

考虑下面的接口，有一个叫做`new`：
```ts
interface Crazy {
    new (): {
        hello: number
    };
}
```
你基本上可以有类似的一些东西：
```ts
class CrazyClass implements Crazy {
    constructor() {
        return { hello: 123 };
    }
}
// Because
const crazy = new CrazyClass(); // crazy would be {hello:123}
```


你可以在这里使用接口声明所有疯狂的 JS，甚至在 TypeScript 安全的使用他们。不意味着你可以使用 TypeScript 类去实现他们。