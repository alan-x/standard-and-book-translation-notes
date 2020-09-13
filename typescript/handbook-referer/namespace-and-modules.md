这个文章描述了在 TypeScript 中使用模块和命名空间组织你的代码的多种方式。我们当然也会设计一些高级主题，比如怎样使用命名空间和模块，并提出一些在 TypeScript 中使用他们的一些陷阱。

查阅[模块]()文档了解更多关于 ES 模块的信息。查阅[命名空间]()文档了解更多关于 TypeScript 命名空间的信息。

注意：在很旧的版本的 TypeScript 中，命名空间被叫做‘内部的模块’，这些早期的模块系统。

### 使用模块

模块可以包含代码和声明。

模块也依赖于模块加载器（比如 CommonJs/Require.js）或者支持 ES 模块的运行时。模块提供了更好的代码复用，更强的隔离和更好的构建工具支持。



### 使用命名空间

命名空间是 TypeScript 特定的组织代码的方式。命名空间只是全局命名空间内具名 JavaScript 对象。这让命名空间非常简单的构造去使用。不想模块，他们可以分离多个文件，并且可以使用`--outFile`结合。命名空间在 Web 应用中也能结构化你的代码的好方法，所有的依赖都包含在你的 HTML 页面的`<script>`。

就像素有全局空间污染，很难去定义组件依赖，特别是大型应用。

值得注意的是，对于 Node.js 应用，模块是默认的，并且我们推荐在现代代码中，使用模块而不是命名空间。

从 ESMAScript 2015 开始，模块是语言的原生部分，并且应该被所有的编译引擎实现支持。因此，对于新的项目，模块是更推荐的代码组织机制。

### 命名空间和模块的陷阱

在这个章节我们将描述一些使用命名空间和模块的常见陷阱和如何避免他们。


#### /// <reference>-ing a module

一个常见错误是去长思使用`/// <reference ... />`语法去引用一个模块文件，而不是使用`import`语法。为了理解区别，我们首先需要去理解编译器如何定义模型的类型信息，基于一个`import`路径（比如，`import x from "...";`中的`...`，`import x = require("...");`等）。

编译器将尝试私用合适的路径去寻找一个`.ts`，`.tsx`，然后是`.d.ts`。如果指定的文件找不到，则编译器将会去选好一个环境模块声明。需要在`.d.ts`文件中调用。

- myModules.d.ts

```
// In a .d.ts file or .ts file that is not a module:
declare module "SomeModule" {
export function fn(): string;
}

```

myOtherModule.ts
```
/// <reference path="myModules.d.ts" />
import * as m from "SomeModule";
```

这里的索引标签允许我们定位包含环境模块的声明文件。这是`node.d.ts`文件被多个 TypeScript 例子消费的方式。

### 不必要的命名空间

如果你正在转化命名空间到模块，可以很简单的以这种形式结束：

- shapes.ts

```
export namespace Shapes {
export class Triangle {
  /* ... */
}
export class Square {
  /* ... */
}
}
```

顶级的模块`Shapes`包裹`Triangle`和`Square`，没有任何原因。这会让你的模块的消费者迷惑和烦恼。


- shapeConsumer.ts
```
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

TypeScript 中，模块的一个关键特性是两个不同的模块不会贡献到相同的范围。因为模块的消费这决定赋值它什么名字，因此不需要去主动包裹导出的标识到命名空间。

为了声明为什么那你不应该尝试使用命名空间包裹你的模块内容，通常，命名空间是为了提供构造的逻辑分组并放置命名冲突。因为模块文件本身已经是一个逻辑分组，并且他的顶级名字是通过引入他的代码定义的，不需要去为导出的对象使用一个额外的模块层。

这是一个订正后的例子

- shapes.ts

```
export class Triangle {
/* ... */
}
export class Square {
/* ... */
}
```

- shapeConsumer.ts

```
import * as shapes from "./shapes";
let t = new shapes.Triangle();
```

### 模块的权衡取舍

就像 JS 文件和模块是一对一的，TypeScript 的模块源文件和他们生成的 JS 文件也是一对一的。这种方式的一种效果是无法链接多个模块源文件，因为于你的目标的模块系统。比如，你不能使用`outFile`选项，当目标是`commonjs`或者`umd`，但是使用 TypeScript 1.8 或者之后，[可能]()去使用`outFile`，当你的目标是`amd`或者`system`。