### 失活处理器

在单例范围为一个类型绑定添加一个失活处理器是可能的。处理器可以是异步或者同步的。失活处理器将会在类型从容器解绑的时候调用：
```ts
@injectable()
class Destroyable {
}

const container = new Container();
container.bind<Destroyable>("Destroyable").toDynamicValue(() => Promise.resolve(new Destroyable())).inSingletonScope()
    .onDeactivation((destroyable: Destroyable) => {
        console.log("Destroyable service is about to be unbinded");
    });

await container.get("Destroyable");

await container.unbind("Destroyable");
```

添加一个失活处理器有多种方式
- 添加一个处理器到容器
- 添加一个处理器到绑定
- 添加一个处理器到一个类，通过一个 [preDestory 装饰器](https://github.com/inversify/InversifyJS/blob/master/wiki/pre_destroy.md)

添加到容器的处理器是第一个被解析的。任何添加到子容器的处理器在添加到他们的父级之前被调用。容器中相关的绑定在之后调用，最终，`preDestroy`方法被调用。在前面的例子，相关的绑定是那些包裹解绑的`Destroyable`服务标识符的的绑定。

下面的例子展示了调用顺序。
```ts
let roll = 1;
let binding = null;
let klass = null;
let parent = null;
let child = null;

@injectable()
class Destroyable {
    @preDestroy()
    public myPreDestroyMethod() {
        return new Promise((presolve) => {
            klass = roll;
            roll += 1;
            presolve({});
        });
    }
}

const container = new Container();
container.onDeactivation("Destroyable", () => {
    return new Promise((presolve) => {
        parent = roll;
        roll += 1;
        presolve();
    });
});

const childContainer = container.createChild();
childContainer.bind<Destroyable>("Destroyable").to(Destroyable).inSingletonScope().onDeactivation(() => new Promise((presolve) => {
    binding = roll;
    roll += 1;
    presolve();
}));
childContainer.onDeactivation("Destroyable", () => {
    return new Promise((presolve) => {
        child = roll;
        roll += 1;
        presolve();
    });
});

childContainer.get("Destroyable");
await childContainer.unbindAsync("Destroyable");

expect(roll).eql(5);
expect(child).eql(1);
expect(parent).eql(2);
expect(binding).eql(3);
expect(klass).eql(4);
```