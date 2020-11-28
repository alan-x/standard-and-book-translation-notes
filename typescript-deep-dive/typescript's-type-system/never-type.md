[已校对]
# never 类型

### never
> [Youtube: never 类型的视频课程](https://www.youtube.com/watch?v=aldIFYWu6xc)
> [Egghead: never 类型的视频课程](https://egghead.io/lessons/typescript-use-the-never-type-to-avoid-code-with-dead-ends-using-typescript)

编程语言设计的确有一个兜底类型的概念，在你进行代码流分析之后，就能自然得出结论。TypeScript 使用代码流分析（😎），因此它需要可靠的表示可能永远不会发生的的东西。

用在 TypeScrpt 的`never`类型去贡献这个兜底类型。这是自然发生的情况：

- 一个函数永远不会返回（比如，如果函数体有`whilte(true){}`）
- 一个函数总是抛出（比如，在`function foo(){throw new Error('Not Implemented')}`，`foo`的返回类型是`never`）

当然你可以自己使用这个声明
```ts
let foo: never; // Okay
```
然而，只有`never`可以被赋值给其他 never，比如
```ts
let foo: never = 123; // Error: Type number is not assignable to never

// Okay as the function's return type is `never`
let bar: never = (() => { throw new Error(`Throw my hands in the air like I just don't care`) })();
```
很好，现在进入它的主要使用场景:)

### 用例：全面检查

你可以在一个 never 上下文调用 never 函数。
```ts
function foo(x: string | number): boolean {
  if (typeof x === "string") {
    return true;
  } else if (typeof x === "number") {
    return false;
  }

  // Without a never type we would error :
  // - Not all code paths return a value (strict null checks)
  // - Or Unreachable code detected
  // But because TypeScript understands that `fail` function returns `never`
  // It can allow you to call it as you might be using it for runtime safety / exhaustive checks.
  return fail("Unexhaustive!");
}

function fail(message: string): never { throw new Error(message); }
```

因为`never`只能赋值给其他`never`，你也可以用他做编译时详细检测。这在[区分联合章节](https://basarat.gitbook.io/typescript/type-system/discriminated-unions)被覆盖。


### 和`void`的混乱

当一个函数永远不会优雅退出，只要告诉你`never`被返回，直觉上，你认为和`void`相同。然而，`void`是一个单元，`never`是一个假值。

一个啥也没有返回的函数返回一个`void`单元。然而，一个函数永远不会返回（或者总是抛出）返回`never`。`void`是可以被赋值的（没有`strictNullChecking`），但是`never`永远不能赋值给 `never` 之外。

### never 返回函数的类型推断

对于函数声明，TypeScript 默认推断`void`，如下显示：
```ts
// Inferred return type: void
function failDeclaration(message: string) {
  throw new Error(message);
}

// Inferred return type: never
const failExpression = function(message: string) {
  throw new Error(message);
};
```
当然你可以通过显式声明修复这个：
```ts
function failDeclaration(message: string): never {
  throw new Error(message);
}
```

主要原因是向后兼容真实世界 JavaScript 代码：
```ts
class Base {
    overrideMe() {
        throw new Error("You forgot to override me!");
    }
}

class Derived extends Base {
    overrideMe() {
        // Code that actually returns here
    }
}
```
如果`Base.overrideMe`.

> 真实世界的 TypeScript 可以使用`abstract`函数克服这个，但是这个推断是为了保持兼容。