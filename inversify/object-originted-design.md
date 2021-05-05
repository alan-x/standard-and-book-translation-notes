### 面向对象设计

InversifyJS 是一个 IoC 容器，一个 IoC 容器是一个工具，帮助你编写面向对象的代码，随着时间容易修改和扩展。然而，一个 IoC 容器可能被误用。正确使用一个 IoC，你必须遵守一些基本面向对象编程原则，比如[SOLID 原则](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))。

这个 wiki 页面将会专注于依赖倒置原则（SOLID 原则中的一个）和组合复用原则。

### 组合复用原则

> 优先选择“对象组合”，而不是“类继承”

使用继承没问题，但是我们应该尽可能使用组合。多于一层的继承可能是一种[代码味道](https://en.wikipedia.org/wiki/Code_smell)。

继承是一个坏的东西，因为他是模块间最强的耦合类型。来看看一个例子：
```ts
import BaseModel from "someframework";

class DerivedModel extends BaseModel {
    public constructor() {
        super();
    }
    public saveOrUpdate() {
        this.doSomething(); // accessing a base class property
        // ...
    }
}

export DerivedModel;
```

前面的代码片段的问题是`DerivedModel`和`BaseModel`类紧紧耦合在一起。在这个场景，我们使用`extends`关键字。这特别糟糕，因为没有办法打破耦合，因为类继承。

下面的例子类似，但是使用“对象组合”，而不是“类继承”：
```ts
@injectable()
class DerivedModel {
    public baseModel: BaseModel;
    public constructor(@inject("BaseModel") baseModel: BaseModel) {
        this.baseModel = baseModel;
    }
    public saveOrUpdate() {
        this.baseModel.doSomething();
        // ...
    }
}

export DerivedModel;
```

### 依赖倒置原则

> 依赖于抽象，不要依赖于具体

依赖注入不再通过类，而是通过构造器或者 setter：
```ts
@injectable()
class Ninja {

    private _katana: Katana;

    public constructor(
        katana: Katana
    ) {
        this._katana = katana;
    }

    public fight() { return this._katana.hit(); };

}
```
在这个场景中，Ninja 类对 Katana 类有依赖：
```ts
Ninja --> Katana

```

注意箭头从左到右指向依赖

如果我们更新 ninja 类去依赖一个 Katana 类的抽闲（Katana 接口）：
```ts
@injectable()
class Ninja {

    private _katana: Katana;

    public constructor(
        @inject("Katana") katana: Katana
    ) {
        this._katana = katana;
    }

    public fight() { return this._katana.hit(); };

}
```

在这个场景，Nonja 类和 Katana 类都依赖 Katana 接口：
```ts
Ninja --> Katana 
Katana --> Katana
```
这也可以标示为：
```ts
Ninja --> Katana <-- Katana

```
你注意到箭头现在是如何反转的吗？