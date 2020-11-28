[已校对]
# 风格指南

> 一个非官方 TypeScript 风格指南

人们问我关于这个的观点。个人而言，我不强制这些在我的团队和项目，但是它在一些人觉得需要有一个强力一致性的时候被提及的确有帮助。还有一些其他东西我觉得更强，覆盖在 [提示章节](https://basarat.gitbook.io/typescript/main-1)(比如，类型断言是坏的，属性设置起是坏的)。

关键章节：
- [变量](https://basarat.gitbook.io/typescript/styleguide#variable-and-function)
- [类](https://basarat.gitbook.io/typescript/styleguide#class)
- [接口](https://basarat.gitbook.io/typescript/styleguide#interface)
- [类型](https://basarat.gitbook.io/typescript/styleguide#type)
- [命名空间](https://basarat.gitbook.io/typescript/styleguide#namespace)
- [枚举](https://basarat.gitbook.io/typescript/styleguide#enum)
- [`null`vs`undefined`](https://basarat.gitbook.io/typescript/styleguide#null-vs-undefined)
- [格式化](https://basarat.gitbook.io/typescript/styleguide#formatting)
- [单引号和双引号](https://basarat.gitbook.io/typescript/styleguide#quotes)
- [Tab 和空格](https://basarat.gitbook.io/typescript/styleguide#spaces)
- [使用逗号](https://basarat.gitbook.io/typescript/styleguide#semicolons)
- [声明数组为`Type[]`](https://basarat.gitbook.io/typescript/styleguide#array)
- [文件名](https://basarat.gitbook.io/typescript/styleguide#filename)
- [`type`和`interface`](https://basarat.gitbook.io/typescript/styleguide#type-vs-interface)

### 变量和函数

- 为变量和函数名使用`camelCase`

> 原因：JavaScript 约定

**不好的：**
```ts
var FooVar;
function BarFunc() { }
```
**好的：**
```
var fooVar;
function barFunc() { }
```

### 类
- 为类名使用`pascalCase`：

> 原因：这个在标准 javaScript 中非常常见

**坏的**
```ts
class foo { }
```
**好的**
```ts
class Foo { }
```
- 为类的成员和方法使用`camelCase`

> 原因：很自然的遵循变量和函数约定

**不好的**
```ts
class Foo {
    Bar: number;
    Baz() { }
}
```

**好的**
```ts
class Foo {
    bar: number;
    baz() { }
}
```

### 接口
- 为名字使用`PascalCase`。
> 原因：和类类似

- 为成员使用`camelCase`
> 原因：和类类似

- 不要使用前缀`I`
> 原因：不是约定。`lib.d.ts`没有使用一个`I`义重要的接口（比如 Window，Document 等）。

**不好的**
```ts
interface IFoo {
}
```
**好的**
```ts
interface Foo {
}
```

### 类型
- 为名字使用`pascalCase`。
> 原因：和类类似

- 为成员使用`cameCase`
> 原因：和类类似

### 命名空间

- 为名字使用`PascalCase`
> 原因：TypeScript 团队的约定。命名空间只对使用静态成员的类有效。类名字是`PascalCase`=>命名空间名字是`PascakCase`

**Bad*
```ts
namespace foo {
}
```

**Good**
```ts
namespace Foo {
}
```

### 枚举

- 为枚举名字使用`PascalCase`

> 原因：和类类似，是一个类型

**不好的**
```ts
enum color {
}
```

**好的**
```ts
enum Color {
}
```

> 原因：TypeScript 团队的原定，比如，语言创建者，比如`SyntaxKind.StringLiteral`：当然，也帮助其他语言转化（代码生成）为 TypeScript。

**坏的**
```ts
enum Color {
    red
}
```
**好的**
```ts
enum Color {
    Red
}
```

### Null vs Undefined

- 对于显示不可用，应该不要用

> 原因：这些值通常用于在值之间保持一致。在 TypeScript 中，你使用 type 去表示这个解构

**不好的**
```ts
let foo = { x: 123, y: undefined };
```
**好的**
```ts
let foo: { x: number, y?: number } = { x:123 };
```
- 通常使用`undefined`（考虑返回一个类似`{valid:boolean, value?:Foo}`的对象）

**坏的**
```ts
return null;
```

**好的**
```ts
return undefined;
```
- 在 API 的一部分使用`null`或者约定

> 原因：在 Node.js 中很方便，比如，在 NodeBack 风格回调中，`error`是`null`

**不好的**
```ts
cb(undefined)
```
**好的**
```ts
cb(null)
```
- 为对象使用真值检测是`null`或者`undefined`

**不好的**
```ts
cb(null)
```

**好的**
```ts
if (error)
```
- 使用`== null`/`!= null`（不是`===`/`!==`）去检测原生`null`/`undefined`，因为他对`nul`/`undefined`都有用，但是对其他假值（比如`''`，`0`，`false`）没用，比如.

**不好的**
```ts
if (error !== null) // does not rule out undefined
```

**好的**
```ts
if (error != null) // rules out both null and undefined
```

### 格式化

TypeScript 编译器带来了一个很好的语言格式化服务。无论他默认提供什么输出，对于缓解团队负载都很有帮助。

在命令行使用`tsfmt`去自动格式化你的代码。当然，你的 IDE（atom/vscode/vs。submit）已经内置支持格式化。

例子：
```ts
// Space before type i.e. foo:<space>string
const foo: string = "hello";
```

### 引号

- 推荐单引号`'`，除非必要

> 原因：很多团队这么做（比如，[airbnb](https://github.com/airbnb/javascript)，[standard](https://github.com/feross/standard)，[npm](https://github.com/npm/npm)，[node](https://github.com/nodejs/node)，[google/angular](https://github.com/angular/angular/)，[facebook/react](https://github.com/facebook/react)）。输入很简单（在大部分键盘不需要 shift）。[Prettier 团队也推荐单引号](https://github.com/prettier/prettier/issues/1105)

> 双引号并非没有急啊值：允许简单复制粘贴对象到 JSON。允许人们使用其他语言去使用而不改变引号。但我决定不偏离 JS 社区的决定。

- 当你不实用双引号，尝试使用反引号（`）。

> 原因：这通常表示足够复杂的字符串。

### 空格

- 使用`2`空格，不是 tab

> 原因：很多团队这么做（比如，[airbnb](https://github.com/airbnb/javascript)，[idiomatic](https://github.com/rwaldron/idiomatic.js)，[standard](https://github.com/feross/standard)，[npm](https://github.com/npm/npm)，[node](https://github.com/nodejs/node)，[google/angular](https://github.com/angular/angular/)，[facebook/react](https://github.com/facebook/react)）。TypeScript/VSCode 团队使用 4 空格，但是对生态系统中例外。

### 封号

- 使用封号

> 原因：明确的封号帮助语言格式化工具提供一致的结果。缺少 ASI（自动封号插入）可以欺骗新的开发者，比如`foo() \n (function(){})`，将会成为一个单独的语句（不是两个）。TC39[也警告这个](https://github.com/tc39/ecma262/pull/1062)。团队例子：[airbnb](https://github.com/airbnb/javascript)，[idiomatic](https://github.com/rwaldron/idiomatic.js)，[google/angulr](https://github.com/angular/angular/)，[facebook/react](https://github.com/facebook/react)，[Microsoft/TypeScript](https://github.com/Microsoft/TypeScript/)。

### 数组

- 声明数组为`foos: Foo[]`，而不是`foos: Array<Foo>`。

> 原因：阅读比较简单。TypeScript 团队这么用。知道某些东西是数组比较简单，因为被后缀的`[]`提示。

### 文件名

使用`cameCase`命名文件。比如`accordion.tsx`，`myControl.tsx`，`utils.ts`，`map.ts etc`。

> 原因：很多 JS 团队间的约定。

### type vs interface

- 使用`type`，当你可能需要一个联合或者交叉的时候：
```ts
type Foo = number | { someProperty: number }
```
- 使用`interface`，当你想要`extends`或者`implement`，比如：
```ts
interface Foo {
  foo: string;
}
interface FooBar extends Foo {
  bar: string;
}
class X implements FooBar {
  foo: string;
  bar: string;
}
```

- 否则，使用任何能让你开心的。