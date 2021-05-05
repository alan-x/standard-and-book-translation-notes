### 怎样从 4.x 升级到 5.x

- 4.x `guid(): string` 方法被`id(): number`覆盖。
- 属性`guide: string`被下面的[接口](https://github.com/inversify/InversifyJS/blob/master/src/interfaces/interfaces.ts)和他们的实现`id: number`替代

    - [Binding](https://github.com/inversify/InversifyJS/blob/master/src/bindings/binding.ts)
    - [Context](https://github.com/inversify/InversifyJS/blob/master/src/planning/context.ts)
    - [Request](https://github.com/inversify/InversifyJS/blob/master/src/planning/request.ts)
    - [Target](https://github.com/inversify/InversifyJS/blob/master/src/planning/target.ts)
    - [Container](https://github.com/inversify/InversifyJS/blob/master/src/container/container.ts)
    - [ContainerModule](https://github.com/inversify/InversifyJS/blob/master/src/container/container_module.ts)
    - [AsyncContainerModule](https://gier/container_module.ts)

### 怎么从 2.x 到 3.x

- 2.x `Kernel` 在 3.x 中命名为`Container`
- 2.x `Kernel`方法`getServiceIdentifierAsString`在 3.x 中不是`Container`的方法
- 2.x `PlanAndResolveArgs`接口在 3.0 中命名为`NextArgs`，并且有一些属性被改变了。
- `Provider`签名被修改
- 在 3.x，`strictNullChecks`被启用。
- 为了去支持新的特性，比如可选的依赖和默认上下文注入，解析协议在 2.0 和 3.0 有一些不同。

### 怎么从 1.x 升级到 2.x

2.x 版本在 API 上引入了一些改变。

#### 命名改变
1.x `TypeBinding` 在 2.x 中命名为 `binding`
1.x `BindingScopeEnum`在 2.x 中命名为`BindingScope`

#### 顺畅的绑定语法
1.x 绑定语法看起来像下面：
```ts
container.bind(new TypeBinding<FooInterface>("FooInterface", Foo, BindingScopeEnum.Transient));
```
2.x 绑定语法看起来像下面：
```ts
container.bind<FooInterface>("FooInterface").to(Foo).inTransientScope()

```

#### 解析语法
1.x `container.resolve<T>(identifier: string)`方法在 2.x 中是`container.get<T>(identifier: string)`

1.x 解析语法看起来像下面：
```ts
var foobar = container.resolve<FooBarInterface>("FooBarInterface");

```
2.x 解析语法看起来像下面：
```ts
var foobar = container.get<FooBarInterface>("FooBarInterface");

```

#### @injectable 和 @inject

你所有的类必须使用`@injectable()`装饰器去声明。如果你的类有一个依赖，那这就足够了。
> 原文（感觉是有问题的，应该是没有依赖才对）：All your classes must be decorated with the @injectable() decorator. If your class has a dependency in a class that's enough:
```ts
@injectable()
class Katana {
    public hit() {
        return "cut!";
    }
}

@injectable()
class Shuriken {
    public throw() {
        return "hit!";
    }
}

@injectable()
class Ninja implements Ninja {

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

如果你的类有一个依赖接口，你需要使用`@inject`装饰器

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