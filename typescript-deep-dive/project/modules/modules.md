# 模块

### 全局模块

默认，当你在新的 TypeScript 文件开始编写代码的时候，你的代码在一个全局命名空间。作为一个例子，假设有文件`foo.ts`:
```ts
var foo = 123;
```

如果你现在创建了一个新的文件`bar.ts`在相同项目，你将会通过 TypeScript 类型系统允许去使用变量`foo`，就像它是全局可用的：
```ts
var bar = foo; // allowed
```

全局命名空间是危险的，因为它让你的代码面临命名冲突。我们推荐使用文件模块，它将会在下面展示。

### 文件模块



也叫做外部模块。如果你的 TypeScript 根级别有一个`import`或者`export`，则它会在文件内创建一个本地作用域。因此，如果我们改变前面的`foo.ts`到下面（注意`export`的使用）：
```ts
export var foo = 123;
```
在全局空间将不会有`foo`。这可以通过如下创建一个新的文件`bar.ts`展示：
```ts
var bar = foo; // ERROR: "cannot find name 'foo'"
```

如果你想要在`bar.ts`中使用`foo.ts`的东西，你需要你需要明确导入它。这显示在一个更新的`bar.ts`：
```ts
import { foo } from "./foo";
var bar = foo; // allowed
```

在`bar.ts`中使用`import`不仅仅允许你从其他文件引入东西，也标志`bar.ts`为一个模块，因此，`bar.ts`内的声明不污染全局命名空间。

从给定的使用外部模块的 TypeScript 文件中生成什么 JavaScript 是通过编译器标志`module`驱动的。