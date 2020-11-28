[已校对]
# 非 React JSX

TypeScript 提供你以类型安全的方式使用 React 之外的 JSX 的能力。下面列出了可自定义的点，但是注意注意这只对高级 UI 框架作者：

- 你可以禁用`react`风格生成，通过使用`"jsx" : "preserve"`选项。这意味着 JSX 将原样生成，然后你可以使用你自己的自定义翻译器去翻译 JSX 部分。

- 使用`JSX`全局模块：
    - 你可以控制哪些 HTML 标签可以用，还有如何通过自定义`JSX.IntrinsicElements`接口成员做类型检测。
    - 当时你用组件：
        - 你可以控制哪些`class`必须继承组件，通过自定义默认`interface ElementClass extends React.Component<any, any> { }`声明
        - 你可以控制哪些属性用于属性的类型检测（默认是`props`），通过自定义`declare module JSX { interface ElementAttributesProperty { props: {}; } }`声明。

### `jsxFactory`

传递`--jsxFactory <JSX factory Name>`和`--jsx react`允许使用一个和`React`默认不同的 JSX 工厂。

新的工厂名字将会用于`createElement`函数的调用。

#### 例子

```ts
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

编译为：
```tsx
tsc --jsx react --reactNamespace jsxFactory --m commonJS
```
生成：
```ts
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```

#### `jsx`指令

你甚至可以使用`jsxPragma`为每个文件指定一个不同的`jsxFactory`，比如：
```ts
/** @jsx jsxFactory */
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

使用`--jsx react`，这个文件将会使用 jsx 指令指定的工厂去生成。