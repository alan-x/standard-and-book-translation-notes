### 提交构造装饰器

为一个类方法添加一个 @postConstruct 装饰器是可能的。这个装饰器将会在一个对象初始化之后，任何激活处理器之前运行。这在构造器被调用，但是组件还没被初始化之前的场景很有用，或者你想要在构造器执行之后执行一些初始化逻辑的场景。

在其他场景中，它给你一个约束，保证在这个对象的整个声明周期中这个方法将只调用一次。查看下面的使用例子。

方法可以是异步或者同步：
```ts
interface Katana {
    use: () => void;
}

@injectable()
class Katana implements Katana {
    constructor() {
        console.log("Katana is born");
    }
    
    public use() {
        return "Used Katana!";
    }
    
    @postConstruct()
    public testMethod() {
        console.log("Used Katana!")
    }
}
```
```ts
container.bind<Katana>("Katana").to(Katana);

```
```ts
let catana = container.get<Katana>();
> Katana is born
> Used Katana!
```

注意你不能在同一个类使用多次 @postConstruct 装饰器。它将会抛出一个错误
```ts
class Katana {
    @postConstruct()
        public testMethod1() {/* ... */}

    @postConstruct()
        public testMethod2() {/* ... */}
    }
            
Katana.toString();
> Error("Cannot apply @postConstruct decorator multiple times in the same class")

```

使用基础 JavaScript
```ts
inversify.decorate(inversify.postConstruct(), Katana.prototype, "testMethod");

```