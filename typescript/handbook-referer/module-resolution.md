模块解析

这个章节假设你有一些关于模块的基本只是。请查阅[模块]()章节文档了解更多信息。

模块解析是编译器用来指出一个导入引用的流程。考虑一个导入语句，类似`import { a } from "moduleA"`;为了检查任何`a`的使用，编译器需要明确知道它的存在，需要检查`moduleA`的定义。

这时候，编译器将会问“`moduleA 的外型是什么？`”尽管这听起来很直接，`moduleA`必须定义在你的其中一个`.ts`/`.tsx`文件中，或者在一个你的代码依赖的`d.ts`。

### 相对和非相对模块导入

模块导入解析根据模块引用是相对的还是非相对的。

相对导入是以`/`，`./`或者`../`开始的，比如：

- `import Entry from "./components/Entry";`

- `import { DefaultHeaders } from "../constants/http";`

- `import "/mod";`

其他导入被认为是非相对的。比如：

- `import * as $ from "jquery";`

- `import { Component } from "@angular/core";`

一个相对导入相对导入的文件解析，不能解析为环境模块声明。你应该为你自己的模块使用相对导入，确保在运行时维护他们的相对定位。

一个非相对导入可以相对`baseUrl`解析，或者通过路径映射，我们将会在稍后提及。他们也可以被解析为[环境模块声明]()。当你导入任何你的外部依赖的时候，使用非相对路径。

### 模块解析策略

有两种可能的模块解析策略：[Node]()和[Classic]()。你可以使用`--moduleResolution`标志去指定模块解析策略。如果没有指定，默认是[Node]()，`--module commonjs`，否则就是[Classic]()（包含当`--module`被设置为`amd`，`system`，`umd`，`es2015`，`esnext`，等）。

注意：`node`模块解析是 TypeScript 社区中最常见的，并且推荐大部分的项目使用。如果你在 TypeScript 中使用`import`和`export`有方案问题，尝试设置`moduleResolution:"node"`去试试能不能修复这个问题。

#### 经典

这在过去是默认的解析策略，现在，这个策略存在主要为了向后兼容。

一个相对导入将会相对于导入文件解析。因此在`/root/src/folder/A.ts`中的` import { b } from "./moduleB"`将会导致下面的寻址：

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`

对于非相对模块导入，然而，编译器遍历包含导入文件的文件夹树，尝试去定位命中的定义文件。

比如：

像`import { b } from "moduleB"`之类的`moduleB`的一个非相对导入，在一个源文件`/root/src/folder/A.ts`，将会导致尝试在下面的未知定位`"moduleB"`:

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`
3. `/root/src/moduleB.ts`
4. `/root/src/moduleB.d.ts`
5. `/root/moduleB.ts`
6. `/root/moduleB.d.ts`
7. `/moduleB.ts`
8. `/moduleB.d.ts`

#### Node

这个解析策略尝试在运行时去模拟[Node]()模块解析机制。完整的 Node.jd 解析算法在[Node.js 模块文档]()标记。

#### Node.js 如何解析模块

为了理解 TS 编译器将会遵循的步骤，阐明以下 Node.js 模块很重要。传统上，Node.js 的导入通过调用名为`require`的调用。这个行为，Node.js 根据给`require`的是一个相对路径还是非相对路径有不同的行为。

相对路径非常公平直接。比如，假设一个文件位于`/root/src/moduleA.js`，包含导入`var x = require("./moduleB");`，Node.js 按下列顺序解析导入：

1. 询问文件名`/root/src/moduleB.js`，如果它存在。

2. 询问文件夹`/root/src/moduleB`，如果它包含名为`package.json`的文件，指定了一个`"main"`模块。在我们的例子，如果 Node.js 找到文件`/root/src/moduleB/package.json`包含`{ "main": "lib/mainModule.js" }`，Node.js 将解析为`/root/src/moduleB/lib/mainModule.js`。

3. 询问文件夹`/root/src/moduleB`，如果它包含一个名为`index.js`的文件，这个文件被认为是文件夹的主模块。

你可以在 Node.js 的[文件模块]()和[文件夹模块]()了解更多。

然而，[非相对模块名]()的解析完全不同。Node 将会在名为`node_modules`的指定文件夹下寻找你的模块。一个`node_modules`文件夹将会和当前文件相同级别，或者比文件夹链更高的级别，Node 将会遍历文件夹链，寻找`node_module`，直到找到你尝试加载的模块。

跟随我们前面的例子，假设`/root/src/moduleA.js`替换使用非相对路径，并且有`var x = require("moduleB");`导入。Node 将会尝试去解析`moduleB`为以下每一额定位，直到其中一个有效。

