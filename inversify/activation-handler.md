### 激活处理器

为一个类型添加激活处理器是可能的。激活处理器在一个依赖被解析之后，添加到缓存和注入之前被调用（如果单例或者请求当例 - [查阅范围](https://github.com/inversify/InversifyJS/blob/master/wiki/scope.md)）。激活处理器将不会调用，如果依赖来自缓存。激活处理器可以是异步或者同步的。

激活处理器对于保持我们横切关注点依赖实现不可知非常有用，比如缓存或者日志。

下面的例子使用一个[proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)去拦截依赖（`Katana`）的一个方法（`use`）。

```ts
interface Katana {
    use: () => void;
}

@injectable()
class Katana implements Katana {
    public use() {
        console.log("Used Katana!");
    }
}

interface Ninja {
    katana: Katana;
}

@injectable()
class Ninja implements Ninja {
    public katana: Katana;
    public constructor(@inject("Katana") katana: Katana) {
        this.katana = katana;
    }
}
```

```ts
container.bind<Ninja>("Ninja").to(Ninja);

container.bind<Katana>("Katana").to(Katana).onActivation((context, katana) => {
    let handler = {
        apply: function(target, thisArgument, argumentsList) {
            console.log(`Starting: ${new Date().getTime()}`);
            let result = target.apply(thisArgument, argumentsList);
            console.log(`Finished: ${new Date().getTime()}`);
            return result;
        }
    };
    katana.use = new Proxy(katana.use, handler);
    return katana;
});
```
```ts
let ninja = container.get<Ninja>();
ninja.katana.use();
> Starting: 1457895135761
> Used Katana!
> Finished: 1457895135762
```

有多种方式可以提供一个激活处理器
- 添加一个处理器到容器
- 添加一个处理器到绑定

当多个激活处理器绑定到一个服务标识符，绑定处理器在任何其他之前调用。然后容器处理器被调用，从根容器开始，以子孙容器降序，并在绑定的容器处停止。