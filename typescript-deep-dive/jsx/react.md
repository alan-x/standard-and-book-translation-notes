[已校对]
# React

> [React/TypeScript 最佳实践的免费的 youtube 视频](https://www.youtube.com/watch?v=7EW67MqgJvs&list=PLYvdvJlnTOjHNayH7MukKbSJ6PueUNkkG)
> [PRO Egghead 的 TypeScript 和 React 课程](https://egghead.io/courses/use-typescript-to-develop-react-applications)

### 设置

我们的[浏览器快速入口已经为你设置好了 react 应用的开发](https://basarat.gitbook.io/typescript/browser)。这是关键点。

- 使用`.tsx`文件扩展（而不是`.ts`）
- 在你的`tsconfig.json`的`compilerOptions`使用`"jsx": "react"`。
- 安装 JSX 和 React 的定义到你的项目：（`npm i -D @types/react @types/react-dom`）。
- 引入 react 到你的`.tsc`文件（import * as React from "react"）。

### HTML 标签 vs 组件

React 可以渲染 HTML 标签（字符串）或者 React 组件。JavaScript 这些元素生成的 js 不一样（`React.createElement('div')` vs `React.createElement(MyComponent)`）。这取决于第一个字符的大小写。`foo`被认为是 HTML 标签，`Foo`是一个组件。

### 类型检测

#### HTML 标签

一个 HTML 标签`foo`的类型是`JSX.IntrinsicElements.foo`。这些类型已经为大部分标签定义在文件`react-jsx.d.ts`中，我们已经安装为设置的一部分。这是这个文件内容的一个例子：
```ts
declare module JSX {
    interface IntrinsicElements {
        a: React.HTMLAttributes;
        abbr: React.HTMLAttributes;
        div: React.HTMLAttributes;
        span: React.HTMLAttributes;

        /// so on ...
    }
}
```

#### 函数组件

你可以使用`React.FunctionComponent`接口定义函数组件：
```ts
type Props = {
  foo: string;
}
const MyComponent: React.FunctionComponent<Props> = (props) => {
    return <span>{props.foo}</span>
}

<MyComponent foo="bar" />
```

#### 空函数组件

随着[@types/react PR #46643](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/46643)，你可以使用一个新的`React.VoidFunctionComponent`或者`React.VFC`类型，如果你希望去声明一个不接受`children`得到组件。这是一个临时解决方案，知道下一个大版本的类型定义（VoidFunctionComponent 将会被废弃，FunctionComponent 将会默认接受无 children）。

```ts
type Props = { 
  foo: string 
}
// OK now, in future, error
const FunctionComponent: React.FunctionComponent<Props> = ({ foo, children }: Props) => {
    return <div>{foo} {children}</div>; // OK
};
// Error now (children not support), in future, deprecated
const VoidFunctionComponent: React.VoidFunctionComponent<Props> = ({ foo, children }) => {
    return <div>{foo}{children}</div>; 
};
```

#### 类组件

组件的类型检测基于组件的`props`属性。这是基于 JSX 转化的方式建模的，比如属性成为组件的`props`。

`react.d.ts`文件定义了`React.Component<Props,State>`类，你应该扩展你自己的类，提供你自己的`Props`和`State`接口。这显示在下面：
```ts
type Props = {
  foo: string;
}
class MyComponent extends React.Component<Props, {}> {
    render() {
        return <span>{this.props.foo}</span>
    }
}

<MyComponent foo="bar" />
```


#### React JSX 提示：可渲染的接口

React 可以渲染一些类似`JSX`或者`string`。这些都考虑在`React.ReactNode`的类型中，因此，在你想要接受可渲染的时候使用它：
```ts
type Props = {
  header: React.ReactNode;
  body: React.ReactNode;
}
class MyComponent extends React.Component<Props, {}> {
    render() {
        return <div>
            {this.props.header}
            {this.props.body}
        </div>;
    }
}

<MyComponent header={<h1>Header</h1>} body={<i>body</i>} />
```

#### React JSX 提示：接受一个组件的实例

React 定义提供`React.ReactElement<T>`去允许你去声明`<T/>`类组件实例的结果。比如：
```ts
class MyAwesomeComponent extends React.Component {
  render() {
    return <div>Hello</div>;
  }
}

const foo: React.ReactElement<MyAwesomeComponent> = <MyAwesomeComponent />; // Okay
const bar: React.ReactElement<MyAwesomeComponent> = <NotMyAwesomeComponent />; // Error!
```

> 当然你可以使用这个作为函数参数声明，甚至 React 组件属性成员。

#### React JSX 提示：接受一个组件，可以在属性上使用，并使用 JSX 渲染

`React.Component<Props>`类型整合了`React.ComponentClass<P> | React.StatelessComponent<P>`，因此你可以接受使用类型`Props`和使用 JSX 渲染的东西：
```ts
const X: React.Component<Props> = foo; // from somewhere

// Render X with some props:
<X {...props}/>;
```

#### React JSX 提示：泛型组件

它按预期工作。这是一个例子：
```ts
/** A generic component */
type SelectProps<T> = { items: T[] }
class Select<T> extends React.Component<SelectProps<T>, any> { }

/** Usage */
const Form = () => <Select<string> items={['a','b']} />;
```

#### 泛型函数

下面也工作的很好：
```ts
function foo<T>(x: T): T { return x; }
```

然而，使用一个箭头泛型函数不太行：
```ts
const foo = <T>(x: T) => x; // ERROR : unclosed `T` tag
```

变通方案：在泛型参数使用`extends`去提示编译器这不是一个泛型，比如：
```ts
const foo = <T extends unknown>(x: T) => x;
```

#### React 提示：强化类型引用

你基本上初始化一个变量作为 ref 和`null`的联合，然后初始化它作为一个回调，比如：
```ts
class Example extends React.Component {
  example() {
    // ... something
  }

  render() { return <div>Foo</div> }
}


class Use {
  exampleRef: Example | null = null; 

  render() {
    return <Example ref={exampleRef => this.exampleRef = exampleRef } />
  }
}
```

对于原生元素的 ref 也是一样的：
```ts
class FocusingInput extends React.Component<{ value: string, onChange: (value: string) => any }, {}>{
  input: HTMLInputElement | null = null;

  render() {
    return (
      <input
        ref={(input) => this.input = input}
        value={this.props.value}
        onChange={(e) => { this.props.onChange(e.target.value) } }
        />
      );
    }
    focus() {
      if (this.input != null) { this.input.focus() }
    }
}
```

#### 类型断言

就像[前面提到的](https://basarat.gitbook.io/typescript/type-system/type-assertion#as-foo-vs-foo)，使用`as Foo`语法作为类型断言。

### 默认属性

- 使用默认属性的状态组件：你可以告诉 TypeScript 一个属性将会使用一个 null 检测操作符去额外（通过 React）提供（这不是一个好主意，但是这是我能想到的最简单最大化的额外代码解决方案）。
```ts
class Hello extends React.Component<{
  /**
   * @default 'TypeScript'
   */
  compiler?: string,
  framework: string
}> {
  static defaultProps = {
    compiler: 'TypeScript'
  }
  render() {
    const compiler = this.props.compiler!;
    return (
      <div>
        <div>{compiler}</div>
        <div>{this.props.framework}</div>
      </div>
    );
  }
}

ReactDOM.render(
  <Hello framework="React" />, // TypeScript React
  document.getElementById("root")
);
```

- 使用默认属性的 SFC：推荐利用简单 JavaScript 模式，因为他们和 TypeScript 的类型系统结合的很好：
```ts
const Hello: React.SFC<{
  /**
   * @default 'TypeScript'
   */
  compiler?: string,
  framework: string
}> = ({
  compiler = 'TypeScript', // Default prop
  framework
}) => {
    return (
      <div>
        <div>{compiler}</div>
        <div>{framework}</div>
      </div>
    );
  };


ReactDOM.render(
  <Hello framework="React" />, // TypeScript React
  document.getElementById("root")
);
```

### 声明一个 webcomponent

如果你使用一个 web 组件，默认的 React 类型定义（`@types/react`）将不知道这个。但是你可以简单声明它，比如去声明一个 webcomponent 叫做`my-awesome-slider`，接受属性`MyAwesomeSliderProps`：
```ts
declare global {
  namespace JSX {
    interface IntrinsicElements {
      'my-awesome-slider': MyAwesomeSliderProps;
    }

    interface MyAwesomeSliderProps extends React.Attributes {
      name: string;
    }
  }
}
```

现在你可以在 TSX 中使用：
```tsx
<my-awesome-slider name='amazing'/>
```