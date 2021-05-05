### whenTargetIsDefault

当给定服务标识符有多个多个绑定，我们可以使用下面的特性去解析潜在的`AMBIGUOUS_MATCH`异常：

- [具名绑定](https://github.com/inversify/InversifyJS/blob/master/wiki/named_bindings.md)
- [标签绑定](https://github.com/inversify/InversifyJS/blob/master/wiki/tagged_bindings.md)
- [上下文绑定](https://github.com/inversify/InversifyJS/blob/master/wiki/contextual_bindings.md)
- 默认目标

在这个章节，我们将解释怎样使用默认目标。

当我们使用具名约束解决一个`AMBIGUOUS_MATCH`异常，：
```ts
container.bind<Weapon>("Weapon").to(Katana).whenTargetNamed("strong");
container.bind<Weapon>("Weapon").to(Shuriken).whenTargetNamed("weak");
```
或者标签约束：
```ts
container.bind<Weapon>("Weapon").to(Katana).whenTargetTagged("strong", true);
container.bind<Weapon>("Weapon").to(Shuriken).whenTargetTagged("strong", false);
```

这个解决方案的问题是我们需要为每一个单独的注入使用`@named("strong")/@named("weak")`或`@tagged("strong", true)/@tagged("strong", false)`

一个更好的解决方案是使用默认目标：
```ts
container.bind<Weapon>(TYPES.Weapon).to(Shuriken).whenTargetNamed(TAG.throwable);
container.bind<Weapon>(TYPES.Weapon).to(Katana).whenTargetIsDefault();
```

我们可以使用`whenTargetIsDefault`去指示哪一个绑定应该作为默认去解决一个`AMBIGUOUS_MATCH`异常，当没有`@named`或者`@tagged`注解可用的时候。
```ts
let TYPES = {
    Weapon: "Weapon"
};

let TAG = {
    throwable: "throwable"
};

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

@injectable()
class Shuriken implements Weapon {
    public name: string;
    public constructor() {
        this.name = "Shuriken";
    }
}

let container = new Container();
container.bind<Weapon>(TYPES.Weapon).to(Shuriken).whenTargetNamed(TAG.throwable);
container.bind<Weapon>(TYPES.Weapon).to(Katana).whenTargetIsDefault();

let defaultWeapon = container.get<Weapon>(TYPES.Weapon);
let throwableWeapon = container.getNamed<Weapon>(TYPES.Weapon, TAG.throwable);

expect(defaultWeapon.name).eql("Katana");
expect(throwableWeapon.name).eql("Shuriken");
```