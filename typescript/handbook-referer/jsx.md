JSX

[JSX]()是一个可嵌入的类似 XML 的语法。他的目的时候转化为有效的 JavaScript，尽管转化信息的语义是实现特定的。JSX 变得受欢迎是因为[React]()框架，但是也看到了其他实现。TypeScript 支持嵌入，类型检查和编译 JSX 到 JavaScript。


### 基本使用

为了使用 JSX，你必须做两件事：

1. 使用`.tsx`扩展命名你的文件。
2. 启用`jsx`选项。

TypeScript 有三种 JSX 模式：`preserve`，`react`，和`react-native`。这些模式只影响发送阶段 - 类型检查是没有影响的。`preserve`模式将会保留 JSX 为输出的一部分，被更深层的转化步骤消费（比如：[Babel]()）。此外，输出将会有一个`.jsx`文件扩展。`react`模式将会发送`React.createElement`，不需要在使用之前经过一个 JSX 转化，并且输出将会有一个`.js`文件扩展。`react-native`模式和`preserver`一样，它保留所有的 JSX，但是书痴将会有一个`.js`文件扩展。

| 模式 | 输入 | 输出 | 输出文件扩展 |
| --- | --- | --- | --- |
| preserve | <div /> | <div /> | .jsx |
| react | <div /> | React.createElement("div") | .js |
| react-native | <div /> | <div /> | .js |


你使用`--jsx`命令行标志或者在你的[tsconfig.json]()文件对应的选项执行这个模式。


注意：你可以使用`--jsxFactory`选项指定使用的 JSX 工厂函数，当目标是 react JSX 的时候（默认是`React.createElement`）。

### as 操作符
```
var foo = <foo>bar;
```
回忆一下如何编写类型断言：
```
var foo = <foo>bar;

```
这个断言变量`bar`有`foo`类型。因为 TypeScript 也为类型断言使用单括号，他和 JSX 语法一起将会导致某种转化困难。作为结果，TypeScript 不允许在`.tsx`中使用角括号类型断言。

因为前面的语法不能在`.tsx`文件中使用，一个替代的类型断言操作符可以使用：`as`。这个例子可以简单的使用`as`操作符语法。

```
var foo = bar as foo;

```

`as`操作符在`.ts`和`.tsx`文件，并且和角括号类型断言风格的行为一样。


### 类型检测

为了理解 JSX 的类型检测，你必须先理解内置元素和基于值的元素。一个 JSX 表达式`<expr />`，`expr`可能引用一些环境内置（比如，一个 DOM 环境的`div`，或者`span`）或者一个你创建的自定义元素。这因为两个原因很重要：
 
 1. 对于 React，内置元素被发送为字符串（`React.createElement("div")`），然而你创建的一个组件不是（`React.createElement(MyComponent)`）。

 2. 被传递进 JSX 元素的属性的类型应该不同。内置元素属性应该应该知道内置的，然而组件可能想要特定他们自己属性的集合。

 TypeScript 使用[React 相同的约定]()用来区分这些。一个内置元素总是以小写字符开始，并且一个基于值的元素总是使用大写字母开始。


 ### 内置元素

 内置元素基于特定`JSX.IntrinsicElements`接口之上。默认，如果这个接口没有指定，则任何内置元素将不会被检查。然而，如果这个接口是存在的，则内置元素的名字将作为`JSX.IntrinsicElements`接口的一个属性。比如：
 ```ts
 declare namespace JSX {
  interface IntrinsicElements {
    foo: any;
  }
}

<foo />; // ok
<bar />; // error
 ```

在前面的例子中，`<foo />`将会工作的很好，但是`<bar />将会导致一个错误，因为它没有被指定在`JSX.IntrinsicElements`。

注意：你也可以指定一个捕捉所有字符串索引在`JSX.IntrinsicElements`如下：
```
declare namespace JSX {
  interface IntrinsicElements {
    [elemName: string]: any;
  }
}

```

基于值的元素


基于值的元素简单的通过标识符在范围内查找：

```
import MyComponent from "./myComponent";

