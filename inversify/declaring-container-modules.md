### 声明容器模块

在超大型应用中，容器模块可以帮助你管理复杂的绑定。

ContainerModule 和 AsyncContainerModule 的构造器参数是一个注册回调，传递一个函数，表现的和 Container 类的方法一样。AsyncContainerModule 注册器回调是异步的。

当一个容器模块加载进一个 Container 的时候，注册回调被调用。这是容器模块注册绑定和处理器的机会，ContainerModule 实例使用 Container 的 load 方法，AsyncContainerModule 实例使用 Container 的 loadAsync 方法。

当你一个容器模块从一个容器卸载，这个容器添加的绑定将会被移除，并且[失活进程]()将会发生在他们的每一个。容器失活和[失活处理器]()将会被移除。当有异步失活处理器或者异步[前置摧毁]()的时候，使用 unloadAsync 方法去卸载。

### 同步容器模块

```ts
let warriors = new ContainerModule((bind: interfaces.Bind, unbind: interfaces.Unbind) => {
    bind<Ninja>("Ninja").to(Ninja);
});

let weapons = new ContainerModule(
    (
        bind: interfaces.Bind,
        unbind: interfaces.Unbind,
        isBound: interfaces.IsBound,
        rebind: interfaces.Rebind,
        unbindAsync: interfaces.UnbindAsync,
        onActivation: interfaces.Container["onActivation"],
        onDeactivation: interfaces.Container["onDeactivation"]
    ) => {
        bind<Katana>("Katana").to(Katana);
        bind<Shuriken>("Shuriken").to(Shuriken);
    }
);

let container = new Container();
container.load(warriors, weapons);
container.unload(warriors);
```

### 异步容器模块
```ts
let warriors = new AsyncContainerModule(async (bind: interfaces.Bind, unbind: interfaces.Unbind) => {
    const ninja = await getNinja();
    bind<Ninja>("Ninja").toConstantValue(ninja);
});

let weapons = new AsyncContainerModule(
    (
        bind: interfaces.Bind,
        unbind: interfaces.Unbind,
        isBound: interfaces.IsBound,
        rebind: interfaces.Rebind,
        unbindAsync: interfaces.UnbindAsync,
        onActivation: interfaces.Container["onActivation"],
        onDeactivation: interfaces.Container["onDeactivation"]
    ) => {
        bind<Katana>("Katana").to(Katana);
        bind<Shuriken>("Shuriken").to(Shuriken);
    }
);

let container = new Container();
await container.loadAsync(warriors, weapons);
container.unload(warriors);
```