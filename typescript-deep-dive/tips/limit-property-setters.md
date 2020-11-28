[已校对]
# 限制属性设置器

优先使用显示的 set/get 函数（比如，`setBar`和`getBar`函数），而不是 setters/getters。

假设有下面代码：
```ts
foo.bar = {
    a: 123,
    b: 456
};
```

存在 setter/getter 的时候：
```ts
class Foo {
    a: number;
    b: number;
    set bar(value:{a:number,b:number}) {
        this.a = value.a;
        this.b = value.b;
    }
}
let foo = new Foo();
```

这不是一个好的属性设计器的使用。阅读第一个代码例子的人没有关于将会改变的任何东西的上下文。尽管一些人调用`foo.setBar(value)`可能知道有一些东西会在`foo`改变。

> Bonus 观点：如果你有不同的函数，会发现引用更好。在 TypeScript，如果你为一个 getter 或者一个 setter 查找引用，你会找到两者，然而使用显示函数调用，你只能得到相关函数的引用。