1. `/root/src/node_modules/moduleB.js`
2. `/root/src/node_modules/moduleB/package.json`(如果它指定了一个`"main"`属性)
3. `/root/src/node_modules/moduleB/index.js`
4. `/root/node_modules/moduleB.js`
5. `/root/node_modules/moduleB/package.json`(如果它指定了一个`"main"`属性)
6. `/root/node_modules/moduleB/index.js`
7. `/node_modules/moduleB.js`
8. `/node_modules/moduleB/package.json`(如果它指定了一个`"main"`属性)
9. `/node_modules/moduleB/index.js`

#### TypeScript 是如何解析模块的

TypeScript 将会模拟 Node.js 运行时解析策略，为了在编译时去定位模块的定义文件。为了完成这个，TypeScript 添加 TypeScript 文件扩展（`.ts`，`.tsx`，`.d.ts`）到 Node 的解析逻辑之上。TypeScript 将会在`package.json`中使用名为`"typed"`的域去模拟`"main"`的目的-编译器将会使用它去找到主定义文件去咨询。

比如，一个类似`import { b } from "./moduleB"`导入语句在`/root/src/moduleA.ts`中将会导致如下`"./moduleB"`定位：

1. `/root/src/moduleB.ts`
2. `/root/src/moduleB.tsx`
3. `/root/src/moduleB.d.ts`
4. `/root/src/moduleB/package.json`(如果它指定了一个`"types"`属性)
5. `/root/src/moduleB/index.ts`
6. `/root/src/moduleB/index.tsx`
7. `/root/src/moduleB/index.d.ts`

回想一下 Node.js 查找名为`moduleB.js`的文件，然后是可用的`package.json`，然后是`index.js`文件。

同样的一个非相对导入将会遵循 Node.js 解析逻辑，首先寻找一个文件，然后寻找一个可用的文件夹。因此源文件中的`/root/src/moduleA.ts`的`import { b } from "moduleB"`将会导致如下寻找：
1. `/root/src/node_modules/moduleB.ts`
2. `/root/src/node_modules/moduleB.tsx`
3. `/root/src/node_modules/moduleB.d.ts`
4. `/root/src/node_modules/moduleB/package.json`(如果它指定了一个`"types"`属性)
5. `/root/src/node_modules/@types/moduleB.d.ts`
6. `/root/src/node_modules/moduleB/index.ts`
7. `/root/src/node_modules/moduleB/index.tsx`
8. `/root/src/node_modules/moduleB/index.d.ts`

9. `/root/node_modules/moduleB.ts`
10. `/root/node_modules/moduleB.tsx`
11. `/root/node_modules/moduleB.d.ts`
12. `/root/node_modules/moduleB/package.json`(如果它指定了一个`"types"`属性)
13. `/root/node_modules/@types/moduleB.d.ts`
14. `/root/node_modules/moduleB/index.ts`
15. `/root/node_modules/moduleB/index.tsx`
16. `/root/node_modules/moduleB/index.d.ts`(如果它指定了一个`"types"`属性)

17. `/node_modules/moduleB.ts`
18. `/node_modules/moduleB.tsx`
19. `/node_modules/moduleB.d.ts`
20. `/node_modules/moduleB/package.json`(如果它指定了一个`"types"`属性)
21. `/node_modules/@types/moduleB.d.ts`
22. `/node_modules/@types/moduleB.d.ts`
22. `/node_modules/moduleB/index.ts`
23. `/node_modules/moduleB/index.tsx`
24. `/node_modules/moduleB/index.d.ts`


不要害怕这里步骤的数量 - TypeScript 只在步骤（9）和（17）跳跃文件夹。这真的没有比 Node.js 自身做的更复杂。

### 附加模块解析标志

一个项目源码布局有时候和输出不一致。使用一个集合的构建步骤导致生成最后的输出。这些包含编译`.ts`到`.js`，从不同的源码定位复制依赖到单一的输出定位。最终结果是运行时模块和包含他们定义的源文件相比可能有不同的名字。或者最终输出中的模块路径可能和编译时对应的源文件路径不一致。

TypeScript 编译器有一个集合的附加标志去告诉转译编译器在源码生成最终输出时期望发生的。

编译器不会执行任何转换，这很重要；它只是使用这一块的信息去直到模块解析到他的定义文件。

#### 基准 URL

在使用 AMD 模块加载器的应用中使用`baseUrl`是一个常见的实践，模块在运行时被“部署到一个单独的文件。这些源模块的源文件可以存在在不同的文件夹，但是一个构建脚本将会吧他们放在一起。

设置`baseUrl`告诉编译器哪去找模块，所有的非相对名字的模块导入都假设和`baseUrl`相对。

baseUrl 如下决定：

- 命令航参数 baseUrl 的值（如果给定路径是相对的，他是基于当前文件夹计算的）

- ‘tsconfig.json’中的 baseUrl 属性的值（如果路给定路径是相对的，它基于'tsconfig.json'的定位为计算）。

