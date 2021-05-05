### 循环依赖

#### 你模块中的循环依赖（ES6， CommonJS，etc）

如果你的两个模块有一个循环依赖，并且你使用`@inject(SomeClass)`注解。运行时，一个模块将会在其他模块之前转化，装饰器将会使用`@inject(SomeClass /* SomeClass = undefined*/)`调用。InversifyJS 将会抛出下面的异常：

> @inject 使用 undefined 调用，意味着类 ${name} 有一个循环依赖问题。你可以使用一个 LazyServiceIdentifer 去克服这个限制。

有两个方式去克服这个限制：
- 使用一个`LazyServiceIdentifer`。lazy 标识符不会延迟依赖的注入，所有的依赖都在类实例创建的时候注入。然而，它的确延迟了属性标识符的访问（解决模块问题）。这个问题的一个例子可以在[我们的单元测试](https://github.com/krzkaczor/InversifyJS/blob/a53bf2cbee65803b197998c1df496c3be84731d9/test/inversify.test.ts#L236-L300)找到。

- 使用`@lazyInject`装饰器。这个装饰器是`[inversify-inject-decorators](https://github.com/inversify/inversify-inject-decorators)`模块的一部分。`@lazyInject`装饰器延迟依赖的注入，知道他们真实使用，这发生在类实例被创建之后。

### 循环依赖在依赖图中

InversifyJS 能够标识循环依赖将会抛出异常去帮助你标示问题的定位，如果一个循环依赖被捕获：
```ts
Error: Circular dependency found: Ninja -> A -> B -> C -> D -> A
```