<MyComponent />; // ok
<SomeOtherComponent />; // error
```

有两种方式去定义一个基于值的元素：

1. 函数组件（FC）
2. 类组件

因为有两种类型的基于值的元素无法从 JSX 表达式中区分，首先 TS 尝试解析表达式为函数组件，使用重载解决方案。如果处理成功，则 TS 完成解析表达式到他的声明。如果值失败的解析为一个函数组件，TS 将尝试解析它为一个类组件。如果失败，TS 将报告一个错误。

### 函数组件

就像名字简易的，组件定义为一个 JavaScript 函数，他的第一个参数是一个`props`对象。TS 强制它返回一个可以赋值给`JSX.Element`的类型。
```ts
interface FooProp {
  name: string;
  X: number;
  Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name={prop.name} />;
}

const Button = (prop: {value: string}, context: { color: string }) => <button>
```

因为一个函数组件只是一个 JaaScript 函数，汗水重载也可以用在这里：
```ts
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
  home: JSX.Element;
}

interface SideProps extends ClickableProps {
  side: JSX.Element | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
  ...
}
```

注意：函数组件通常被认为是无状态函数组件（SFC），函数组件在最近版本的 react 中，也被认为是无状态的，`SFC`和他的别名`StatelessComponent`都被废弃了。

### 类组件

定义类组件的类型是可能的。然而，为了这么做必须理解两个新的术语：元素类类型和元素实例类型。

给定`<Expr />`，元素类类型是`Expr`的类型。因此在前面的例子中，如果`MyComponent`是一个 ES6 类，类类型将会是类的构造器，并且是静态的。如果`MyComponent`是一个工厂函数，类类型将会是那个函数。

一旦类类型被建立，实例类型被类类型的构造或者调用签名的返回类型的联合决定（取决于哪一个存在）。子啊一次，在一个 ES6 类的场景中，实例类型将会是类的实例的类型，在工厂函数的场景中，它将会是函数返回的值的类型。
```ts
class MyComponent {
  render() {}
}

// use a construct signature
var myComponent = new MyComponent();

// element class type => MyComponent
// element instance type => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {}
  };
}

// use a call signature
var myComponent = MyFactoryFunction();

// element class type => FactoryFunction
// element instance type => { render: () => void }
```
元素实例类型很有趣，因为它必须可以被赋值给`JSX.ElementClass`，或者它将会导致一个错误。通常`JSX.ElementClass`是`{}`，但是他可以被扩展去限制`JSX`只兼容适合的接口。
```ts
declare namespace JSX {
  interface ElementClass {
    render: any;
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} };
}

<MyComponent />; // ok
<MyFactoryFunction />; // ok

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {};
}

<NotAValidComponent />; // error
<NotAValidFactoryFunction />; // error
```

### 属性类型检测

属性类型检测的第一步是去决定元素属性的类型。内置和基于值的元素有一点不同。

对于内置元素，是`JSX.IntrinsicElements`的属性的类型。

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean };
  }
}

// element attributes type for 'foo' is '{bar?: boolean}'
<foo bar />;
```

对于基于元素的值，更加复杂一点。它取决于前面决定的元素实例类型上的属性的类型。使用哪个属性取决于`JSX.ElementAttributesProperty`。它应该被声明为单独的属性。属性的名字接下来将被使用。在 TypeScript 2.8，如果`JSX.ElementAttributesProperty`没有提供，类元素的构造器或者函数组件的调用的第一个参数将会被用于替代。
```ts
declare namespace JSX {
  interface ElementAttributesProperty {
    props; // specify the property name to use
  }
}

class MyComponent {
  // specify the property on the element instance type
  props: {
    foo?: string;
  };
}

// element attributes type for 'MyComponent' is '{foo?: string}'
<MyComponent foo="bar" />;
```ts
元素属性类型用于 JSX 中属性类型检测。可选和必须属性都是支持的：
```
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number };
  }
}

<foo requiredProp="bar" />; // ok
<foo requiredProp="bar" optionalProp={0} />; // ok
<foo />; // error, requiredProp is missing
<foo requiredProp={0} />; // error, requiredProp should be a string
<foo requiredProp="bar" unknownProp />; // error, unknownProp does not exist
<foo requiredProp="bar" some-unknown-prop />; // ok, because 'some-unknown-prop' is not a valid identifier
```

注意：如果属性名字不是一个有效的 JS 标识符（），如果没有找到元素属性类型，也不会认为是一个错误。

