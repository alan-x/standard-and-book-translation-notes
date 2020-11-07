[已校对]
# 函数

- [参数声明](https://basarat.gitbook.io/typescript/type-system/functions#parameter-annotations)
- [返回类型声明](https://basarat.gitbook.io/typescript/type-system/functions#return-type-annotation)
- [可选参数](https://basarat.gitbook.io/typescript/type-system/functions#optional-parameters)
- [重载](https://basarat.gitbook.io/typescript/type-system/functions#overloading)

### 函数

TypeScript 类型系统对函数付出了很多的爱，毕竟他们是一个可组合系统的核心构建块。

#### 参数声明

当然你可以声明函数参数，就像声明其他变量：
```ts
// variable annotation
var sampleVariable: { bar: number }

// function parameter annotation
function foo(sampleParameter: { bar: number }) { }
```

这里我使用内联类型声明。当然你可以使用接口等。

#### 返回类型声明

你可以像对变量使用的风格一样在函数参数列表之后声明返回类型，比如，下面例子中的`: Foo`：
```ts
interface Foo {
    foo: string;
}

// Return type annotated as `: Foo`
function foo(sample: Foo): Foo {
    return sample;
}
```

当然，我在这里使用了`interface`，但是你可以使用使用其他声明，比如，内联声明。

通常你不需要去声明一个函数的返回值，因为它通常可以被编译器推断出来。

```ts
interface Foo {
    foo: string;
}

function foo(sample: Foo) {
    return sample; // inferred return type 'Foo'
}
```

然而，添加这些声明去帮助处理错误通常是一个好主意，比如：
```ts
function foo() {
    return { fou: 'John Doe' }; // You might not find this misspelling of `foo` till it's too late
}

sendAsJSON(foo());
```

如果你不打算从一个函数返回任何东西，你可以声明它为`:void`。你通常可以移除`:void`，让他留给引擎推断。？

#### 可选参数

你可以标记参数为可选的：
```ts
function foo(bar: number, bas?: string): void {
    // ..
}

foo(123);
foo(123, 'hello');
```

此外，你甚至可以提供默认的值（在参数声明后面使用`= someValue`），如果调用者没有提供这个参数，就会为你注入：
```ts
function foo(bar: number, bas: string = 'hello') {
    console.log(bar, bas);
}

foo(123);           // 123, hello
foo(123, 'world');  // 123, world
```

#### 重载

TypeScript 允许你去声明函数重载。这对文档 + 类型安全目的很有用。考虑下面代码：
```ts
function padding(a: number, b?: number, c?: number, d?: any) {
    if (b === undefined && c === undefined && d === undefined) {
        b = c = d = a;
    }
    else if (c === undefined && d === undefined) {
        c = a;
        d = b;
    }
    return {
        top: a,
        right: b,
        bottom: c,
        left: d
    };
}
```

如果你小心查看代码，你会发现`a`、`b`、`c`、`d`的意义基于传入的参数的数量改变。当然，函数只期待`1`，`2`，或者`4`个参数。这些约束可以使用函数重载约束和记录。你只是声明函数头多次。最后的函数头是真正激活函数体的，但是不能被外界使用。

这显示在下面：
```ts
// Overloads
function padding(all: number);
function padding(topAndBottom: number, leftAndRight: number);
function padding(top: number, right: number, bottom: number, left: number);
// Actual implementation that is a true representation of all the cases the function body needs to handle
function padding(a: number, b?: number, c?: number, d?: number) {
    if (b === undefined && c === undefined && d === undefined) {
        b = c = d = a;
    }
    else if (c === undefined && d === undefined) {
        c = a;
        d = b;
    }
    return {
        top: a,
        right: b,
        bottom: c,
        left: d
    };
}
```

前三个函数头可以有效的调用`padding`：
```ts
padding(1); // Okay: all
padding(1,1); // Okay: topAndBottom, leftAndRight
padding(1,1,1,1); // Okay: top, right, bottom, left

padding(1,1,1); // Error: Not a part of the available overloads
```

当然，最终声明（真实声明可以在函数内部看到）兼容所有的重载很重要。这是因为函数体需要关系函数调用的真实本质。

> TypeScript 中的函数重载没有任何运行时重载。它只允许你去记录你期待函数被调用的手段和编译器对你剩下的代码的检测。

### 声明函数

> 快速提示：类型声明是描述已存在的实现的类型的方式

有两种声明一个没有提供实现的函数的类型的方式。比如。
```ts
type LongHand = {
    (a: number): number;
};

type ShortHand = (a: number) => number;
```

签名的两个是完全相同的。当你想要添加不同的时候，就会有重载。你只可以在长版本中添加重载，比如：
```ts
type LongHandAllowsOverloadDeclarations = {
    (a: number): number;
    (a: string): string;
};
```