注意相对模块导入不暗示 baseUrl 被设置，因为他俩们总是解析为相对他们导入文件解析。


你可以在[RequireJS]()和[SystemJS]()文档找到更多关于 baseUrl 的文档。

#### 路径映射

有时候，模块不是直接基于 baseUrl 下定位。比如，导入一个模块`"jQuery"`将会在运行时翻译为`"node_modules/jquery/dist/jquery.slim.min.js"`。加载器使用一个映射配置在运行时去映射模块名到文件，查看[RequeJs 文档]()和[SystemJS 文档]()。

TypeScript 编译器使用`tsconfig.json`中的`"path"`支持这类映射。这是一个如何指定`jquery`的`"paths"`属性的例子。

```
{
  "compilerOptions": {
    "baseUrl": ".", // This must be specified if "paths" is.
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"] // This mapping is relative to "baseUrl"
    }
  }
}

```

注意`"paths"`相对于`"baseUrl"`解析。当设置`"baseUrl"`到非`"."`的值的时候，比如，`tsconfig.json`的文件夹，映射必须基于这个改变。也就是说，你在前面的例子设置`"baseUrl": "./src"`，jQuery 应该映射为`"../node_modules/jquery/dist/jquery"`。

使用`"paths"`也允许复杂的映射，包括多重回落定位。假设一个项目配置只有一些模块在一个地方可用，另一些在其他地方。一个构建步骤将把他们放在一起。项目布局可能像这样：
```
projectRoot
├── folder1
│   ├── file1.ts (imports 'folder1/file2' and 'folder2/file3')
│   └── file2.ts
├── generated
│   ├── folder1
│   └── folder2
│       └── file3.ts
└── tsconfig.json
```
对饮的`tsconfig.js`可能像这样：
```
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": ["*", "generated/*"]
    }
  }
}

```

这告诉编译器，任何匹配模式`"*"`的模块导入（比如，任何值），去两个地方寻找：

1. `"*"`意味着没有改变名字，因此`<moduleName>`=>`<baseUrl>/<moduleName>`

2. `"generated/*"`意味着模块名有一个前缀“generated”，因此，映射`<moduleName>`=>`<baseUrl>/generated/<moduleName>`

使用这个逻辑，编译器将会尝试如下解析两个导入：

import 'folder1/file2':

1. 模式‘*’匹配和命中整个模块名
2. 尝试列表中的第一个场景：'*'->`folder1/file2`
3. 这个场景是一个非相对名字 - 将它和 baseUrl 结合 -> `projectRoot/folder1/file2.ts`
4. 文件存在。完成。

import 'folder2/ffile3':

1. 模式'*'名字和匹配整个模块名
2. 首先尝试列表中的第一个场景：'*'->`folder2/file3`。
3. 场景的结果是非相对名字 - 将它和`baseUrl` 结合 -> `projectRoot/folder2/file3.ts`
4. 文件u 存在，移动到第二个场景
5. 第二个场景'generated/*'->`generated/folder2/file3`
6. 场景的结果是一个非相对名字 - 将它和 baseUrl 结合 -> `projectRoot/generated/folder2/file3.ts`
7. 文件存在，完成。

#### 使用 rootDirs 的虚拟文件夹

有时候来自多个文佳佳的源码在编译时别绑定生成到一个单独的输出文件夹。这可以认为一系列的源码文件夹创建了一个“虚拟”文件夹。

使用‘rootDirs’，你可以通知编译器创建这个“虚拟”文件夹；因此编译器可以解析相对模块导入，使用这些虚拟“文件夹”，就像合并到一个文件夹。

比如，假设这个文件夹结构：
```
 src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

`src/views`中的文件是为了一些 UI 控制的用户代码，`generated/templates`是 UI 模板绑定代码自动生成的，通过一个模板生成器，作为构建的一部分。一个构建步骤将会复制`/src/views`和`/generrated/templateds/views`到输出中相同的文件夹。在运行时，一个视图可以认为他的模板存在在他的附近，并且应该使用相对名字引入，就像`"./template"`。

为了给编译器指定这个关系，使用`"rootDirs"`。`"rootDirs"`指定一个列表的根，他们的内容将会在运行时被合并。因此，下面我们的例子，`tsconfig.json`温江看起来像这样：
```
{
  "compilerOptions": {
    "rootDirs": ["src/views", "generated/templates/views"]
  }
}
```

每当编译器发现一个相对模块导入是在`rootDirs`中的一个子文件夹，它将会尝试去在`rootDirs`的每一个入口查找导入。

`rootDirs`的灵活不仅仅局限于指定一个列表的逻辑合并的物理代码文件夹。可接受的数组可能包含任何数量的 ad hoc，任意文件夹名字，无论他们四否存在。这允许编译器去捕捉复杂的绑定和运行时特性，比如条件包含，和类型安全的项目特定加载器插件。

假设一个国际化场景，一个构建工具自动生成本地化指定包，通过Tina 集阿一个指定路径标记，叫做`#{locale}`，作为相对模块的一部分，比如`./#{locale}/message`。在这个假设的设置中，工具枚举支持本地化，映射到抽象的路径到`./zh/messages`，`./de/message`等。

