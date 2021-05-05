### 安装

你可以使用 npm 得到最新的发行和类型定义：
```ts
npm install inversify@5.0.5 reflect-metadata --save
```
或者使用 yarn：
```ts
yarn add inversify@5.0.5 reflect-metadata
```

InversifyJS 类型定义包含在 inversify npm 包。InversifyJS 需要在你的`tsconfig.json`文件中包含`experimentalDecorators`，`emitDecoratorMetadata`，和`lib`编译选项。

```ts
{
    "compilerOptions": {
        "target": "es5",
        "lib": ["es6"],
        "types": ["reflect-metadata"],
        "module": "commonjs",
        "moduleResolution": "node",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

InversifyJS 需要一个现代化 JavaScript 引擎支持：
- [Reflect metadata](https://github.com/rbuckton/ReflectDecorators/blob/master/spec/metadata.md)
- [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)（只有在使用[provider injection](https://github.com/inversify/InversifyJS#injecting-a-provider-asynchronous-factory)的时候才需要）
- [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)（只有在使用[activation handles](https://github.com/inversify/InversifyJS/blob/master/wiki/activation_handler.md)的是有才需要）

如果你的环境不支持其中一个，才需要引入一个 shim 或者 polyfill。

在 wiki 页面查阅[环境支持和 polifills](https://github.com/inversify/InversifyJS/blob/master/wiki/environment.md)页面了解更多。