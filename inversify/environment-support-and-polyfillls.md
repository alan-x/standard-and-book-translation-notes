### 环境支持和 polyfill

InversifyJS 需要一个现代 JavaScript 引擎，支持[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)，[Metadata Reflection API](http://rbuckton.github.io/ReflectDecorators/#reflect)和[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)对象。如果你的环境不支持其中一个，你需要去引入一个 shim 或者 polyfill。

#### Metadata Reflection API

> ⚠️ `reflect-metadata` polyfill 在你的整个应用中只需要引入一次，因为 Reflect 对象是一个全局单例。更多细节可以在[这里](https://github.com/inversify/InversifyJS/issues/262#issuecomment-227593844)找到。

总是需要。使用[reflect-metadata](https://www.npmjs.com/package/reflect-metadata)作为 polyfill。

```ts
$ npm install reflect-metadata
```

reflect-metadata 的类型定义包含在 npm 包。你需要在你的`tsconfig.json`添加下面的引用：
```ts
"types": ["reflect-metadata"]
```
最后，引入 reflect-metadata。如果你使用 Node.js，你可以使用：
```ts
import "reflect-metadata";

```
如果你在一个 web 浏览器使用，你可以使用一个 script 标签：
```ts
<script src="./node_modules/reflect-metadata/Reflect.js"></script>

```
这会创建一个 Reflect 全局对象。

#### Map

当使用 InversifyJS 3 或者更高的时候，[Maps](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)是需要的。

大部分现代 JavaScript 引擎支持 map，但是如果你需要在旧版浏览器支持，你将会需要一个 map polyfill（比如，[es-map](https://www.npmjs.com/package/es6-map)）。

#### Promise
如果你想要做以下的事情，[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)是必须的：

[注入一个 provider](https://github.com/inversify/InversifyJS/blob/master/wiki/provider_injection.md)或[注入异步动态值](https://github.com/inversify/InversifyJS/blob/master/wiki/value_injection.md)。

处理[post construction](https://github.com/inversify/InversifyJS/blob/master/wiki/post_construct.md)和[activation](https://github.com/inversify/InversifyJS/blob/master/wiki/activation_handler.md)，或异步的[pre destory](https://github.com/inversify/InversifyJS/blob/master/wiki/pre_destroy.md)和[deactivation](https://github.com/inversify/InversifyJS/blob/master/wiki/deactivation_handler.md)。

大部分现代 JavaScript 引擎支持 promise，但是如果你需要支持旧的浏览器，你需要使用一个 promise polyfill（比如，[es6-promise](https://github.com/stefanpenner/es6-promise)或[bluebird](https://www.npmjs.com/package/bluebird)）。

#### Proxy

[Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 只有在你想要[注入一个 proxy](https://github.com/inversify/InversifyJS/blob/master/wiki/activation_handler.md)的时候才需要。

如今（2016年9月）代理支持的并不是很好，很可能需要使用一个 proxy polyfill。比如，我们使用[harmony-proxy](https://www.npmjs.com/package/harmony-proxy)作为 polyfill 去运行我们的单元测试。