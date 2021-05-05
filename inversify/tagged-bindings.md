# 标签绑定

我们可以使用标签绑定去修复`AMBIGUOUS_MATCH`错误，当两个或者多个具体被绑定到一个抽象。注意构造器参数`Ninja`类是怎样使用`@tagged`装饰器注解的：
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
        @inject("Weapon") @tagged("canThrow", false) katana: Weapon,
        @inject("Weapon") @tagged("canThrow", true) shuriken: Weapon
    ) {
        this.katana = katana;
        this.shuriken = shuriken;
    }
}
```

我们可以绑定`Katana`和`Shuriken`到`Weapon`，但是`whenTargetTagged`约束被天骄，避免`AMBIGUOUS_MATCH`错误：
```ts
container.bind<Ninja>(ninjaId).to(Ninja);
container.bind<Weapon>(weaponId).to(Katana).whenTargetTagged("canThrow", false);
container.bind<Weapon>(weaponId).to(Shuriken).whenTargetTagged("canThrow", true);
```