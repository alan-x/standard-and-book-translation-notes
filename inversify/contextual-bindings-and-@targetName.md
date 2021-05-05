### 上下文绑定和 @targetName

`@targetName`装饰器用于从上下文约束访问构造器参数的名字，甚至当代码被压缩的时候。`constructor(katana, shuriken) { ...`压缩后变成`constructor(a, b) { ...`，但是感谢`@targetname`，我们依旧可以在运行时引用设计态名称`katana`和`shuriken`。
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
        @inject("Weapon") @targetName("katana") katana: Weapon,
        @inject("Weapon") @targetName("shuriken") shuriken: Weapon
    ) {
        this.katana = katana;
        this.shuriken = shuriken;
    }
}
```

我们绑定`Katana`和`Shuriken`到`Weapon`，但是一个自定义`when`约束被添加去避免`AMBIGUOUS_MATCH`错误：
```ts
container.bind<Ninja>(ninjaId).to(Ninja);

container.bind<Weapon>("Weapon").to(Katana).when((request: interfaces.Request) => {
    return request.target.name.equals("katana");
});

container.bind<Weapon>("Weapon").to(Shuriken).when((request: interfaces.Request) => {
    return request.target.name.equals("shuriken");
});
```

目标域是想`IQueryableString`接口去帮助你去创建你自定义约束：
```ts
interface QueryableString {
	 startsWith(searchString: string): boolean;
	 endsWith(searchString: string): boolean;
	 contains(searchString: string): boolean;
	 equals(compareString: string): boolean;
	 value(): string;
}

```

我们已经包含了一些助手去创建自定义约束：
```ts
import { Container, traverseAncerstors, taggedConstraint, namedConstraint, typeConstraint } from "inversify";

let whenParentNamedCanThrowConstraint = (request: interfaces.Request) => {
    return namedConstraint("canThrow")(request.parentRequest);
};

let whenAnyAncestorIsConstraint = (request: interfaces.Request) => {
    return traverseAncerstors(request, typeConstraint(Ninja));
};

let whenAnyAncestorTaggedConstraint = (request: interfaces.Request) => {
    return traverseAncerstors(request, taggedConstraint("canThrow")(true));
};
```

InversifyJS 绑定的顺畅语法宝航一些已经是想的常见上下文约束：

```ts
interface BindingWhenSyntax<T> {
    when(constraint: (request: interfaces.Request) => boolean): interfaces.BindingOnSyntax<T>;
    whenTargetNamed(name: string): interfaces.BindingOnSyntax<T>;
    whenTargetTagged(tag: string, value: any): interfaces.BindingOnSyntax<T>;
    whenInjectedInto(parent: (Function|string)): interfaces.BindingOnSyntax<T>;
    whenParentNamed(name: string): interfaces.BindingOnSyntax<T>;
    whenParentTagged(tag: string, value: any): interfaces.BindingOnSyntax<T>;
    whenAnyAncestorIs(ancestor: (Function|string)): interfaces.BindingOnSyntax<T>;
    whenNoAncestorIs(ancestor: (Function|string)): interfaces.BindingOnSyntax<T>;
    whenAnyAncestorNamed(name: string): interfaces.BindingOnSyntax<T>;
    whenAnyAncestorTagged(tag: string, value: any): interfaces.BindingOnSyntax<T>;
    whenNoAncestorNamed(name: string): interfaces.BindingOnSyntax<T>;
    whenNoAncestorTagged(tag: string, value: any): interfaces.BindingOnSyntax<T>;
    whenAnyAncestorMatches(constraint: (request: interfaces.Request) => boolean): interfaces.BindingOnSyntax<T>;
    whenNoAncestorMatches(constraint: (request: interfaces.Request) => boolean): interfaces.BindingOnSyntax<T>;
}
```