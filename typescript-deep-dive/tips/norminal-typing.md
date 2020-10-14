# Norminal Typing

### Norminal Typing

TypeScript 类型系统是结构化的，[并且这是收益的主要动机之一]()。然而，真实世界的系统有一些使用场景，你希望两个变量不同，因为他们有不同的类型名字，就算他们有相同的结构。一个非常常见的额使用场景是标示结构（通常只是字符串，但是有语义和他们的名字关联，在类似 C#/Java 的语言中）。

有一些模式出现在社区，我按个人的喜好倒序覆盖他们：


### 使用字面量类型

这个模式使用泛型和字面量类型：
```ts
/** Generic Id type */
type Id<T extends string> = {
  type: T,
  value: string,
}

/** Specific Id types */
type FooId = Id<'foo'>;
type BarId = Id<'bar'>;

/** Optional: constructors functions */
const createFoo = (value: string): FooId => ({ type: 'foo', value });
const createBar = (value: string): BarId => ({ type: 'bar', value });

let foo = createFoo('sample')
let bar = createBar('sample');

foo = bar; // Error
foo = foo; // Okay
```

- 优点
    - 不需要任何类型断言
- 缺点
    - 结构`{type, value}`可能不是期待的，并且需要服务器的序列化支持。

### 使用枚举

[TypeScript 中国内地额枚举]()提供某个级别的 norminal typing。两个枚举类型不想等，如果他们名字不同。我们使用这个事实为结构上兼容的类型提供 nominal typing。

解决方案包括：
- 创建一个 brand 枚举
- 创建类型作为一个 brand 枚举 + 实际结构的交叉（`&`）。

这显示在下面，类型的结构只是一个字符串：
```ts
// FOO
enum FooIdBrand { _ = "" };
type FooId = FooIdBrand & string;

// BAR
enum BarIdBrand  { _ = "" };
type BarId = BarIdBrand & string;

/**
 * Usage Demo
 */
var fooId: FooId;
var barId: BarId;

// Safety!
fooId = barId; // error
barId = fooId; // error

// Newing up
fooId = 'foo' as FooId;
barId = 'bar' as BarId;

// Both types are compatible with the base
var str: string;
str = fooId;
str = barId;
```

注意 brand 枚举是如何，前面的`FooIdBrand`和`BarIdBrand`，每一个都以一个单独的成员（`_`）映射到一个空字符串，通过`{_=""}`指定。这强制 TypeScript 去推断这些基于字符串的枚举，使用`string`类型的值，而不是没使用`number`类型的枚举值。这是必须的，因为 TypeScriot 推断一个空枚举（`{}`）到一个数字化枚举，在 TypeScript 3.6.2，数字化`enum`和`string`的交叉是`never`。

### 使用接口

因为`number`和`enum`是类型兼容的，前面的技术不能为他们使用。相反，我们使用接口去破坏结构兼容。这方法依旧被 TypeScript 编译器团队使用，因此值得提起。使用`_`前缀和一个`Brand`后缀是一个我强烈推荐的惯例（并且[这跟随 TypeScript 团队]()）。

这个解决方案包含如下：

- 添加一个未使用的属性在一个类型去破坏结构化兼容性。
- 使用一个类型断言，当需要去向上或者向下转型的时候。

这显示在下面：
```ts
// FOO
interface FooId extends String {
    _fooIdBrand: string; // To prevent type errors
}

// BAR
interface BarId extends String {
    _barIdBrand: string; // To prevent type errors
}

/**
 * Usage Demo
 */
var fooId: FooId;
var barId: BarId;

// Safety!
fooId = barId; // error
barId = fooId; // error
fooId = <FooId>barId; // error
barId = <BarId>fooId; // error

// Newing up
fooId = 'foo' as any;
barId = 'bar' as any;

// If you need the base string
var str: string;
str = fooId as any;
str = barId as any;
```