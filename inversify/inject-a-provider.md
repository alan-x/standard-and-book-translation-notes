### 注入一个 Provider（异步 Factory）

绑定一个抽象到一个 Provider。一个提供者是一个异步工厂，当处理异步 IO 操作的时候，这很有用。
```ts
type KatanaProvider = () => Promise<Katana>;

@injectable()
class Ninja implements Ninja {

    public katana: Katana;
    public shuriken: Shuriken;
    public katanaProvider: KatanaProvider;

    public constructor(
	    @inject("KatanaProvider") katanaProvider: KatanaProvider, 
	    @inject("Shuriken") shuriken: Shuriken
    ) {
        this.katanaProvider = katanaProvider;
        this.katana= null;
        this.shuriken = shuriken;
    }

    public fight() { return this.katana.hit(); };
    public sneak() { return this.shuriken.throw(); };

}
```

```ts
container.bind<KatanaProvider>("KatanaProvider").toProvider<Katana>((context) => {
    return () => {
        return new Promise<Katana>((resolve) => {
            let katana = context.container.get<Katana>("Katana");
            resolve(katana);
        });
    };
});

var ninja = container.get<Ninja>("Ninja");

ninja.katanaProvider()
     .then((katana) => { ninja.katana = katana; })
     .catch((e) => { console.log(e); });
```

#### Provider 自定义参数

`toProvider`绑定期待一个`ProviderCreator`作为它唯一的参数：
```ts
interface ProviderCreator<T> extends Function {
    (context: Context): Provider<T>;
}
```

provider 的签名看起来像这样：
```ts
interface Provider<T> extends Function {
    (...args: any[]): (((...args: any[]) => Promise<T>) | Promise<T>);
}
```

这些类型签名允许传递自定义参数到一个 provider：
```ts
let container = new Container();

interface Sword {
    material: string;
    damage: number;
}

@injectable()
class Katana implements Sword {
    public material: string;
    public damage: number;
}

type SwordProvider = (material: string, damage: number) => Promise<Sword>;

container.bind<Sword>("Sword").to(Katana);

container.bind<SwordProvider>("SwordProvider").toProvider<Sword>((context) => {
    return (material: string, damage: number) => { // Custom args!
        return new Promise<Sword>((resolve) => {
            setTimeout(() => {
                let katana = context.container.get<Sword>("Sword");
                katana.material = material;
                katana.damage = damage;
                resolve(katana);
            }, 10);
        });
    };
});

let katanaProvider = container.get<SwordProvider>("SwordProvider");

katanaProvider("gold", 100).then((powerfulGoldKatana) => { // Apply all custom args
    expect(powerfulGoldKatana.material).to.eql("gold");
    expect(powerfulGoldKatana.damage).to.eql(100);
});

katanaProvider("gold", 10).then((notSoPowerfulGoldKatana) => {
    expect(notSoPowerfulGoldKatana.material).to.eql("gold");
    expect(notSoPowerfulGoldKatana.damage).to.eql(10);
});
```

### Provider 部分应用

我们也可以使用部分应用传递参数：
```ts
let container = new Container();

interface Sword {
    material: string;
    damage: number;
}

@injectable()
class Katana implements Sword {
    public material: string;
    public damage: number;
}

type SwordProvider = (material: string) => (damage: number) => Promise<Sword>;

container.bind<Sword>("Sword").to(Katana);

container.bind<SwordProvider>("SwordProvider").toProvider<Sword>((context) => {
    return (material: string) => {  // Custom arg 1!
        return (damage: number) => { // Custom arg 2!
            return new Promise<Sword>((resolve) => {
                setTimeout(() => {
                    let katana = context.container.get<Sword>("Sword");
                    katana.material = material;
                    katana.damage = damage;
                    resolve(katana);
                }, 10);
            });
        };
    };
});

let katanaProvider = container.get<SwordProvider>("SwordProvider");
let goldKatanaProvider = katanaProvider("gold");  // Apply the first custom arg!

goldKatanaProvider(100).then((powerfulGoldKatana) => { // Apply the second custom args!
    expect(powerfulGoldKatana.material).to.eql("gold");
    expect(powerfulGoldKatana.damage).to.eql(100);
});

goldKatanaProvider(10).then((notSoPowerfulGoldKatana) => {
    expect(notSoPowerfulGoldKatana.material).to.eql("gold");
    expect(notSoPowerfulGoldKatana.damage).to.eql(10);
});
```

