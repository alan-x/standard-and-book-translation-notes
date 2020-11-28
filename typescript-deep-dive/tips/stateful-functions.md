[已校对]
# 有状态的函数

在其他编程语言的一个常见特性是使用`static`关键字去增加一个函数变量的寿命（不是范围的）去会在函数调用之外。这是达到这个的一个`C`例子：
```ts
void called() {
    static count = 0;
    count++;
    printf("Called : %d", count);
}

int main () {
    called(); // Called : 1
    called(); // Called : 2
    return 0;
}
```

因为 JavaScript(或者 TypeScript)没有函数静态，你可以达到相同的目的，使用多个抽象，使用本地变量包裹，比如，使用一个`class`：
```ts
const {called} = new class {
    count = 0;
    called = () => {
        this.count++;
        console.log(`Called : ${this.count}`);
    }
};

called(); // Called : 1
called(); // Called : 2
```

> C++ 开发者当然也尝试使用一个叫做`functor`的模式达到这个（一个覆盖`()`操作符的类）。

