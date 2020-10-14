# 命名空间

命名空间为 JavaScript 中常用的模式提供你一个便捷的语法：
```ts
(function(something) {

    something.foo = 123;

})(something || (something = {}))
```

基本上`something || (something = {})`允许一个匿名的函数`function(something) {}`添加东西到一个存在的对象（`something ||`部分），或者开始一个新的对象，然后添加东西到这个对象（`|| (something = {})`部分）。这意味着你可以有两个这种块通过一些执行包裹分离：
```ts
(function(something) {

    something.foo = 123;

})(something || (something = {}))

console.log(something); // {foo:123}

(function(something) {

    something.bar = 456;

})(something || (something = {}))

console.log(something); // {foo:123, bar:456}
```

这在 JavaScript 中很常见，用于确保东西不会泄露到全局命名空间。使用基于文件的模块，你不需要担心这个，但是这个模式对于逻辑分组一串的函数依旧很有用。因此，TypeScript 提供了`namespace`关键字去分组这些，比如：
```ts
namespace Utility {
    export function log(msg) {
        console.log(msg);
    }
    export function error(msg) {
        console.error(msg);
    }
}

// usage
Utility.log('Call me');
Utility.error('maybe!');
```

`namespace`关键字生成和我们前面看到的相同的 JavaScript：
```ts
(function (Utility) {

// Add stuff to Utility

})(Utility || (Utility = {}));
```

要注意，命名空间可以嵌套，因此你可以做类似`namespace Utility.Messaging`去嵌套一个`Messaging`命名空间在`Utility`下。

对于大部分项目，我们推荐使用外部模块，并使用`namepsace`作为快速的 demo，和传输旧的 JavaScript 代码。