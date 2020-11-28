# 类很有用

下面的结构很常见：
```ts
function foo() {
    let someProperty;

    // Some other initialization code

    function someMethod() {
        // Do some stuff with `someProperty`
        // And potentially other things
    }
    // Maybe some other methods

    return {
        someMethod,
        // Maybe some other methods
    };
}
```

这作为 revealing 模块模式被知道，在 JavaScript（利用 JavaScript 的闭包）中很常见。

如果你使用[文件模式（你应该这么做，因为全局范围不好）](https://basarat.gitbook.io/typescript/project/modules)，则你的文件的效果相同。然而，有很多的例子表明，人们将会如下编写代码：
```ts
let someProperty;

function foo() {
   // Some initialization code
}
foo(); // some initialization code

someProperty = 123; // some more initialization

// Some utility function not exported

// later
export function someMethod() {

}
```

尽管我不是继承的大粉丝，我的确发现人们使用类帮助他们更好的组织代码。同样的开发者会直观的如下编写：
```ts
class Foo {
    public someProperty;

    constructor() {
        // some initialization
    }

    public someMethod() {
        // some code
    }

    private someUtility() {
        // some code
    }
}

export = new Foo();
```

不仅仅是开发者，创建基于类的提供好的可视化开发工具更加普通，这样你的团队就少一种模式需要去理解和维护。

> PS：如果他们提供重要的重用和减少样板代码，那么在我看来浅层类继承在我看来没有啥错误。

