[已校对]
# 单例模式

便捷的单例模式是用于克服所有代码都必须在`class`中的事实：
```ts
class Singleton {
    private static instance: Singleton;
    private constructor() {
        // do something construct...
    }
    static getInstance() {
        if (!Singleton.instance) {
            Singleton.instance = new Singleton();
            // ... any one time initialization goes here ...
        }
        return Singleton.instance;
    }
    someMethod() { }
}

let something = new Singleton() // Error: constructor of 'Singleton' is private.

let instance = Singleton.getInstance() // do something with the instance...
```

然而，如果你不想要懒初始化，你可以使用一个命名空间代替：
```ts
namespace Singleton {
    // ... any one time initialization goes here ...
    export function someMethod() { }
}
// Usage
Singleton.someMethod();
```

> 警告：单例只是[全局](http://stackoverflow.com/a/142450/390330)的一个花名

对于大部分项目，`namespace`可以被一个模块替换。
```ts
// someFile.ts
// ... any one time initialization goes here ...
export function someMethod() { }

// Usage
import {someMethod} from "./someFile";
```