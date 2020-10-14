# 类型实例化

假设你有一个泛型参数，比如一个类`Foo`：
```ts
class Foo<T>{
    foo: T;
}
```
你想要为一个特定的类型创建一个指定版本的类型。这个模式是去赋值项到一个新的变量，并给他类型声明，使用具体的类型覆盖泛型。比如，如果你想要一个类`Foo<number>`：
```ts
class Foo<T>{
    foo: T;
}
let FooNumber = Foo as { new ():Foo<number> }; // ref 1
```

在`ref 1`，你可以说`FooNumber`和`Foo`相同，但是只是对待它为一些东西，当调用`new`操作的时候，给一个`Foo<Number>`的实例。

### 继承

类型断言毛事是不安全的，它相信你去做正确的东西。对于一个类，其他语言一个常见的模式是使用继承：
```
class FooNumber extends Foo<number>{}
```

一句话警告：如果你在基类上使用装饰器，则继承类可能没有和基类相同的行为（它不再被装饰器包裹）。

当然，如果你不指定类，你依旧需要一个强制/断言模式，因此我们先展示常见的断言模式，比如：
```ts
function id<T>(x: T) { return x; }
const idNum = id as {(x:number):number};
```

> 受这个[stackoverflow 问题]()启发