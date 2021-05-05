### 具名绑定

我们可以使用具名绑定去修复修复`AMBIGUOUS_MATCH`错误，当两个或者多个具体被绑定到一个抽象。注意构造器参数`Ninja`类是如何使用`@named`装饰器注解：
```ts
interface Weapon {}

@injectable()
class Katana implements Weapon {}

@injectable()
class Shuriken implements Weapon {}

interface Ninja {
    katana: Weapon;
    shuriken: Weapon;
}

@injectable()
class Ninja implements Ninja {
    public katana: Weapon;
    public shuriken: Weapon;
    public constructor(
        @inject("Weapon") @named("strong") katana: Weapon,
        @inject("Weapon") @named("weak") shuriken: Weapon
    ) {
        this.katana = katana;
        this.shuriken = shuriken;
    }
}
```

我们绑定`Katana`和`Shuriken`到`Weapon`，但是一个`whenTargetNamed`约束被添加，避免`AMBIGUOUS_MATCH`错误：
```ts
container.bind<Ninja>("Ninja").to(Ninja);
container.bind<Weapon>("Weapon").to(Katana).whenTargetNamed("strong");
container.bind<Weapon>("Weapon").to(Shuriken).whenTargetNamed("weak");
```