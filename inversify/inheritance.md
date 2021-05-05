### 继承

我们尝试提供开发者有用的错误反馈，类似：

> Error: Missing required @injectable annotation in: SamuraiMaster

这在大部分场景很有用，但是在使用继承的时候会有一些问题。

比如，下面的代码脚本抛出一个误导错误：

> 派生类的构造器参数的数量必须 >= 基类的构造器参数。

```ts
@injectable()
class Warrior {
    public rank: string;
    public constructor(rank: string) { // args count  = 1
        this.rank = rank;
    }
}

@injectable()
class SamuraiMaster extends Warrior {
    public constructor() { // args count = 0
        super("master");
    }
}
```

为了克服这个问题，InversifyJS 根据两条规则限制继承的使用：

> 一个派生类必须明确声明它的构造器。

> 一个派生类构造器参数的数量必须 >= 基类的构造器参数。

如果你不遵守这个规则，将会抛出一个异常：

> Error: The number of constructor arguments in the derived class SamuraiMaster must be >= than the number of constructor arguments of its base class.

用户有一些方式去克服这个限制：

### 变通方法 A) 使用 @unmanaged 装饰器

`@unmanaged()`装饰器允许用户去标志一个参数将会手动注入到基类。我们使用单词“unmanaged”是因为 InversifyJS 不控制用户提供的值，它不管理他们的依赖。

下面的代码注入展示了怎样应用这个装饰器：

```ts
import { Container, injectable, unmanaged } from "../src/inversify";

const BaseId = "Base";

@injectable()
class Base {
    public prop: string;
    public constructor(@unmanaged() arg: string) {
        this.prop = arg;
    }
}

@injectable()
class Derived extends Base {
    public constructor() {
        super("unmanaged-injected-value");
    }
}

container.bind<Base>(BaseId).to(Derived);
let derived = container.get<Base>(BaseId);

derived instanceof Derived2; // true
derived.prop; // "unmanaged-injected-value"
```

### 变通方法 B）属性设置器

你可以使用`public`，`protected`，或者`private`访问装饰器和一个属性设置器避免注入基本类：
```ts
@injectable()
class Warrior {
    protected rank: string;
    public constructor() { // args count = 0
        this.rank = null;
    }
}

@injectable()
class SamuraiMaster extends Warrior {
    public constructor() { // args count = 0
        super();
        this.rank = "master";
    }
}
```

### 变通方法 C) 属性注入

我们也可以使用属性注入去避免注入基类：
```ts
@injectable()
class Warrior {
    protected rank: string;
    public constructor() {} // args count = 0
}

let TYPES = { Rank: "Rank" };

@injectable()
class SamuraiMaster extends Warrior {
    @injectNamed(TYPES.Rank, "master")
    @named("master")
    protected rank: string;

    public constructor() { // args count = 0
        super();
    }
}

container
    .bind<string>(TYPES.Rank)
    .toConstantValue("master")
    .whenTargetNamed("master");
```

### 变通方法 D) 注入到派生类

如果我们想要避免注入基类，我们可以注入派生类，然后就注入到基类使用它的构造器（super）。
```ts
@injectable()
class Warrior {
    protected rank: string;
    public constructor(rank: string) { // args count = 1
        this.rank = rank;
    }
}

let TYPES = { Rank: "Rank" };

@injectable()
class SamuraiMaster extends Warrior {
    public constructor(
        @inject(TYPES.Rank) @named("master") rank: string // args count = 1
    ) {
        super(rank);
    }
}

container
    .bind<string>(TYPES.Rank)
    .toConstantValue("master")
    .whenTargetNamed("master");
```
下面也能工作：
```ts
@injectable()
class Warrior {
    protected rank: string;
    public constructor(rank: string) { // args count = 1
        this.rank = rank;
    }
}

interface Weapon {
    name: string;
}

@injectable()
class Katana implements Weapon {
    public name: string;
    public constructor() {
        this.name = "Katana";
    }
}

let TYPES = {
    Rank: "Rank",
    Weapon: "Weapon",
};

@injectable()
class SamuraiMaster extends Warrior {
    public weapon: Weapon;
    public constructor(
        @inject(TYPES.Rank) @named("master") rank: string, // args count = 2
        @inject(TYPES.Weapon) weapon: Weapon
    ) {
        super(rank);
        this.weapon = weapon;
    }
}

container.bind<Weapon>(TYPES.Weapon).to(Katana);

container
    .bind<string>(TYPES.Rank)
    .toConstantValue("master")
    .whenTargetNamed("master");
```

### 变通方法 E) 跳过基类`@injectable`检测

设置`skipBaseClassChecks`选项为`true`金庸所有基类的检测。这意味着他将完全取决于用户确保`super()`构造器使用正确的参数在正确的时间调用。
```ts
// Not injectable
class UnmanagedBase {
    public constructor(public unmanagedDependency: string) {}
}

@injectable()
class InjectableDerived extends UnmanagedBase {
    public constructor() // Any arguments defined here will be injected like normal
    {
        super("Don't forget me...");
    }
}

const container = new Container({ skipBaseClassChecks: true });
container.bind(InjectableDerived).toSelf();
```

这可以用工作，你将能够正常使用`InjectableDerived`类，包含从其他地方注入到构造器的依赖。有一个警告是你必须确保你的`UnmanagedBase`接受正确的参数。

### 当我的基类通过第三方模块提供的时候我可以做啥？

在某些场景，你可能会在第三方模块提供的类得到一个缺少注解的错误，比如：
> Error: Missing required @injectable annotation in: SamuraiMaster

你可以克服这个问题，使用`decorate`函数：
```ts
import { decorate, injectable } from "inversify";
import SomeClass from "some-module";

decorate(injectable(), SomeClass);
return SomeClass;
```

查阅 wiki [JS 例子](https://github.com/inversify/InversifyJS/blob/master/wiki/basic_js_example.md)页面了解更多信息。