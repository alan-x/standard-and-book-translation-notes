### 多注入

我们可以使用多注入，当多个关注点被包裹到一个抽象。注意一个数据的`[Weapon]`是怎样注入到一个`Ninja`类，通过构造器，感谢`@multiInject`装饰器的使用：
```ts
interface Weapon {
    name: string;
}

@injectable()
class Katana implements Weapon {
    public name = "Katana";
}

@injectable()
class Shuriken implements Weapon {
    public name = "Shuriken";
}

interface Ninja {
    katana: Weapon;
    shuriken: Weapon;
}

@injectable()
class Ninja implements Ninja {
    public katana: Weapon;
    public shuriken: Weapon;
    public constructor(
	    @multiInject("Weapon") weapons: Weapon[]
    ) {
        this.katana = weapons[0];
        this.shuriken = weapons[1];
    }
}
```

我们绑定`Katana`和`Shuriken`到`Weapon`：
```ts
container.bind<Ninja>("Ninja").to(Ninja);
container.bind<Weapon>("Weapon").to(Katana);
container.bind<Weapon>("Weapon").to(Shuriken);
```

### 关于`...`扩展操作符

在 InversifyJS 的早期发行中，扩展操作符总是失败并且不抛出任何错误。这是不可接受，的，我们实现了一个修复，允许你去使用扩展操作符注入一个数组。然而，不推荐使用它，因为它没啥用。

你可以使用`@multiInject`和`...`如下注入：
```ts
@injectable()
class Foo {
    public bar: Bar[];
    constructor(@multiInject(BAR) ...args: Bar[][]) {
        // args will always contain one unique item the value of that item is a Bar[] 
        this.bar = args[0];
    }
}
```

主要问题是这需要`args`的类型是`Bar[][]`，因为多注入将会包裹注入，使用一个数组和扩展操作符将左一样的事情。最后，注入通过一个数据被包裹两次。

我们尝试解决这个问题，但是唯一的方式是使用一个`@spread()`装饰器生成额外的元数据。
```ts
@injectable()
class Foo {
    public bar: Bar[];
    constructor(@multiInject(BAR) @spread() ...args: Bar[]) {
        this.bar = args[0];
    }
}
```

我们放弃了这个想法，因为它最好使用装饰器，当没有其他方式去达到这个目的。在这个场景，有多个更简单的方式去达到这个期待的结果。我们只需要使用`@multiInject`并避免使用`...`：
```ts
@injectable()
class Foo {
    public bar: Bar[];
    constructor(@multiInject(BAR) args: Bar[]) {
        this.bar = args;
    }
}
```