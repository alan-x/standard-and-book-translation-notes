### 对类的支持

InversifyJS 允许你的类对其他类有直接依赖。当这么做的时候，你需要使用`@injectable`装饰器，但是`@inject`装饰器不是必须的。

当你使用类的时候，`@inject`装饰器不是必须的。这个注解不是必须的是因为 typescript 编译器为我们生产了元数据。然而，如果你忘了下面的事情，这不会发生：
- 导入`reflect-metadata`
- 在`tsconfig.json`中设置`emitDecoratorMetadata`为`true`

```ts
import { Container, injectable, inject } from "inversify";

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
class Ninja implements Warrion {

    private _katana: Katana;
    private _shuriken: Shuriken;

    public constructor(katana: Katana, shuriken: Shuriken) {
        this._katana = katana;
        this._shuriken = shuriken;
    }

    public fight() { return this._katana.hit(); };
    public sneak() { return this._shuriken.throw(); };

}

var container = new Container();
container.bind<Ninja>(Ninja).to(Ninja);
container.bind<Katana>(Katana).to(Katana);
container.bind<Shuriken>(Shuriken).to(Shuriken);
```

### 具体类型的自我绑定

如果你解析的类型是一个具体类型，绑定注册可能会有一点重复和繁琐：
```ts
container.bind<Samurai>(Samurai).to(Samurai);
```
一个更好的方案是使用`toSelf`方法：
```ts
container.bind<Samurai>(Samurai).toSelf();
```

### 已知限制：类作为标识符和循环依赖

如果我们在循环依赖中使用类作为一个标识符，一个异常：
> Error: Missing required @Inject or @multiinject annotation in: argument 0 in class Dom.

将会被抛出。比如：
```ts
import "reflect-metadata";
import { Container, injectable } from "inversify";
import getDecorators from "inversify-inject-decorators";

let container = new Container();
let { lazyInject } = getDecorators(container);

@injectable()
class Dom {
    public domUi: DomUi;
    constructor (domUi: DomUi) {
        this.domUi = domUi;
    }
}

@injectable()
class DomUi {
    @lazyInject(Dom) public dom: Dom;
}

@injectable()
class Test {
    constructor(dom: Dom) {
        console.log(dom);
    }
}

container.bind<Dom>(Dom).toSelf().inSingletonScope();
container.bind<DomUi>(DomUi).toSelf().inSingletonScope();
const dom = container.resolve(Test); // Error!
```

这个错误看起来有点误导，因为当使用类作为服务标识符的时候，`@inject`不是必须的，如果我们添加一个像`@inject(DOM)`或者`@inject(DomUi)`的注解，我们依旧会得到相同的异常。这是因为在这个时候，装饰器被调用，类还没被声明，因此装饰器调用是`@inject(undefined)`。这让 InversifyJS 认为注解没有被添加。

解决方案是使用类似`Symbol.for("DOM")`作为服务标识符替代类似`DOM`的类：
```ts
import "reflect-metadata";
import { Container, injectable, inject } from "inversify";
import getDecorators from "inversify-inject-decorators";

const container = new Container();
const { lazyInject } = getDecorators(container);

const TYPE = {
    Dom: Symbol.for("Dom"),
    DomUi: Symbol.for("DomUi")
};

@injectable()
class DomUi {
    public dom: Dom;
    public name: string;
    constructor (
        @inject(TYPE.Dom) dom: Dom
    ) {
        this.dom = dom;
        this.name = "DomUi";
    }
}

@injectable()
class Dom {
    public name: string;
    @lazyInject(TYPE.DomUi) public domUi: DomUi;
    public constructor() {
        this.name = "Dom";
    }
}

@injectable()
class Test {
    public dom: Dom;
    constructor(
        @inject(TYPE.Dom) dom: Dom
    ) {
        this.dom = dom;
    }
}

container.bind<Dom>(TYPE.Dom).to(Dom).inSingletonScope();
container.bind<DomUi>(TYPE.DomUi).to(DomUi).inSingletonScope();

const test = container.resolve(Test); // Works!
```