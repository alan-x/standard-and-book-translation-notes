### 注入一个类构造器

InversifyJS 支持构造器注入，允许在可注入对象创建过程中传递一个抽象或者具体类的实例。

对于抽象（接口）你需要使用 @inject 装饰器。这是必须的，因为抽象的元数据在运行时是不够的：
```ts
@injectable()
class Ninja implements Ninja {

    private _katana: Katana;
    private _shuriken: Shuriken;

    public constructor(
	    @inject("Newable<Katana>") Katana: interfaces.Newable<Katana>, 
	    @inject("Shuriken") shuriken: Shuriken
	) {
        this._katana = new Katana();
        this._shuriken = shuriken;
    }

    public fight() { return this._katana.hit(); };
    public sneak() { return this._shuriken.throw(); };

}
```

```ts
container.bind<interfaces.Newable<Katana>>("Newable<Katana>").toConstructor<Katana>(Katana);

```

对于具体注入，你可以简单定义你的构造器参数，不需要使用 @inject 装饰器。

InversifyJS 也支持 TypeScript 的构造器赋值，因此在你的参数，你可以有 private 和 protected 访问修复器，容器注入依赖没啥问题：
```ts
@injectable()
class Ninja implements Ninja {

    public constructor(private _dagger:Dagger) {

    }

    public throwDagger() {
        this._dagger.throw();
    }

}
```
```ts
container.bind<Dagger>(Dagger).toSelf()

```