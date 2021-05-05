### 可选依赖

当我们声明一个可选依赖，使用`@optional()`装饰器：
```ts
@injectable()
class Katana {
    public name: string;
    public constructor() {
        this.name = "Katana";
    }
}

@injectable()
class Shuriken {
    public name: string;
    public constructor() {
        this.name = "Shuriken";
    }
}

@injectable()
class Ninja {
    public name: string;
    public katana: Katana;
    public shuriken: Shuriken;
    public constructor(
        @inject("Katana") katana: Katana,
        @inject("Shuriken") @optional() shuriken: Shuriken // Optional!
    ) {
        this.name = "Ninja";
        this.katana = katana;
        this.shuriken = shuriken;
    }
}

let container = new Container();

container.bind<Katana>("Katana").to(Katana);
container.bind<Ninja>("Ninja").to(Ninja);

let ninja = container.get<Ninja>("Ninja");
expect(ninja.name).to.eql("Ninja");
expect(ninja.katana.name).to.eql("Katana");
expect(ninja.shuriken).to.eql(undefined);

container.bind<Shuriken>("Shuriken").to(Shuriken);

ninja = container.get<Ninja>("Ninja");
expect(ninja.name).to.eql("Ninja");
expect(ninja.katana.name).to.eql("Katana");
expect(ninja.shuriken.name).to.eql("Shuriken");
```

在这个例子，我们可以看到我们第一次是如何解析`Ninja`，它的属性`shuriken`是 undefined，因为没有声明为`Shuriken`的绑定并且属性使用`@optional()`装饰器声明

在为`Shuriken`声明绑定之后：
```ts
container.bind<Shuriken>("Shuriken").to(Shuriken);

```

所有解析的`Ninja`实例将会包含`Shuriken`的实例


#### 默认值

如果一个依赖使用`@optional()`装饰器，我们将可以使用声明一个默认值，就像你可以在任何其他 TypeScript 应用那样做：
```ts
@injectable()
class Ninja {
    public name: string;
    public katana: Katana;
    public shuriken: Shuriken;
    public constructor(
        @inject("Katana") katana: Katana,
        @inject("Shuriken") @optional() shuriken: Shuriken = { name: "DefaultShuriken" } // Default value!
    ) {
        this.name = "Ninja";
        this.katana = katana;
        this.shuriken = shuriken;
    }
}

```