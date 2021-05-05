### 控制依赖的范围

InversifyJS 默认使用临时范围，但是你可以使用 singleton 和 request 范围：
```ts
container.bind<Shuriken>("Shuriken").to(Shuriken).inTransientScope(); // Default
container.bind<Shuriken>("Shuriken").to(Shuriken).inSingletonScope();
container.bind<Shuriken>("Shuriken").to(Shuriken).inRequestScope();
```

#### 关于`inSingletonScope`

有很多可以用的绑定类型
```ts
interface BindingToSyntax<T> {
    to(constructor: { new (...args: any[]): T; }): BindingInWhenOnSyntax<T>;
    toSelf(): BindingInWhenOnSyntax<T>;
    toConstantValue(value: T): BindingWhenOnSyntax<T>;
    toDynamicValue(func: (context: Context) => T): BindingWhenOnSyntax<T>;
    toConstructor<T2>(constructor: Newable<T2>): BindingWhenOnSyntax<T>;
    toFactory<T2>(factory: FactoryCreator<T2>): BindingWhenOnSyntax<T>;
    toFunction(func: T): BindingWhenOnSyntax<T>;
    toAutoFactory<T2>(serviceIdentifier: ServiceIdentifier<T2>): BindingWhenOnSyntax<T>;
    toProvider<T2>(provider: ProviderCreator<T2>): BindingWhenOnSyntax<T>;
}
```

根据范围行为，我们可以将这些类型的绑定分为两个主要的组：
- 绑定将会注入一个`object`，
- 绑定将会注入一个`function`

#### 绑定将会注入一个`object`

这个分组包含下面类型的绑定：
```ts
interface BindingToSyntax<T> {
    to(constructor: { new (...args: any[]): T; }): BindingInWhenOnSyntax<T>;
    toSelf(): BindingInWhenOnSyntax<T>;
    toConstantValue(value: T): BindingWhenOnSyntax<T>;
    toDynamicValue(func: (context: Context) => T): BindingInWhenOnSyntax<T>;
}

```

默认使用`inTransientScope`，我们可以选择这个类型的绑定，除了`toConstantValue`，它总是使用`inSingletonScope`。

当我们第一次调用`container.get`，并且我们使用`to`，`toSelf`或者`toDynamicValue`，InversifyJS 容器将会尝试生成一个对象实例，或者使用构造器的值或者动态值工厂。如果范围设置为`inSingletonScope`，值将会被缓存。第二次我们为想通资源 ID 调用`container.get`的时候，如果`inSingletonScope`被选择，InversifyJS 将会尝试从缓存获取值。

注意一个类可以有一些依赖，并且一个动态值可以通过当前上下文访问其他类型。这些依赖可能是或者不是单例依赖，独立于他们父对象格子组合树选择的范围。

#### 将会注入一个`function`的绑定

这个分组包含下面类型的绑定：
```ts
interface BindingToSyntax<T> {
    toConstructor<T2>(constructor: Newable<T2>): BindingWhenOnSyntax<T>;
    toFactory<T2>(factory: FactoryCreator<T2>): BindingWhenOnSyntax<T>;
    toFunction(func: T): BindingWhenOnSyntax<T>;
    toAutoFactory<T2>(serviceIdentifier: ServiceIdentifier<T2>): BindingWhenOnSyntax<T>;
    toProvider<T2>(provider: ProviderCreator<T2>): BindingWhenOnSyntax<T>;
}

```

我们不能选择这个类型的绑定的范围，因为被注入的值（一个工厂`function`）总是一个单例。然而，工厂内部实现可能是或者不是一个单例。

比如，下面的绑定将会注入一个总是单例的工厂。
```ts
container.bind<interfaces.Factory<Katana>>("Factory<Katana>").toAutoFactory<Katana>("Katana");

```
然而，工厂返回的值可能是或者不是一个单例：
```ts
container.bind<Katana>("Katana").to(Katana).inTransientScope();
// or
container.bind<Katana>("Katana").to(Katana).inSingletonScope();
```

#### 关于`inRequestScope`

当我们使用 inRequestScope，我们使用特殊类型的单例。

- `inSingletonScope`创建一个持续整个生命周期的类型绑定单例。这意味着`inSingletonScope`可以使用`container.unbind`从内存解绑一个类型绑定。

- 一旦调用`container.get`，`container.getTagged`，`container.getNamed`方法，`inRequestScope`创建一个整个生命周期的类型绑定单例。这些方法的调用将会解析一个根依赖和它的所有子依赖。内部，InversifyJS 创建一个被称为“解析计划”的依赖图。这减少需要解析的数量，并且在某些场景，可以用于性能优化。