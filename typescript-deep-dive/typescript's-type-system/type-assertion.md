# 类型断言

TypeScript 允许你以你想要的任何方式去覆盖他对类型的视图的推断和分析。这通过叫做“类型断言”的机制实现。TypeScript 的类型断言仅仅告诉编译器你对类型知道的比它多，而它不应该二次猜测你。

类型断言的一个常见使用场景当你从 JavaScript 迁移代码到 TypeScript。比如考虑下面的模式：
```ts
var foo = {};
foo.bar = 123; // Error: property 'bar' does not exist on `{}`
foo.bas = 'hello'; // Error: property 'bas' does not exist on `{}`
```

这里的代码错误是因为`foo`的类型推断是`{}`，比如，一个没有属性的对象。因此你不允许添加`bar`或者`bas`到它。你可以修复这个，简单的通过一个类型断言`as Foo`:
```ts
interface Foo {
    bar: number;
    bas: string;
}
var foo = {} as Foo;
foo.bar = 123;
foo.bas = 'hello';
```

### `as foo` vs `<foo>`

最初添加的语法是`<foo>`。这显示在下面：
```ts
var foo: any;
var bar = <string> foo; // bar is now of type "string"
```

然而，在 JSX 中，使用`<foo>`风格断言在语法上有歧义：
```ts
var foo = <string>bar;
</string>
```
因此，为了一致性，现在推荐你只使用`as foo`。

### 类型断言和转换

它不叫做“类型转化”的原因是因为转化通常暗示着一些运行时支持。然而，类型断言只是一个编译时构造，和让你提供提示给编译器你想要怎样分析你的代码的一个方式，

### 断言被认为是有害的

在很多场景，断言将会允许你去简单升级遗留代码（甚至复制粘贴其他代码例子到你的代码库）。然而，你应该小心使用断言。使用我们的代码作为一个例子，编译器不会从忘记实际添加你承诺的属性中保护你：
```ts
interface Foo {
    bar: number;
    bas: string;
}
var foo = {} as Foo;
// ahhhh .... forget something?
```

当然，另一种常见想法是使用断言作为提供自动完成的手段，比如：
```ts
interface Foo {
    bar: number;
    bas: string;
}
var foo = <Foo>{
    // the compiler will provide autocomplete for properties of Foo
    // But it is easy for the developer to forget adding all the properties
    // Also this code is likely to break if Foo gets refactored (e.g. a new property added)
};
```

但是危害依旧相同，如果你忘记一个属性，编译器将不会抱怨。如果你如下执行将会更好：
```ts
interface Foo {
    bar: number;
    bas: string;
}
var foo: Foo = {
    // the compiler will provide autocomplete for properties of Foo
};
```

在某些场景，你可能需要创建一个临时变量，但是至少你将不会承诺（可能是错的）并依赖类型索引为你做类型检测。

### 双重断言

类型断言，正如我们显示的，尽管有一点不安全，并不是完全开放的。比如，下面是一个非常有用的例子（比如，用户想要传递进来的事件是更特殊的）并且类型断言如预期使用：
```ts
function handler (event: Event) {
    let mouseEvent = event as MouseEvent;
}
```
然而，下面大概有一个错误，并且 TypeScript 将会如显示抱怨，尽管用户的类型断言：
```ts
function handler(event: Event) {
    let element = event as HTMLElement; // Error: Neither 'Event' nor type 'HTMLElement' is assignable to the other
}
```

如果你依旧想要这个这个类型，聂可以使用双重断言，但是第一次断言到`unknow`(或者`any`），它和任何类型都兼容，因此，编译器不再抱怨：
```ts
function handler(event: Event) {
    let element = event as unknown as HTMLElement; // Okay!
}
```


### TypeScript 是如何判断单个断言是不够的

基本上，从类型`S`到`T`的断言成功只有如果`S`是`T`的子类型，护着`T`是`S`的子类型。这在执行类型断言的时候提供了额外的安全...完全狂热的断言会非常不安全，并且你需要使用`unknow`（或者`any`）去达到这个不安全。