### 为什么是 InversifyJS

有很多使用 InversifyJS 的好理由，但是我们想强调几点：

### 1. 真正的解耦

InversifyJS 提供真正的解耦。考虑下面的类：
```ts
let TYPES = {
  Ninja: Symbol.for("Ninja"),
  Katana: Symbol.for("Katana"),
  Shuriken: Symbol.for("Shuriken")
};

export { TYPES };
```

```ts
import { TYPES } from "./constants/types";

@injectable()
class Ninja implements Ninja {

    private _katana: Katana;
    private _shuriken: Shuriken;

    public constructor(
        @inject(TYPES.Katana) katana: Katana,
        @inject(TYPES.Shuriken) shuriken: Shuriken
    ) {
        this._katana = katana;
        this._shuriken = shuriken;
    }

    public fight() { return this._katana.hit(); };
    public sneak() { return this._shuriken.throw(); };

}
```

`Ninja`类将永远不会指向`Katana`或`Shuriken`类。然而，它将会指向接口（设计态）或者 Symbols（运行态）是允许的，因为这些抽象和[依赖抽象](https://en.wikipedia.org/wiki/Dependency_inversion_principle)就是 DI 的全部。

InversifyJS 容器是应用中唯一需要关心生命周期和依赖的。我们推荐将这个放到命名为`inversify.config.ts`的文件，并存储文件到包含整个应用的源代码的根文件夹：
```ts
import { TYPES } from "./constants/types";
import { Katana } from "./entitites/katana";
import { Shuriken } from "./entitites/shuriken";
import { Ninja } from "./entitites/ninja";

container.bind<Katana>(TYPES.KATANA).to(Katana);
container.bind<Shuriken>(TYPES.SHURIKEN).to(Shuriken);
container.bind<Ninja>(TYPES.NINJA).to(Ninja);
```

这意味着你应用中所有的耦合都发生在一个单独的地方：`inversify.config.ts`文件。这很重要，我们将使用一个例子来争鸣它。假设我们要改变游戏的难度。我们只需要去`inversify.config.ts`，并改变 Katana 绑定：
```ts
import { Katana } from "./entitites/SharpKatana";

if(difficulty === "hard") {
    container.bind<Katana>(TYPES.KATANA).to(SharpKatana);
} else {
    container.bind<Katana>(TYPES.KATANA).to(Katana);
}
```

你不需要改变 Ninja 文件！

代价使用 Symbols 或者字符串字面量，但是这个代价很小，如果你声明你的所有字符串字面量到一个包含常量的文件（[比如 Redux 中的 actions](https://github.com/reactjs/redux/blob/master/examples/todomvc/src/constants/ActionTypes.js)）。好消息是在未来，symbols 或者字符串字面量[最终将会被 TS 编译器生成](https://github.com/Microsoft/TypeScript/issues/2577)，但是目前这还在 TC39 会员会手中。

### 2.解决竞争问题

一些“旧”的 JavaScript IoC 容器，像 angular 1.x `$injector`有一些问题：

- 内部缓存 - 依赖单例提供。无论何时请求服务，在整个生命周期中，它只创建一次。创建工厂及其非常公平

- 命名空间冲突 - 一个应用中，只能有一个“类型”的词素。如果我们有一个 car 服务，然后有一个第三方扩展引入一个想通名字的服务，我们有一个问题。

- 构建到框架 - Angular 1 的 DI 直接结合进了框架。没有给我们解耦作为一个单例系统的方式。

[源码](https://angular.io/docs/ts/latest/guide/dependency-injection.html)

InversifyJS 解决这些问题：

- 支持临时和单例范围
- 感谢标签，命名和上下文绑定，没有命名冲突
- 是一个单独的库

### 3. 你需要的所有特性

据我所知，这是 JavaScript 唯一的 IoC 容器，具有复杂的依赖处理（比如，上下文绑定），多范围（临时，单例）和很多其他特性。最重要的是，还有很多成长空间，比如拦截或者 web worker 范围。我们也有计划为开发者提供开发工具。比如浏览器扩展和中间件（日志，缓存...）。

### 4. 对象组合很痛苦

你可能觉得你不需要一个 IoC 容器。

如果[上面的论述](http://stackoverflow.com/questions/871405/why-do-i-need-an-ioc-container-as-opposed-to-straightforward-di-code)不足够，你可能想要阅读下面：
- [JavaScript 依赖导致的现状](http://blog.wolksoftware.com/the-current-state-of-dependency-inversion-in-javascript)
- [TypeScript/ES 中关于面向对象特性和“class”和“extends”关键字](http://blog.wolksoftware.com/about-classes-inheritance-and-object-oriented-design-in-typescript-and-es6)


### 5. 类型安全

这个库使用 TypeScript 开发，因此类型安全开箱即用，如果你使用 TypeScript，值得提醒的是，如果你想要注入一个 Katana 到一个类，期待一个`Shuriken`实现，你将会得到一个编译错误。

### 6. 良好的开发体验

我们努力提供为你的 JavaScript 应用提供一个好的 IoC 容器和一个好的开发体验。我们花了很多时间，尝试让 InversifyJS 变的尽可能对用户友好，正在努力开发 Chrome 工具，我们已经开发了一个日志中间件去帮助你在 Node.js 中调试。

![](https://camo.githubusercontent.com/4e0fbd05ef535409a1b7456baee15a3c6f69ed9a479f423124a61e31a250fdf6/687474703a2f2f696e766572736966792e696f2f696d672f646576746f6f6c73312e706e67)