此外，`JSX.IntrinsicAttributes`接口可以用于指定被 JSX 框架使用的额外的属性，通常不用于组件的属性或者参数 - 比如`React`的`key`。甚至，通用的`JSX.IntrinsicClassAttributes<T>`类型可能用于指定类组件相同类型的额外属性（并且不是函数组件）。在这个类型，通用参数表示类属实例类型。在 React，这用于允许`ref`属性的`Ref<T>`。通常来说，这个接口所有大户型都是可选的，除非你想要爹 JSX 框架的用户需要在每一个标签提示一些属性。

扩展操作符也能用：
```ts
var props = { requiredProp: "bar" };
<foo {...props} />; // ok

var badProps = {};
<foo {...badProps} />; // error
```

### Children 类型检测

在 TypeScript 2.3，TS 引入 children 的的类型检测。children 在元素属性类型是一个特殊属性。类似 TS 如何使用`JSX.ElementAttributesProperty`去决定属性的名字，TS 使用`JSX.ElementChildrenAttribute`去决定 children 的属性的名字。`JSX.ElementChildrenAttribut`应该声明一个单独的属性。
```ts
declare namespace JSX {
  interface ElementChildrenAttribute {
    children: {}; // specify children name to use
  }
}
```
```ts
<div>
  <h1>Hello</h1>
</div>;

<div>
  <h1>Hello</h1>
  World
</div>;

const CustomComp = (props) => <div>{props.children}</div>
<CustomComp>
  <div>Hello World</div>
  {"This is just a JS expression..." + 1000}
</CustomComp>
```

你可以指定 children 的类型，就像其他属性。这将会覆盖默认乐行，莫如[React 类型]()。如果你使用他们。

```ts
interface PropsType {
  children: JSX.Element
  name: string
}

class Component extends React.Component<PropsType, {}> {
  render() {
    return (
      <h2>
        {this.props.children}
      </h2>
    )
  }
}

// OK
<Component name="foo">
  <h1>Hello World</h1>
</Component>

// Error: children is of type JSX.Element not array of JSX.Element
<Component name="bar">
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// Error: children is of type JSX.Element not array of JSX.Element or string.
<Component name="baz">
  <h1>Hello</h1>
  World
</Component>
```

### JSX 结果类型

默认情况下 JSX 表达式结果的类型是`any`。你可以通过指定`JSX.Element`接口来指定类型。然而，从这个接口获取元素、属性或者 JSX 的 children 的类型是不可能的，他是一个黑盒。


### 嵌入的表达式

JSX 允许你在标签中去嵌入表达式，通过使用花括号（`{}`）去包裹表达式。
```
var a = <div>
  {["foo", "bar"].map(i => <span>{i / 2}</span>)}
</div>
```
前面的例子将会导致一个错误，因为你无法铜鼓数字除以一个字符。结果，当使用`preerve`选项，看起来：
```
var a = <div>
  {["foo", "bar"].map(function (i) { return <span>{i / 2}</span>; })}
</div>

```


### React 集成

为了在 React 中使用 JSX，你应该使用[React 类型]()。这些类型为使用 React 定义了适当的`JSX`命名空间。

```
/// <reference path="react.d.ts" />

interface Props {
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>;
  }
}

<MyComponent foo="bar" />; // ok
<MyComponent foo={0} />; // error
```


### 工厂函数

使用明确的工厂函数通过`jsx: react`编译器选项是可配置的。他可以使用`jsxFactory`命令行选项或者在每一个文件内联`@jsx`评论编译指令来设置。比如，如果你设置`jsxFactory`为`createElement`，`<div />`将会发射为`createElement("div")`，而不是`React.createElement("div")`。

评论编译指令版本可能被这么使用（在 TypeScript 2.8）:
```
import preact = require("preact");
/* @jsx preact.h */
const x = <div />;
```
发射为：
```
const preact = require("preact");
const x = preact.h("div", null);
```

工厂的选择将会影响`JSX`命名空间的寻址（为了类型检查信息），在落到全局的那个之前。如果工厂定义为`React.createElement`（默认），编译器将会检查`React.JSX`，在检查全局的`JSX`之前。如果工厂定义为`h`，它将会检查`h.JSX`，在检查全局`JSX`之前。