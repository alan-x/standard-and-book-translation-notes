三斜线指令是单行评论包含一个单独的 XML 标签。评论的内容用做编译器指令。

三斜线指令只在包含他们的文件的顶部才有效。一个三斜线指前面只能有单行或者多行评论，包含其他三斜线指令。如果在语句或者一个声明之后遇到，他们被认为是一个常规单行注释，而没有特殊意义。


### /// <reference path="..." />

`/// <reference path="..." />`指令是这个分组最常见的。它在文件间的作为依赖声明存在。

三斜线索引至死后编译器在编译进程包含额外的文件。

当使用`--out`或者`--outFile`的时候，他们也作为排序输出的方法。文件以相同的顺序被发送到输出文件定位，就像预处理之后的输入。

#### 预处理输入文件

编译器在输入文件上执行一个预处理过程，解析所有三斜线索引指令。在这个过程，额外的文件被添加到编译器。

这个过程以一个集合的 root 文件开始，是指定在命令行或者在`tsconfig.json`文件的`"files"`列表。这个 root 文件以他们被指定的顺序被预处理。在一个文件被添加到列表之前，它内部的所有的三斜线索引都被处理，他们的目标也被包含。三斜线索引以深度有限的方式被解析。以这种方式，，他们可以在文件内可见。

一个三斜线索引路径被解析为根据包含文件相对，如果不是根。

#### 错误

引用一个不存在的文件是一个错误。一个文件引用自己也是一个错误。

#### 使用 --noResolve

如果编译器标识`--noResolve`被指定，三斜线引用被忽略；他们要么添加新的文件，要么改变提供的文件的顺序。

### /// <reference types="..." />

和作为依赖声明的`/// <reference path="..." />`指令类似，`/// <reference types="..." />`指令在一个包中声明一个依赖。

解析这些包名和`import`语句中的包哦名解析过程类似。理解三斜线索引类型指令的方式是作为`import`声明包。

比如，在一个声明文件包含`/// <reference types="node" />`声明这个文件使用声明在`@types/node/index.d.ts`中的in 工资；因此，这个包需要和声明文件包含在编译器。

只有当你是`d.ts`文件的作者的时候，才使用这些指令，

因为声明文件在编译器见生成，编译器将自动为你添加`/// <reference types="..." />`；一个`/// <reference types="..." />`在一个生成的声明文件被添加，如果并且只有如果结果文件的任何从索引的包中被使用。

为了在一个`.ts`文件声明一个在`@types`包的依赖，在命令航使用`--types`，或者在你的`tsconfig.json`。查阅[使用 @types，typeconfig.json 文件中的 typrRoots 和 types]()了解更多。

### /// <reference lib="..." />

这个指令允许一个文件去明确的包含一个存在的内建的库文件。

内建的 lib 文件以 tsconfig.json 内`"lib"`编译选项相同的风格引用（比如，使用`lib="es2015"`和非`lib="lib.es2015.d.ts"`等）。

对于依赖内建类型的声明文件作者，比如，类似`Symbol`或者`Iterable`类似的内建 JS 运行时构造器或者 DOM API，三斜线引用库指令是推荐的。前面的这些 .d.ts 文件文件必须添加这些类型的前向/重复声明。

比如，添加`/// <reference lib="es2017.string" />`到编译的一个文件和`--lib es2017.string`编译选项相同。
```
/// <reference lib="es2017.string" />

"foo".padStart(4);
```

### /// <reference no-default-lib="true"/>

这个指令标志一个文件为默认库。你将在`lib.d.ts`文件的头部看到这个评论和它的变体。

这个指令指示编译器在编译的时候不要包含默认库（比如，`lib.d.ts`）。和在命令行传递`--noLib`了类似。

也要注意，当传递`--skipDefaultLibCheck`，编译器将只跳过使用`/// <reference no-default-lib="true"/>`的检查。


### /// <amd-module />

默认，AMD 模块生成是匿名的。这会导致问题，当其他工具用来处理生的模块的时候，比如打包器（比如，`r.js`）。

`amd-module`指令允许传递一个可选的模块名字到编译器：

amdModule.ts
```
///<amd-module name="NamedModule"/>
export class C {}
```

将会导致赋值名字`NamedModule`给模块，作为调用 AMD`define`的一部分：

amdModule.js
```
define("NamedModule", ["require", "exports"], function (require, exports) {
  var C = (function () {
    function C() {}
    return C;
  })();
  exports.C = C;
});

```

### /// <amd-dependency />

注意：这个指令已经被废弃，使用`import "moduleName";`语句替代。

`/// <amd-dependency path="x" />`通知编译器关于一个非 TS 模块依赖需要注入到生成的模块的 require 调用。

`amd-dependency`指令可以有一个可选的`name`属性；这允许为 amd 依赖传递一个可选的名字。

```
/// <amd-dependency path="legacy/moduleA" name="moduleA"/>
declare var moduleA: MyType;
moduleA.callStuff();

```
生成的 JS 代码：
```
define(["require", "exports", "legacy/moduleA"], function (
  require,
  exports,
  moduleA
) {
  moduleA.callStuff();
});

```

