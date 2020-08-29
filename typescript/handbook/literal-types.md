### 字面量类型

字面量是集合类型更具体的子类型。这意味着`"Hello World"`是一个`string`，但是`string`不是`"Hello World"`，在类型系统中。

现在在 TypeScript 有三种集合的字面量类型：字符串，数字，和布尔；通过使用字面量你可以让字符串、数字、或者布尔有一个更明确的值。

### 字面量紧缩

当你通过`var`或者`let`声明一个变量。你在告诉编译器这是这个变量改变他的内容的机会。相反，使用`const`去声明一个变量将会告诉 TypeScript 这个对象将不会改变。

```
// We're making a guarantee that this variable
// helloWorld will never change, by using const.

// So, TypeScript sets the type to be "Hello World" not string
const helloWorld = "Hello World";

// On the other hand, a let can change, and so the compiler declares it a string
let hiWorld = "Hi World";
```

从无限个潜在情况（有无限多中情况的字符串值）到更小的，有限数量的有限场景（在案例 1 中是`helloWorld`）叫做紧缩。

### 字符串字面量类型

实践中，字符串字面量类型和联合类型，类型守卫，和类型别名结合的很好。你可以使用这些特性和字符串获得类似枚举的行为。

```
type Easing = "ease-in" | "ease-out" | "ease-in-out";

class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") {
      // ...
    } else if (easing === "ease-out") {
    } else if (easing === "ease-in-out") {
    } else {
      // It's possible that someone could reach this
      // by ignoring your types though.
    }
  }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy");
Argument of type '"uneasy"' is not assignable to parameter of type 'Easing'.
```

你可以传递任何允许的三个字符串，但是任何其他字符串将会得到一个错误
```
Argument of type '"uneasy"' is not assignable to parameter of type '"ease-in" | "ease-out" | "ease-in-out"'

```

字符串字面量类型可以使用相同的方式去区分重载：
```
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
  // ... code goes here ...
}
```

### 数字字面量类型
TypeScript 当然也有数字字面量类型，和前面的字符字面量表现一致：
```
function rollDice(): 1 | 2 | 3 | 4 | 5 | 6 {
  return (Math.floor(Math.random() * 6) + 1) as 1 | 2 | 3 | 4 | 5 | 6;
}

const result = rollDice();
```

他们的常见场景是用来描述配置值：
```
interface MapConfig {
  lng: number;
  lat: number;
  tileSize: 8 | 16 | 32;
}

setupMap({ lng: -73.935242, lat: 40.73061, tileSize: 16 });
```

### 布尔字面量类型

TypeScript 当然也有布尔字面量类型。你可能使用这些去约束属性值是相关联的对象值。
```
interface ValidationSuccess {
  isValid: true;
  reason: null;
};

interface ValidationFailure {
  isValid: false;
  reason: string;
};

type ValidationResult =
  | ValidationSuccess
  | ValidationFailure;
```