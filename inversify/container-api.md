### Container API

InversifyJS 容器就是依赖第一次或者之后通过绑定配置，重新配置或者移除的地方。容器可以直接使用，也可以利用容器模块。你可以使用`resolved`和`get`方法查询配置，病、并解析配置的依赖。你可以响应容器的激活处理器或者和容器的失活处理器。你可以创建容器层级，祖先容器可以供应子孙的依赖。为了测试，状态可以保存为一个快照到一个栈，之后再恢复。为了更好的控制，你可以应用中间件去拦截解析请求和解析依赖。你甚至可以提供你自己的注解解决方案。

### 容器选项

容器选项可以传递给 Container 构造器，如果你没有提供，或者缺省一些选项，将会提供默认的选项。选项可以在构造之后被改变，并且将会被从 Container 创建的子容器共享，如果你没有为他们提供选项的话。

#### defaultScope

当绑定 to/toSelf/toDynamicValue/toService，默认的范围是`transient`。其他绑定类型是`singleton`。

你可以使用容器选项在应用级别改变绑定的默认`transient`范围：
```ts
let container = new Container({ defaultScope: "Singleton" });
```

对于所有类型绑定，你可以在声明的时候改变范围：
```ts
container.bind<Warrior>(TYPES.Warrior).to(Ninja).inSingletonScope();
container.bind<Warrior>(TYPES.Warrior).to(Ninja).inTransientScope();
container.bind<Warrior>(TYPES.Warrior).to(Ninja).inRequestScope();
```

#### autoBindInjectable

你可以使用这个为`@injectable()`注解的类激活自定绑定。
```ts
let container = new Container({ autoBindInjectable: true });
container.isBound(Ninja);          // returns false
container.get(Ninja);              // returns a Ninja
container.isBound(Ninja);          // returns true
```

手动定义的绑定将会优先：
```ts
let container = new Container({ autoBindInjectable: true });
container.bind(Ninja).to(Samurai);
container.get(Ninja);              // returns a Samurai
```

#### skipBaseClassChecks

你可以使用这个去跳过`@injectable`属性的基类检测，这特别有用，如果你的任意一个`@injectable`类继承任何你不想控制（第三方类）的类。默认，这个值是`false`。
```ts
let container = new Container({ skipBaseClassChecks: true });
```

#### Container.merge(a: interfaces.Container, b: interfaces.Container, ...containers: interfaces.Container[]): interfaces.Container

创建一个新容器，包含两个或者更多容器的绑定（克隆的绑定）：
```ts
@injectable()
class Ninja {
    public name = "Ninja";
}

@injectable()
class Shuriken {
    public name = "Shuriken";
}

let CHINA_EXPANSION_TYPES = {
    Ninja: "Ninja",
    Shuriken: "Shuriken"
};

let chinaExpansionContainer = new Container();
chinaExpansionContainer.bind<Ninja>(CHINA_EXPANSION_TYPES.Ninja).to(Ninja);
chinaExpansionContainer.bind<Shuriken>(CHINA_EXPANSION_TYPES.Shuriken).to(Shuriken);

@injectable()
class Samurai {
    public name = "Samurai";
}

@injectable()
class Katana {
    public name = "Katana";
}

let JAPAN_EXPANSION_TYPES = {
    Katana: "Katana",
    Samurai: "Samurai"
};

let japanExpansionContainer = new Container();
japanExpansionContainer.bind<Samurai>(JAPAN_EXPANSION_TYPES.Samurai).to(Samurai);
japanExpansionContainer.bind<Katana>(JAPAN_EXPANSION_TYPES.Katana).to(Katana);

let gameContainer = Container.merge(chinaExpansionContainer, japanExpansionContainer);
expect(gameContainer.get<Ninja>(CHINA_EXPANSION_TYPES.Ninja).name).to.eql("Ninja");
expect(gameContainer.get<Shuriken>(CHINA_EXPANSION_TYPES.Shuriken).name).to.eql("Shuriken");
expect(gameContainer.get<Samurai>(JAPAN_EXPANSION_TYPES.Samurai).name).to.eql("Samurai");
expect(gameContainer.get<Katana>(JAPAN_EXPANSION_TYPES.Katana).name).to.eql("Katana");
```


