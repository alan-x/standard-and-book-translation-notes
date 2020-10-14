这个指南的目标是去指导你怎样编写高质量的声明文件。这个指南通过显示一些 APPI 的文档构成，伴随着这些 API 的简单使用，并解释怎样编写对应的声明。

这些例子是按照复杂性的递增排列的。


### 具有属性的对象

*文档*

全局对象`myLib`有一个函数`makeGreting`去打招呼，和一个属性`numberOfGretings`指示目前打了多少个招呼。


*代码*
```ts
let result = myLib.makeGreeting("hello, world");
console.log("The computed greeting is:" + result);

let count = myLib.numberOfGreetings;
```


*声明*

使用`declare namespace`去描述通过点符号访问类型或者值。
```ts
declare namespace myLib {
  function makeGreeting(s: string): string;
  let numberOfGreetings: number;
}
```
### 重载函数

*文档*
`getWidget`函数接受一个数字并返回一个 Widget，或者接受一个字符串并返回一个 Widget 数组。

*代码*
```ts
let x: Widget = getWidget(43);

let arr: Widget[] = getWidget("all of them");
```

*声明*
```ts
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];
```

### 可重用的类型（接口）

*文档*

当指定一个招呼的时候，你必须传递一个`GreetingSettings`对象。这个对象有下面的属性：

1 - greting：强制字符串
2 - duration：可选的时间长度（毫秒）
3 - color：可选的字符串，比如'#ff00ff'

*代码*

```ts
greet({
  greeting: "hello world",
  duration: 4000
});
```

*声明*

使用`interface`去定义拥有属性的类型：
```
interface GreetingSettings {
  greeting: string;
  duration?: number;
  color?: string;
}

declare function greet(setting: GreetingSettings): void;
```

### 可重用类型（类型别名）

*文档*

无论在什么地方期待问候，你可以提供一个`syting`，一个函数发挥一个`string`，或者一个`Greter`实例。

*代码*
```ts
function getGreeting() {
  return "howdy";
}
class MyGreeter extends Greeter {}

greet("hello");
greet(getGreeting);
greet(new MyGreeter());
```

*声明*

你可以使用类型别名去为一个类型创建一个缩写：
```ts
type GreetingLike = string | (() => string) | MyGreeter;

declare function greet(g: GreetingLike): void;
```

### 组织类型

*文档*

`greeter`对象可以记录一个文件或者显示一个弹窗。你可以提供 Logoption 给`.log(...)`和弹窗选项给`.alert(>>>)`。

*代码*
```ts
const g = new Greeter("Hello");
g.log({ verbose: true });
g.alert({ modal: false, title: "Current Greeting" });

```

*声明*

使用命名空间去组织类型。
```ts
declare namespace GreetingLib {
  interface LogOptions {
    verbose?: boolean;
  }
  interface AlertOptions {
    modal: boolean;
    title?: string;
    color?: string;
  }
}
```

你也可以在一个声明内部创建嵌套的命名空间：
```ts
declare namespace GreetingLib.Options {
  // Refer to via GreetingLib.Options.Log
  interface Log {
    verbose?: boolean;
  }
  interface Alert {
    modal: boolean;
    title?: string;
    color?: string;
  }
}
```

### 类

*文档*

你可以创建一个 greeter，通过实例化`Greeter`对象，或者创建一个自定义的 greeter，通过扩展它。

*代码*
```ts
const myGreeter = new Greeter("hello, world");
myGreeter.greeting = "howdy";
myGreeter.showGreeting();

class SpecialGreeter extends Greeter {
  constructor() {
    super("Very special greetings");
  }
}

```

*声明*

使用`declare class`去描述一个类或者类似类的对象。类可以和构造器一样使用属性和方法。
```ts
declare class Greeter {
  constructor(greeting: string);

  greeting: string;
  showGreeting(): void;
}
```

### 全局变量

*文档*

全局变量`foo`包含存在的组件的数量。

*代码*
```ts
console.log("Half the number of widgets is " + foo / 2);
```

*声明*

使用`declare var`去声明变量。如果变量是只读的，你可以使用`declare const`。你也可以使用`declare let`，如果变量是块级作用域。
```ts
/** The number of widgets present */
declare var foo: number;
```

### 全局函数。

*文档*

你可以使用字符串调用函数`greet`去显示一个招呼给用户

*代码*
```ts
greet("hello, world");
```

*声明*

使用`declare function`去声明函数。
```ts
declare function greet(greeting: string): void;
```