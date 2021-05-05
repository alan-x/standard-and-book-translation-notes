InversifyJS 在解析一个依赖之前执行三个必须的操作：
- Annotation
- Planning
- Resolution

在某些场景下，有一些额外操作：
- Activation
- Middleware

如果我们配置一些 Middleware，它将会在某些点执行，planning，resolution 和 activation 阶段之前或者之后。

中间件可以用于实现强大的开发工具。这些工具将会帮助开发者在开发过程标识问题。

### 基础中间件
```ts
import { interfaces, Container } from "inversify";

function logger(planAndResolve: interfaces.Next): interfaces.Next {
    return (args: interfaces.NextArgs) => {
        let start = new Date().getTime();
        let result = planAndResolve(args);
        let end = new Date().getTime();
        console.log(`wooooo  ${end - start}`);
        return result;
    };
}

let container = new Container();
container.applyMiddleware(logger);
```

注意我们声明了一个中间件，可以创建一个新的 Container 并使用 applyMiddleware 方法去应用它：
```ts
interface Ninja {}

@injectable()
class Ninja implements Ninja {}

let container = new Container();
container.bind<Ninja>("Ninja").to(Ninja);

container.applyMiddleware(logger);
```

日志中间件将会在控制台记录执行时间：
```ts
let ninja = container.get<Ninja>("Ninja");

> 21
```

### 多中间件函数

当多个中间件函数被应用
```ts
container.applyMiddleware(middleware1, middleware2);

```

中间件将会从右到左执行。这意味着`middleware2`将会在`middleware1`之前调用。

### 上下文拦截

有时候，你可能想要拦截某些解析计划。默认`contextInterceptor`作为`args`的一个属性传递到中间件。
```ts
function middleware1(planAndResolve: interfaces.Next): interfaces.Next<any> {
    return (args: interfaces.NextArgs) => {
        // args.nextContextInterceptor
        // ...
    };
}
```

你可以使用函数扩展默认`contextInterceptor`：
```ts
function middleware1(planAndResolve: interfaces.Next<any>): interfaces.Next<any> {
    return (args: interfaces.NextArgs) => {
        let nextContextInterceptor = args.contextInterceptor;
        args.contextInterceptor = (context: interfaces.Context) => {
            console.log(context);
            return nextContextInterceptor(context);
        };
        return planAndResolve(args);
    };
}
```

### 自定义元数据阅读器

> ⚠️ 请注意，不推荐创建你自己的自定义元数据阅读器。我们包含了这个特性是为了允许库/框架创建者去有一个更高级别的自定义，但是大部分用户不应该使用一个自定义元数据阅读器。通常，自定义元数据阅读器应该只被用于开发一个框架，为了提供一个必默认注解 API 更明确的 API。

> 如果你开发一个框架或者库，你创建一个自定义元数据阅读器，轻记住为你的框架在默认 API 提供一个可替代的装饰器：`@injectable`，`@inject`，`@multiInject`，`@tagged`，`@named`，`@optional`，`@targetName`和`@unmanaged`。

中间件允许你去拦截一个计划，并解析他，但是你不允许去改变 annotation 阶段方式的行为。

这是第二个扩展点，允许你去决定你想要使用哪一种注解系统。默认注解系统是装饰器和 reflect-metadata 实现的：
```ts
@injectable()
class Ninja implements Ninja {

    private _katana: Katana;
    private _shuriken: Shuriken;

    public constructor(
        @inject("Katana") katana: Katana,
        @inject("Shuriken") shuriken: Shuriken
    ) {
        this._katana = katana;
        this._shuriken = shuriken;
    }

    public fight() { return this._katana.hit(); };
    public sneak() { return this._shuriken.throw(); };

}
```

你可以使用自定义元数据阅读器去实现一个自定义注解系统。

比如，你可以基于静态属性实现一个注解系统：
```ts
class Ninja implements Ninja {

    public static constructorInjections = [
        "Katana", "Shuriken"
    ];

    private _katana: Katana;
    private _shuriken: Shuriken;

    public constructor(
        katana: Katana,
        shuriken: Shuriken
    ) {
        this._katana = katana;
        this._shuriken = shuriken;
    }

    public fight() { return this._katana.hit(); };
    public sneak() { return this._shuriken.throw(); };

}
```

一个自定义元数据阅读器必须实现`interfaces.MetadataReader`接口。

一个完整例子[可以在我们的单元测试找到](https://github.com/inversify/InversifyJS/blob/master/test/features/metadata_reader.test.ts)。

一旦你有一个自定义元数据阅读器，你将会准备去应用它：
```ts
let container = new Container();
container.applyCustomMetadataReader(new StaticPropsMetadataReader());
```