假设每一个模块导出一个字符串数组。比如`./zh/message`可能包含：
```
export default ["您好吗", "很高兴认识你"];
```

通过利用`rootDirs`，我们可以通知编译器这个映射，从而允许它被安全的解析`./#{locale}/messages`，尽管文件夹根本不存在。比如，使用下面的`tsconfig.json`： 
```
{
  "compilerOptions": {
    "rootDirs": ["src/zh", "src/de", "src/#{locale}"]
  }
}
```

为了工具的目的，编译器现在将解析`import messages from './#{locale}/messages'`为`import messages from './zh/messages'`，允许开发者在一个区域无关的方式下不损失设计时间支持。

 
### 跟踪模块解析

就像前面讨论的，编译器可以访问当前文件夹之外的文件，当解析一个模块的时候。这在诊断为啥一个模块没有被解析的时候很难，或者加息到一个错误的定义。使用`--traceResolution`启用编译器模块解析跟踪，提供模块解析进程中的内部视角。

比如我们有一个应用，使用了`typescript`模块。`app.ts`有一个引入`import * as ts from "typescript"`。

```
│   tsconfig.json
├───node_modules
│   └───typescript
│       └───lib
│               typescript.d.ts
└───src
        app.ts

```

使用`--traceResolution`调用编译器
```
tsc --traceResolution
```
导致如下输出和：
```
======== Resolving module 'typescript' from 'src/app.ts'. ========
Module resolution kind is not specified, using 'NodeJs'.
Loading module 'typescript' from 'node_modules' folder.
File 'src/node_modules/typescript.ts' does not exist.
File 'src/node_modules/typescript.tsx' does not exist.
File 'src/node_modules/typescript.d.ts' does not exist.
File 'src/node_modules/typescript/package.json' does not exist.
File 'node_modules/typescript.ts' does not exist.
File 'node_modules/typescript.tsx' does not exist.
File 'node_modules/typescript.d.ts' does not exist.
Found 'package.json' at 'node_modules/typescript/package.json'.
'package.json' has 'types' field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.
File 'node_modules/typescript/lib/typescript.d.ts' exist - use it as a module resolution result.
======== Module name 'typescript' was successfully resolved to 'node_modules/typescript/lib/typescript.d.ts'. ========
```

需要关心的东西

- 导入的名字和定位

======== Resolving module ‘typescript’ from ‘src/app.ts’. ========

- 编译器使用的策略

Module resolution kind is not specified, using ‘NodeJs’.

- 从 npm 包加载的类型

‘package.json’ has ‘types’ field ‘./lib/typescript.d.ts’ that references ‘node_modules/typescript/lib/typescript.d.ts’.

- 最终结果

======== Module name ‘typescript’ was successfully resolved to ‘node_modules/typescript/lib/typescript.d.ts’. ========

### 使用 --noResolve

重唱编译器将会尝试去解析所有导入的木块，在他开始编译过程之前。每一次它成功解析一个`import`到一个文件，文件会被添加到文件集合，编译器将会在之后处理。

`--noResolve`编译器选项指示编译器不要去添加任何命令行没有传递的文件到编译器。它将会继续尝试去解析木块到文件，但是如果文件没有指定，他将不会被包含。

比如：

app.ts

```
import * as A from "moduleA"; // OK, 'moduleA' passed on the command-line
import * as B from "moduleB"; // Error TS2307: Cannot find module 'moduleB'.
```
```
tsc app.ts moduleA.ts --noResolve
```
使用`--noResolve`编译`app.ts`将会导致：
- 因为它在命令行被传递，将会正确找到`moduleA`
- 因为它没有被传递，所以找不到`moduleB`

### 常见问题

#### 为什么在排除列表的模块依旧被编译器引用？

`tsconfig.json`转化一个文件夹到“项目”。没有指定任何`"exclude"`或者`"files"`入口，包含`tsconfig.json`的文件夹和它的子文件夹的文件都包含在你的编译中。如果你想要去排除一些文件，使用`"exclude"`，如果你想要指定所有的文件，而不是让编译器去寻找他们，使用`"files"`。

这就是`tsconfig.json`自动包含。这没有嵌入到前面讨论的模块解析。如果编译器标示一个文件作为模块导入，他将会被包含在编译中，无论它是否在前面的步骤被排除。

所以，为了从编译排除一个文件，你需要去排除它，并且所有的你文件有一个`import`或者`/// <reference path="..." />`重定向到它。