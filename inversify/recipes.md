### 菜谱

这个页面包含一些代码片段，展示具体的使用场景，也叫做“菜谱”。

### 注入依赖到一个函数

你需要通过声明你的绑定开始，就像在任何其他场景：
```ts
let TYPES: {
    something: "something",
    somethingElse: "somethingElse"
};

export { TYPES };
```
```ts
let inversify = require("inversify");
import { TYPES } from "./constants/types";

// declare your container
let container = new inversify.Container();
container.bind(TYPES.something).toConstantValue(1);
container.bind(TYPES.somethingElse).toConstantValue(2);

export { container };
```

继续声明下面的帮助函数：
```ts
import { container } from "./inversify.config"

function bindDependencies(func, dependencies) {
    let injections = dependencies.map((dependency) => {
        return container.get(dependency);
    });
    return func.bind(func, ...injections);
}

export { bindDependencies };
```

声明你的函数和绑定它的依赖到它的参数，使用`bindDependencies`助手：
```ts
import { bindDependencies } from "./utils/bindDependencies";
import { TYPES } from "./constants/types";

function testFunc(something, somethingElse) {
  console.log(`Injected! ${something}`);
  console.log(`Injected! ${somethingElse}`);
}

testFunc = bindDependencies(testFunc, [TYPES.something, TYPES.somethingElse]);

export { testFunc };
```

使用你的函数：
```ts
import { testFunc } from "./x/test_func";

testFunc();

// > Injected! 1
// > Injected! 2
```

### 在单元测试中覆盖绑定

有时候你想要在你的单元测试使用你的绑定声明，但是你需要去克服一些东西。我们推荐你在你的应用内去声明你的绑定作为容器模块。
```ts
let warriors = new ContainerModule((bind: Bind) => {
    bind<Ninja>("Ninja").to(Ninja);
});

let weapons = new ContainerModule((bind: Bind) => {
    bind<Katana>("Katana").to(Katana);
    bind<Shuriken>("Shuriken").to(Shuriken);
});

export { warriors, weapons };
```

然后你就可以创建一个新的容器，使用你的应用的绑定：
```ts
import { warriors, weapons} from './shared/container_modules';
import { Container } from "inversify";

describe("something", () => {

  let container: Container;

  beforeEach(() => {
      container = new Container();
      container.load(warriors, weapons);
  });

  afterEach(() => {
      container = null;
  });

  it("Should...", () => {
      container.unbind(MyService);
      container.bind(MyService).to(MyServiceMock);
      // do something
  });

});
```
正如你看到的，你可以在每一个测试用力去覆盖指定绑定。

### 当处理循环依赖的时候，使用 request 范围和激活处理器去避免工厂

如果我们有一个场景，使用循环依赖，例子如下：

- `Warrior`有一个属性名为`weapon`，是一个`Weapon`实例
- `Weapon`有一个属性名为`owner`，是一个`Weapon`实例