### Provider 作为单例

一个 Provider 总是注入为单例，但是你可以控制，如果 Provider 的返回值使用单例或者临时范围：
```ts
let container = new Container();

interface Warrior {
    level: number;
}

@injectable()
class Ninja implements Warrior {
    public level: number;
    public constructor() {
        this.level = 0;
    }
}

type WarriorProvider = (level: number) => Promise<Warrior>;

container.bind<Warrior>("Warrior").to(Ninja).inSingletonScope(); // Value is singleton!

container.bind<WarriorProvider>("WarriorProvider").toProvider<Warrior>((context) => {
    return (increaseLevel: number) => {
        return new Promise<Warrior>((resolve) => {
            setTimeout(() => {
                let warrior = context.container.get<Warrior>("Warrior"); // Get singleton!
                warrior.level += increaseLevel;
                resolve(warrior);
            }, 100);
        });
    };
});

let warriorProvider = container.get<WarriorProvider>("WarriorProvider");

warriorProvider(10).then((warrior) => {
    expect(warrior.level).to.eql(10);
});

warriorProvider(10).then((warrior2) => {
    expect(warrior.level).to.eql(20);
});
```

### 默认 Provider

下列的函数可以用作一个助手去提供一个默认值，当 provider 被注入的时候：
```ts
function valueOrDefault<T>(provider: () => Promise<T>, defaultValue: T) {
    return new Promise<T>((resolve, reject) => {
        provider().then((value) => {
            resolve(value);
        }).catch(() => {
            resolve(defaultValue);
        });
    });
}
```

下面例子展示怎样应用`valueOrDefault`助手。
```ts
@injectable()
class Ninja {
    public level: number;
    public rank: string;
    public constructor() {
        this.level = 0;
        this.rank = "Ninja";
    }
    public train(): Promise<number> {
        return new Promise<number>((resolve) => {
            setTimeout(() => {
                this.level += 10;
                resolve(this.level);
            }, 100);
        });
    }
}

@injectable()
class NinjaMaster {
    public rank: string;
    public constructor() {
        this.rank = "NinjaMaster";
    }
}

type NinjaMasterProvider = () => Promise<NinjaMaster>;

let container = new Container();

container.bind<Ninja>("Ninja").to(Ninja).inSingletonScope();
container.bind<NinjaMasterProvider>("NinjaMasterProvider").toProvider((context) => {
    return () => {
        return new Promise<NinjaMaster>((resolve, reject) => {
            let ninja = context.container.get<Ninja>("Ninja");
            ninja.train().then((level) => {
                if (level >= 20) {
                    resolve(new NinjaMaster());
                } else {
                    reject("Not enough training");
                }
            });
        });
    };
});

let ninjaMasterProvider = container.get<NinjaMasterProvider>("NinjaMasterProvider");

valueOrDefault(ninjaMasterProvider, { rank: "DefaultNinjaMaster" }).then((ninjaMaster) => {
    // Using default here because the provider was rejected (the ninja has a level below 20)
    expect(ninjaMaster.rank).to.eql("DefaultNinjaMaster");
});

valueOrDefault(ninjaMasterProvider, { rank: "DefaultNinjaMaster" }).then((ninjaMaster) => {
    // A NinjaMaster was provided because the the ninja has a level above 20
    expect(ninjaMaster.rank).to.eql("NinjaMaster");
    done();
});
```