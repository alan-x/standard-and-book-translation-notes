[已校对]
# global.d.ts

我们讨论的全局 vs 文件模块，当我们覆盖[项目](https://basarat.gitbook.io/typescript/project/modules)的时候，推荐使用基于文件的模块，不污染全局命名空间。

然而，如果你有 TypeScript 开发新手，你可以给他们一个`global.d.ts`文件去放置接口/类型在全局命名空间，让获取一些类型简单，可以在你所有的 TypeScript 代码可用。

`global.d.ts`另一个使用场景是声明编译时常量，Webpack 通过标准[DefinePlugin](https://webpack.js.org/plugins/define-plugin/)插件注入到源代码

```ts
declare const BUILD_MODE_PRODUCTION: boolean; // can be used for conditional compiling
declare const BUILD_VERSION: string;

```

> 对于任何生成 JavaScript 的代码，我们强烈推荐使用文件模块，只使用`global.d.ts`去声明编译时常量和/或去扩展声明在`lib.d.ts`中国的标准类型声明。


- 奖励：`global.d.ts`文件对于快速升级 JS 到 TS 声明`declare module "some-library-you-dont-care-to-get-defs-for";`也很好。