#### container.applyCustomMetadataReader(metadataReader: interfaces.MetadataReader): void

一个高级特性...查阅[中间件]()

#### container.applyMiddleware(...middleware: interfaces.Middleware[]): void

一个高级特性，可以用于横切关注点。查阅[中间件]()

#### container.bind<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): interfaces.BindingToSyntax<T>

#### container.createChild(containerOptions?: interfaces.ContainerOptions): Container;

创建一个[容器层级](https://github.com/inversify/InversifyJS/blob/master/wiki/middleware.md)。如果你不提供选项，子容器将会接受父亲的选项。

#### container.get<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): T

通过它的运行时标识符解析一个依赖。运行时标识符必须关联绑定，并且绑定必须同步解析，否则将会抛出一个错误：
```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana);

let katana = container.get<Weapon>("Weapon");
```

#### container.getAsync<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): Promise<T>

通过它的运行时标识符解析一个依赖。运行时标识符必须关联一个绑定，否则将会抛出一个错误：
```ts
async function buildLevel1(): Level1 {
    return new Level1();
}

let container = new Container();
container.bind("Level1").toDynamicValue(() => buildLevel1());

let level1 = await container.getAsync<Level1>("Level1"); // Returns Promise<Level1>
```

#### container.getNamed<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, named: string | number | symbol): T

通过匹配给定命名约束的运行时标识符解析一个依赖。运行时标识符必须并且只能关联一个绑定，绑定必须是同步解析，否则将会抛出一个错误。

```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana).whenTargetNamed("japanese");
container.bind<Weapon>("Weapon").to(Shuriken).whenTargetNamed("chinese");

let katana = container.getNamed<Weapon>("Weapon", "japanese");
let shuriken = container.getNamed<Weapon>("Weapon", "chinese");
```

#### container.getNamedAsync<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, named: string | number | symbol): Promise<T>

通过匹配给定命名约束的运行时标识符解析一个依赖。运行时标识符必须并且只能关联一个绑定，否则将会抛出一个错误。

```ts
let container = new Container();
container.bind<Weapon>("Weapon").toDynamicValue(async () => new Katana()).whenTargetNamed("japanese");
container.bind<Weapon>("Weapon").toDynamicValue(async () => new Weapon()).whenTargetNamed("chinese");

let katana = await container.getNamedAsync<Weapon>("Weapon", "japanese");
let shuriken = await container.getNamedAsync<Weapon>("Weapon", "chinese");
```

#### container.getTagged<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, key: string | number | symbol, value: any): T

解析一个依赖，通过匹配给定标签约束的运行时标识符。运行时标识符必须并且只能关联一个绑定，绑定必须是同步解析，否则将会抛出一个错误。

```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana).whenTargetTagged("faction", "samurai");
container.bind<Weapon>("Weapon").to(Shuriken).whenTargetTagged("faction", "ninja");

let katana = container.getTagged<Weapon>("Weapon", "faction", "samurai");
let shuriken = container.getTagged<Weapon>("Weapon", "faction", "ninja");
```

#### container.getAll<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): T[]

通过匹配给定标签约束的运行时标识符解析一个依赖。运行时标识符必须并且只能关联一个绑定，否则将会抛出一个错误。

```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana);
container.bind<Weapon>("Weapon").to(Shuriken);

let weapons = container.getAll<Weapon>("Weapon");  // returns Weapon[]
```

#### container.getAllAsync<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): Promise<T[]>

获取给定标识符所有可用绑定：
```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana);
container.bind<Weapon>("Weapon").toDynamicValue(async () => new Shuriken());

let weapons = await container.getAllAsync<Weapon>("Weapon");  // returns Promise<Weapon[]>
```

#### container.getAllNamed<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, named: string | number | symbol): T[]

通过匹配给定具名约束的运行时标识符解析所有依赖，所有绑定必须同步解析，否则将会抛出一个错误：
```ts
let container = new Container();

interface Intl {
    hello?: string;
    goodbye?: string;
}

container.bind<Intl>("Intl").toConstantValue({ hello: "bonjour" }).whenTargetNamed("fr");
container.bind<Intl>("Intl").toConstantValue({ goodbye: "au revoir" }).whenTargetNamed("fr");

container.bind<Intl>("Intl").toConstantValue({ hello: "hola" }).whenTargetNamed("es");
container.bind<Intl>("Intl").toConstantValue({ goodbye: "adios" }).whenTargetNamed("es");

let fr = container.getAllNamed<Intl>("Intl", "fr");
expect(fr.length).to.eql(2);
expect(fr[0].hello).to.eql("bonjour");
expect(fr[1].goodbye).to.eql("au revoir");

let es = container.getAllNamed<Intl>("Intl", "es");
expect(es.length).to.eql(2);
expect(es[0].hello).to.eql("hola");
expect(es[1].goodbye).to.eql("adios");
```

#### container.getAllNamedAsync<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, named: string | number | symbol): Promise<T[]>


通过匹配给定具名约束的运行时标识符解析所有依赖:
```ts
let container = new Container();

interface Intl {
    hello?: string;
    goodbye?: string;
}

container.bind<Intl>("Intl").toDynamicValue(async () => ({ hello: "bonjour" })).whenTargetNamed("fr");
container.bind<Intl>("Intl").toDynamicValue(async () => ({ goodbye: "au revoir" })).whenTargetNamed("fr");

container.bind<Intl>("Intl").toDynamicValue(async () => ({ hello: "hola" })).whenTargetNamed("es");
container.bind<Intl>("Intl").toDynamicValue(async () => ({ goodbye: "adios" })).whenTargetNamed("es");

let fr = await container.getAllNamedAsync<Intl>("Intl", "fr");
expect(fr.length).to.eql(2);
expect(fr[0].hello).to.eql("bonjour");
expect(fr[1].goodbye).to.eql("au revoir");

let es = await container.getAllNamedAsync<Intl>("Intl", "es");
expect(es.length).to.eql(2);
expect(es[0].hello).to.eql("hola");
expect(es[1].goodbye).to.eql("adios");
```

#### container.getAllTagged<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, key: string | number | symbol, value: any): T[]

通过匹配给定标签约束的运行时标识符解析所有依赖，所有绑定必须同步解析，否则将会抛出一个错误：

```ts
let container = new Container();

interface Intl {
    hello?: string;
    goodbye?: string;
}

container.bind<Intl>("Intl").toConstantValue({ hello: "bonjour" }).whenTargetTagged("lang", "fr");
container.bind<Intl>("Intl").toConstantValue({ goodbye: "au revoir" }).whenTargetTagged("lang", "fr");

container.bind<Intl>("Intl").toConstantValue({ hello: "hola" }).whenTargetTagged("lang", "es");
container.bind<Intl>("Intl").toConstantValue({ goodbye: "adios" }).whenTargetTagged("lang", "es");

let fr = container.getAllTagged<Intl>("Intl", "lang", "fr");
expect(fr.length).to.eql(2);
expect(fr[0].hello).to.eql("bonjour");
expect(fr[1].goodbye).to.eql("au revoir");

let es = container.getAllTagged<Intl>("Intl", "lang", "es");
expect(es.length).to.eql(2);
expect(es[0].hello).to.eql("hola");
expect(es[1].goodbye).to.eql("adios");
```

#### container.getAllTaggedAsync<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, key: string | number | symbol, value: any): Promise<T[]>

通过匹配给定标签约束的运行时标识符解析所有依赖：
```ts
let container = new Container();

interface Intl {
    hello?: string;
    goodbye?: string;
}

container.bind<Intl>("Intl").toDynamicValue(async () => ({ hello: "bonjour" })).whenTargetTagged("lang", "fr");
container.bind<Intl>("Intl").toDynamicValue(async () => ({ goodbye: "au revoir" })).whenTargetTagged("lang", "fr");

container.bind<Intl>("Intl").toDynamicValue(async () => ({ hello: "hola" })).whenTargetTagged("lang", "es");
container.bind<Intl>("Intl").toDynamicValue(async () => ({ goodbye: "adios" })).whenTargetTagged("lang", "es");

let fr = await container.getAllTaggedAsync<Intl>("Intl", "lang", "fr");
expect(fr.length).to.eql(2);
expect(fr[0].hello).to.eql("bonjour");
expect(fr[1].goodbye).to.eql("au revoir");

let es = await container.getAllTaggedAsync<Intl>("Intl", "lang", "es");
expect(es.length).to.eql(2);
expect(es[0].hello).to.eql("hola");
expect(es[1].goodbye).to.eql("adios");
```

#### container.isBound(serviceIdentifier: interfaces.ServiceIdentifier<any>): boolean

你可以使用`isBound`方法去检测给定服务标识符是否有注册的绑定。
```ts
interface Warrior {}
let warriorId = "Warrior";
let warriorSymbol = Symbol.for("Warrior");

@injectable()
class Ninja implements Warrior {}

interface Katana {}
let katanaId = "Katana";
let katanaSymbol = Symbol.for("Katana");

@injectable()
class Katana implements Katana {}

let container = new Container();
container.bind<Warrior>(Ninja).to(Ninja);
container.bind<Warrior>(warriorId).to(Ninja);
container.bind<Warrior>(warriorSymbol).to(Ninja);

expect(container.isBound(Ninja)).to.eql(true);
expect(container.isBound(warriorId)).to.eql(true);
expect(container.isBound(warriorSymbol)).to.eql(true);
expect(container.isBound(Katana)).to.eql(false);
expect(container.isBound(katanaId)).to.eql(false);
expect(container.isBound(katanaSymbol)).to.eql(false);
```

#### container.isBoundNamed(serviceIdentifier: interfaces.ServiceIdentifier<any>, named: string): boolean


你可以使用`isBoundNamed`方法使用给定命名约束去检测给定服务标识符是否有注册的绑定。
```ts
const zero = "Zero";
const invalidDivisor = "InvalidDivisor";
const validDivisor = "ValidDivisor";
let container = new Container();

expect(container.isBound(zero)).to.eql(false);
container.bind<number>(zero).toConstantValue(0);
expect(container.isBound(zero)).to.eql(true);

container.unbindAll();
expect(container.isBound(zero)).to.eql(false);
container.bind<number>(zero).toConstantValue(0).whenTargetNamed(invalidDivisor);
expect(container.isBoundNamed(zero, invalidDivisor)).to.eql(true);
expect(container.isBoundNamed(zero, validDivisor)).to.eql(false);

container.bind<number>(zero).toConstantValue(1).whenTargetNamed(validDivisor);
expect(container.isBoundNamed(zero, invalidDivisor)).to.eql(true);
expect(container.isBoundNamed(zero, validDivisor)).to.eql(true);
```


#### container.isBoundTagged(serviceIdentifier: interfaces.ServiceIdentifier<any>, key: string, value: any): boolean

你可以使用`isBoundNamed`方法使用给定标签约束去检测给定服务标识符是否有注册的绑定。

```ts
const zero = "Zero";
const isValidDivisor = "IsValidDivisor";
let container = new Container();

expect(container.isBound(zero)).to.eql(false);
container.bind<number>(zero).toConstantValue(0);
expect(container.isBound(zero)).to.eql(true);

container.unbindAll();
expect(container.isBound(zero)).to.eql(false);
container.bind<number>(zero).toConstantValue(0).whenTargetTagged(isValidDivisor, false);
expect(container.isBoundTagged(zero, isValidDivisor, false)).to.eql(true);
expect(container.isBoundTagged(zero, isValidDivisor, true)).to.eql(false);

container.bind<number>(zero).toConstantValue(1).whenTargetTagged(isValidDivisor, true);
expect(container.isBoundTagged(zero, isValidDivisor, false)).to.eql(true);
expect(container.isBoundTagged(zero, isValidDivisor, true)).to.eql(true);
```

#### container.load(...modules: interfaces.ContainerModule[]): void

调用每一个模块的注册方法。查阅[容器模块](https://github.com/inversify/InversifyJS/blob/master/wiki/container_modules.md)

#### container.loadAsync(...modules: interfaces.AsyncContainerModule[]): Promise<void>

和前面的 load 类似，但是是异步注册。

#### container.rebind<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): : interfaces.BindingToSyntax<T>


你可以使用`rebind`方法去覆盖所有给定`serviceIdentifier`存在的绑定。这个函数返回一个`BindingToSyntax`实例，允许创建覆盖的绑定。
```ts
let TYPES = {
    someType: "someType"
};

let container = new Container();
container.bind<number>(TYPES.someType).toConstantValue(1);
container.bind<number>(TYPES.someType).toConstantValue(2);

let values1 = container.getAll(TYPES.someType);
expect(values1[0]).to.eq(1);
expect(values1[1]).to.eq(2);

container.rebind<number>(TYPES.someType).toConstantValue(3);
let values2 = container.getAll(TYPES.someType);
expect(values2[0]).to.eq(3);
expect(values2[1]).to.eq(undefined);
```

#### container.rebindAsync<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>): Promise<interfaces.BindingToSyntax<T>>

这是异步版本的 rebind。如果你知道失活是异步的，则你应该使用这个方法。如果你不确定，则使用这个方法！

### container.resolve<T>(constructor: interfaces.Newable<T>): T

解析和`container.get<T>(serviceIdentifier: ServiceIdentifier<T>)`很像，但是它允许用户去创建一个实例，甚至如果没有绑定被声明：
```ts
@injectable()
class Katana {
    public hit() {
        return "cut!";
    }
}

@injectable()
class Ninja implements Ninja {
    public katana: Katana;
    public constructor(katana: Katana) {
        this.katana = katana;
    }
    public fight() { return this.katana.hit(); }
}

const container = new Container();
container.bind(Katana).toSelf();

const tryGet = () => container.get(Ninja);
expect(tryGet).to.throw("No matching bindings found for serviceIdentifier: Ninja");

const ninja = container.resolve(Ninja);
expect(ninja.fight()).to.eql("cut!");
```

请注意，它只允许跳过依赖图中根元素的绑定声明（组合根）。所有子依赖（比如,前面例子的`Katana` ）将需要一个绑定去声明。

#### container.onActivation<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, onActivation: interfaces.BindingActivation<T>): void

使用指定标识符为所有依赖添加一个活跃处理器
```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana);
container.onActivation("Weapon", (context: interfaces.Context, katana: Katana): Katana | Promise<Katana> => {
    console.log('katana instance activation!');
    return katana;
});

let katana = container.get<Weapon>("Weapon");
```
#### onDeactivation<T>(serviceIdentifier: interfaces.ServiceIdentifier<T>, onDeactivation: interfaces.BindingDeactivation<T>): void

为所有的依赖标识符添加一个失活处理器。
```ts
let container = new Container();
container.bind<Weapon>("Weapon").to(Katana);
container.onDeactivation("Weapon", (katana: Katana): void | Promise<void> => {
    console.log('katana instance deactivation!');
});

container.unbind("Weapon");
```

#### container.restore(): void;

恢复容器状态到最新的快照。

#### container.snapshot(): void

保存容器状态用于之后给 restore 方法恢复使用。

#### container.unbind(serviceIdentifier: interfaces.ServiceIdentifier<any>): void

移除这个容器内这个服务标识符所有绑定。这件会导致[失活进程](https://github.com/inversify/InversifyJS/blob/master/wiki/deactivation_handler.md)。


#### container.unbindAsync(serviceIdentifier: interfaces.ServiceIdentifier<any>): Promise<void>

这是异步版本的 unbind。如果你知道失活是异步的，则这个应该使用。如果你不缺人，则使用这个方法。

#### container.unbindAll(): void

移除这个容器内的所有绑定。这将会导致[失活进程](https://github.com/inversify/InversifyJS/blob/master/wiki/deactivation_handler.md)。

#### container.unbindAllAsync(): Promise<void>

这是异步版本的 unbindAll。如果你知道失活是异步的，则使用这个。如果你不缺人，则使用这个方法！

#### container.unload(...modules: interfaces.ContainerModuleBase[]): void

异步版本的 unload。如果你知道失活是异步的，则这个方法应该被使用。如果你不缺人，则使用这个方法！

#### container.parent: Container | null;

访问该容器的层级

#### container.id: number

一个自动产生的唯一的标识符。