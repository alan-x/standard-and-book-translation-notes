[已校对]
# 避免默认导出

假设你有一个文件`foo.ts`，有下面的内容：
```ts
class Foo {
}
export default Foo;
```

你将会如下使用 ES6 语法导入它（在`bar.ts`）：
```ts
import Foo from "./foo";
```

这里有一些维护性问题：

- 如果你在`foo.ts`重构`Foo`，它不会在`bar.ts`重命名。
- 如果你最终需要从`foo.ts`导出更多东西（这也是你的很多文件会有的），则你需要去改变导入语句。

因为这个原因，我推荐简单导出 + 解构导入。比如，`foo.ts`：
```ts
export class Foo {
}
```
然后：
```ts
import { Foo } from "./foo";
```

下面我也会展示一些更多的原因。

### 可怜的可发现性

摸到导出的可发现型非常差。你不能通过智能提示探索一个模块去查阅他是否有一个默认导出。

在默认导出的情况下，你啥也得不到（可能它导出默认/可能他不导出`¯\_(ツ)_/¯`）。

```ts
import /* here */ from 'something';
```

没有默认导出，你将会得到一个很好的智能提示：
```ts
import { /* here */ } from 'something';
```

### 自动完成

不管你是否了解导出，你都可以在这里`import {/*here*/} from "./foo";`的鼠标为止自动完成。给你的开发者一点手腕解脱。

### CommonJS 互操作

使用`default`，对于 commonJS 用户的体验非常糟糕，需要编写`const {default} = require('module/foo');`而不是`const {Foo} = require('module/foo')`。你基本上要想重命名`default`导出到你想要导入的。

### 错误输入保护

你不会获得想一个开发者执行`import Foo from "./foo";`，而另一个`import foo from "./foo";`这么做的错误输入。

### TypeScript 自动导入

自动导入工作的很好。你使用`Foo`和自动导入将会写下`import { Foo } from "./foo";`的代码，因为他有一个很好的来自模块的具名导出。一些工具将会尝试魔法阅读和推断一个默认导出，但是魔法是脆弱的。

### 重新导出

重新导出是在 npm 包的`index`根文件非常常见，并强制你去手动命名默认导出，比如`export { default as Foo } from "./foo";`（使用默认）vs `export * from "./foo"`(使用具名导出)

### 动态导入

默认导出在动态`导入`很差的暴露他们为`default`，比如：
```ts
const HighCharts = await import('https://code.highcharts.com/js/es-modules/masters/highcharts.src.js');
HighCharts.default.chart('container', { ... }); // Notice `.default`
```

具名导出更加优雅：
```ts
const {HighCharts} = await import('https://code.highcharts.com/js/es-modules/masters/highcharts.src.js');
HighCharts.chart('container', { ... }); // Notice `.default`
```

### 非类/非函数需要两行

对于函数/类可以一个语句，比如：
```ts
export default function foo() {
}
```
为非具名/类型声明对象可以一个语句，比如：
```ts
export default {
  notAFunction: 'Yeah, I am not a function or a class',
  soWhat: 'The export is now *removed* from the declaration'
};
```

但是其他需要两个语句：
```ts
// If you need to name it (here `foo`) for local use OR need to annotate type (here `Foo`)
const foo: Foo = {
  notAFunction: 'Yeah, I am not a function or a class',
  soWhat: 'The export is now *removed* from the declaration'
};
export default foo;
```
