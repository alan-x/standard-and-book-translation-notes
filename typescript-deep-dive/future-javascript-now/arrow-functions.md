[已校对]
# 箭头函数

### 箭头函数

被亲切的称为胖箭头（因为`->`是一个瘦箭头，`=>`是一个胖箭头），并且也叫做 lambada 函数（因为其他语言）。另一使用的特性是胖箭头函数`()=>something`。胖箭头的动机是：

1. 你不需要去保持输入`function`
2. 它词法上捕获`this`的意义
4. 它词法上捕获`arguments`的意义

对于一个声称为函数式的语言，在 JavaScript 你往往输入`function`非常多余。胖箭头让你创建一个函数更加简单
```ts
var inc = (x)=>x+1;
```

`this`通常是 JavaScript 的一个痛点。正如一位智者说的“我讨厌 JavaScript 因为它往往简单的丢失`this`”。胖箭头修复它，通过从环境上下文捕获`this`的意义。考虑这个纯洁的 JavaScript 类：
```ts
function Person(age) {
    this.age = age;
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, should have been 2
```

如果你在浏览器运行这个代码，函数内的`this`将会指向`window`，因为`window`将会执行`growOld`函数。使用一个箭头函数修复：
```ts
function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

这可以工作的原因是`this`的引用被箭头函数从函数体外部捕获。这和下面的 JavaScript 代码相同（这是你可能会写的，如果你没有 TypeScript）：
```ts
function Person(age) {
    this.age = age;
    var _this = this;  // capture this
    this.growOld = function() {
        _this.age++;   // use the captured this
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

注意，因为你使用 TypeScript，你甚至可以在语法上更加精简，并结合箭头函数和类：
```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

[关于这个模式一个甜美的视频](https://egghead.io/lessons/typescript-make-usages-of-this-safe-in-class-methods)

### 提示：箭头函数需求

除了简短的语法，你只需要使用箭头函数，如果你要吧函数给其他去调用。实际上：
```ts
var growOld = person.growOld;
// Then later someone else calls it:
growOld();
```
如果你想要自己去调用，比如：
```ts
person.growOld();

```
`this`将会正确的调用上下文（在这个例子是`person`）。

### 提示：箭头函数危险

实际上，如果你想要`this`成为调用上下文，你不应该使用箭头函数。这是被使用回调的类似 jquery，undescore，mocha 和其他的库的场景。如果文档提示行数在`this`上，则你应该只使用`function`，而不是胖箭头。同样的，如果你计划使用`arguments`，不要使用胖箭头。

### 提示：箭头函数和使用`this`的库

很多库这么做，比如`jQuery`迭代（一个例子是[https://api.jquery.com/jquery.each/](https://api.jquery.com/jquery.each/)）将使用`this`去传递你它当前遍历的对象。在这个场景，如果你想要访问库传递的`this`，只要包裹的上下文只使用一个临时变量，类似`_self`，就像你将会在箭头函数中缺省的那样。
```ts
let _self = this;
something.each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```

### 提示：箭头函数和继承

箭头函数作为类的属性和继承合作的很好：
```ts
class Adder {
    constructor(public a: number) {}
    add = (b: number): number => {
        return this.a + b;
    }
}
class Child extends Adder {
    callAdd(b: number) {
        return this.add(b);
    }
}
// Demo to show it works
const child = new Child(123);
console.log(child.callAdd(123)); // 246
```

然而，他们无法和`super`关键字一起使用，当你尝试在子类覆盖一个函数的时候。属性在`this`上。因此只有一个`this`，这类函数不能参与`super`的调用（`super`只在原型成员上可用）。你可以简单的绕过它，通过创建方法的一个复制，在子类覆盖他之前。
```ts
class Adder {
    constructor(public a: number) {}
    // This function is now safe to pass around
    add = (b: number): number => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Create a copy of parent before creating our own
    private superAdd = this.add;
    // Now create our override
    add = (b: number): number => {
        return this.superAdd(b);
    }
}
```

### 提示：快速对象返回



有时候你需要一个函数只返回简单对象字面量，就像
```ts
// WRONG WAY TO DO IT
var foo = () => {
    bar: 123
};
```

会被 JavaScirpt 运行时转化为块包含一个 JavaScript 标签（因为 JavaScript 规格）。

> 如果这没有意义，不用担心，因为你会从 TypeScript 得到一个漂亮的编译器错误，“未使用的标签”。白噢钱是一个旧的（）JavaScript 特性，你可以当作现代的 GOTO 忽略（被有经验的开发者认为是很坏的）。

你可以修复它，通过使用`()`包裹对象字面量：
```ts
// Correct 🌹
var foo = () => ({
    bar: 123
});
```