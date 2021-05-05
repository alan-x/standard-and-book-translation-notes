### 架构概览

这个页面创建使为了让贡献这个生活更容易。

InversifyJS 的内部架构被 [Ninject](https://github.com/ninject/Ninject)严重影响。被影响不意味着两个架构完全相同。实际上，两个架构非常不同，因为 C# 和 JavaScript 使非常不同的编程语言。然而，用于描述这个库的某些元素的术语和解析过程的没有啥不同。


### 解析过程

InversifyJS 在解析一个依赖之前执行三个强制操作：
- Annotation
- Planning
- Middleware (optional)
- Resolution
- Activation (optional)

在某些场景，有一些额外操作（中间件和激活/失活）。

项目文件夹架构根据解析过程的阶段关联在一起：
```ts
├── src
│   ├── annotation
│   │   ├── context.ts
│   │   ├── metadata.ts
│   │   ├── queryable_string.ts
│   │   ├── request.ts
│   │   └── target.ts
│   ├── bindings
│   │   ├── binding.ts
│   │   ├── binding_count.ts
│   │   └── binding_scope.ts
│   ├── constants
│   │   ├── error_msgs.ts
│   │   └── metadata_keys.ts
│   ├── decorators
│   │   ├── decorator_utils.ts
│   │   ├── inject.ts
│   │   ├── named.ts
│   │   ├── target_name.ts
│   │   └── tagged.ts
│   ├── interfaces
│   │   └── ...
│   ├── inversify.ts
│   ├── container
│   │   ├── container.ts
│   │   ├── key_value_pair.ts
│   │   ├── lookup.ts
│   │   ├── plan.ts
│   │   ├── planner.ts
│   │   └── resolver.ts
│   └── middleware
│       └── logger.ts
```

### 激活阶段

注解阶段读取装饰器生成的元数据，将它转化到一些列的 Request 和 Target 类实例。Request 和 Target 实例之后在 Planning 阶段用于解析计划。

### Planning 阶段

当我们调用一个 Container ‘get’ 方法，比如：
```ts
var obj = container.get<SomeType>("SomeType");
```

我们开始一个新的解析，这意味着容器将会创建一个新的解析上下文。这个解析上下文容器包含一个容器的引用，和一个 Plan 的应用。

一个 Plan 包含一个上下文的索引和一个 Request 的索引。一个请求表示一个将会被注入到 Target 的依赖。

来看看下面的代码片段：
```ts
@injectable()
class FooBar implements FooBarInterface {
  public foo : FooInterface;
  public bar : BarInterface;
  public log() {
    console.log("foobar");
  }
  constructor(
    @inject("FooInterface") foo : FooInterface, 
    @inject("BarInterface") bar : BarInterface
  ) {
    this.foo = foo;
    this.bar = bar;
  }
}

var foobar = container.get<FooBarInterface>("FooBarInterface");
```

前面的代码片段将会创建一个新的 Context 和新的 Plan。plan 将会包含一个根 Request，它的 Target 使 null，还有两个子 Request：

- 第一个子 request 标示一个 FooInterface 依赖，它的目标使构造器参数名 `foo`
- 第二个子 request 标示一个 BarInterface 依赖，它的目标是构造器参数名 `bar`

下面的图可以帮助你理解饿极细 Context 的形状，还有内部使如何互相应用的：

![](https://camo.githubusercontent.com/eab45ec6a577c24010844eedf9a9ecc6dbc116ac5286771193457e10ce4d0aee/687474703a2f2f692e696d6775722e636f6d2f4e5353625057792e706e67)

### 中间件阶段

如果我们配置了一些 Middleware，它将会在解析阶段发生之前执行。Middleware 可以用于开发一些浏览器扩展，允许我们去展示解析计划，使用一些数据可视化工具，比如 D3.js。这类工具将会帮助开发者标示处开发过程的问题。

[inversify-logger-middleware](https://github.com/inversify/inversify-logger-middleware)就是中间件的例子，可以用于在控制台展示解析计划创建和解析的事件：

![](https://camo.githubusercontent.com/d5903f1cf5a10746e3e1c6259ea151a7e62c89b2a3300b0d42877fe5bfba7ef4/687474703a2f2f692e696d6775722e636f6d2f6946416f67726f2e706e67)

### Resolution Phase

Plan 用于解析。解析江西继续解析 Request 树的每一个依赖，从叶子开始，到根 Request 结束。

解析过程将会是异步/同步，这有助于提升性能

### Activation Phase

Activation 在依赖被解析之后发生。在它被添加到缓存（如果单例或者请求单例 - [查阅范围](https://github.com/inversify/InversifyJS/blob/master/wiki/scope.md)）和注入之前。添加一个事件处理器在 activation 完成之前使可能的。这个特性允许开发者做点类似注入一个代理拦截对象所有的属性或者方法调用。[activation 处理器](https://github.com/inversify/InversifyJS/blob/master/wiki/activation_handler.md)将不会调用，如果类型从缓存解析。activation 处理器可以是同步或者异步。


### Deactivation Phase

deactivation 阶段发生在 Container 方法 unbind/unbindAsync/unbindAll/unbindAllAsync 调用的时候。Deactivation 也发正在 [模型模块](https://github.com/inversify/InversifyJS/blob/master/wiki/container_module.md)调用 unbind 和 unbindAsync 注册参数或者当一个容器模块从 Container unload 的时候。为一个绑定到单例范围的类型绑定添加一个 [deactivation 处理器](https://github.com/inversify/InversifyJS/blob/master/wiki/deactivation_handler.md)。处理器可以是同步的或者异步的。

