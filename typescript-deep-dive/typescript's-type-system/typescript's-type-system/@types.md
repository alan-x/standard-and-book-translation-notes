# @types

[Definitely Typed]()的确是 TypeScript 的最强大的力量之一。社区有效向前推进，并且记录了接近 90% 顶部 JavaScript 项目。

这意味着你可以使用这些项目，以非常交互和探索的方式，不需要去在分离的窗口打开文档并确保你不需要编码。


### 使用`@types`

安装十分简单，因为它只是基于`npm`工作。因此作为一个例子，你可以为`jquery`简单安装类型定义，就像：
```ts
npm install @types/jquery --save-dev
```
`@types`支持全局和模块类型定义

### 全局`@types`

默认，任何支持全局消费的定义都自动包含。比如，对于`jquery`，你应该可以在你的项目全局使用`$`。

然而，对于库（比如，`jquery`），我通常推荐使用模块：

### 模块`@types`

在安装之后，没有特殊的配置需要，你就像一个模块一样使用，比如：
```ts
import * as $ from "jquery";

// Use $ at will in this module :)
```

### 控制全局

正如可以看到的，有一个允许全局自动泄漏的定义对于一些团队可能是问题。因此你可以选择显示的只引入有意义的类型，使用`tsconfig.json``compilerOptions.types`，比如：
```ts
{
    "compilerOptions": {
        "types" : [
            "jquery"
        ]
    }
}
```
前面的例子显示了只有`jquery`允许被使用。就算人们安装其他类似`npm install @types/node`，他的全局（比如，`process`）将不会泄漏到你的代码，知道你添加他们到`tsconfig.json`类型选项。