我们可以使用工厂处理这个问题：
```ts
import { inject, injectable, Container, interfaces } from "inversify";
import "reflect-metadata";

type FactoryOfWeapon = (parent: IWeaponHolder) => IWeapon;

const TYPE = {
    OrphanWeapon: Symbol.for("OrphanWeapon"),
    FactoryOfWeapon: Symbol.for("FactoryOfWeapon"),
    WeaponName: Symbol.for("WeaponName"),
    WeaponHolder: Symbol.for("WeaponHolder")
};

interface IWeapon {
    parent: IWeaponHolder;
    use(): string;
    owner(): string;
}

interface IWeaponHolder {
    name: string;
    weapon: IWeapon;
    fight(): string;
}

@injectable()
class Weapon implements IWeapon {
    private readonly _name: string;
    public parent: IWeaponHolder;

    public constructor(
        // We can inject stuff into Weapon
        @inject(TYPE.WeaponName) name: string
    ) {
        this._name = name;
    }

    public use() {
        return this._name;
    }

    public owner() {
        return `Owned by ${this.parent.name}!`;
    }

}

@injectable()
class Character implements IWeaponHolder {
    public weapon: IWeapon;
    public name: string;
    public constructor(
        @inject(TYPE.FactoryOfWeapon) factoryOfWeapon: FactoryOfWeapon
    ) {
        this.name = "Ninja";
        this.weapon = factoryOfWeapon(this);
    }
    public fight() {
        return `Using ${this.weapon.use()}!`;
    }
}

const container = new Container();

// We inject a string just to demostrate that we can inject stuff into Weapon
container.bind<string>(TYPE.WeaponName).toConstantValue("Katana");

// We declare a binding for Weapon so we can use it within the factory
container.bind<IWeapon>(TYPE.OrphanWeapon).to(Weapon);

container.bind<FactoryOfWeapon>(TYPE.FactoryOfWeapon).toFactory<IWeapon>(
    (ctx: interfaces.Context) => {
        return (parent: IWeaponHolder) => {
            const orphanWeapon = ctx.container.get<IWeapon>(TYPE.OrphanWeapon);
            orphanWeapon.parent = parent;
            return orphanWeapon;
        };
    });

container.bind<IWeaponHolder>(TYPE.WeaponHolder).to(Character);

const character = container.get<IWeaponHolder>(TYPE.WeaponHolder);
console.log(character.fight());
console.log(character.weapon.owner());
```

但是如果因为一些原因，我们真的想要避免工厂，我们可以使用 request 范围和激活处理器去避免工厂：
```ts
import { inject, injectable, Container, interfaces } from "inversify";
import "reflect-metadata";

type FactoryOfWeapon = (parent: IWeaponHolder) => IWeapon;

const TYPE = {
    WeaponName: Symbol.for("WeaponName"),
    WeaponHolder: Symbol.for("WeaponHolder"),
    Weapon: Symbol.for("Weapon")
};

interface IWeapon {
    parent: IWeaponHolder;
    use(): string;
    owner(): string;
}

interface IWeaponHolder {
    name: string;
    weapon: IWeapon;
    fight(): string;
}

@injectable()
class Weapon implements IWeapon {
    private readonly _name: string;
    public parent: IWeaponHolder;

    public constructor(
        // We can inject stuff into Weapon
        @inject(TYPE.WeaponName) name: string
    ) {
        this._name = name;
    }

    public use() {
        return this._name;
    }

    public owner() {
        return `Owned by ${this.parent.name}!`;
    }

}

@injectable()
class Character implements IWeaponHolder {
    public weapon: IWeapon;
    public name: string;
    public constructor(
        @inject(TYPE.Weapon) weapon: IWeapon
    ) {
        this.name = "Ninja";
        this.weapon = weapon; // No need for factory :)
    }
    public fight() {
        return `Using ${this.weapon.use()}!`;
    }
}

const container = new Container();

// We inject a string just to demostrate that we can inject stuff into Weapon
container.bind<string>(TYPE.WeaponName).toConstantValue("Katana");

// The inRequestScope is important here
container.bind<IWeapon>(TYPE.Weapon).to(Weapon).inRequestScope();

// We can use onActivation adn search for Weapon in the inRequestScope
container.bind<IWeaponHolder>(TYPE.WeaponHolder)
    .to(Character)
    .onActivation((ctx: interfaces.Context, weaponHolderInstance: IWeaponHolder) => {
        const scope = ctx.plan.rootRequest.requestScope;
        if (scope) {
            // We search in the entire inRequestScope, this
            // takes O(n) execution time so It is slower than the factory
            const weaponInstance = Array.from(scope.values())
                                        .find(v => v instanceof Weapon);
            weaponInstance.parent = weaponHolderInstance;
        }
        return weaponHolderInstance;
    });

const character = container.get<IWeaponHolder>(TYPE.WeaponHolder);

console.log(character.fight()); // Using Katana!
console.log(character.weapon.owner()); // Owned by Ninja!.
```

请注意这个变通方式非常易碎，在很多上下文无法工作。`onActivation`可能需要额外的逻辑去处理复杂的场景。