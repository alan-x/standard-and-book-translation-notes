工具类型


TypeScript 提供多个工具类去促进常见类型转化。这些工具全局可用。

### Partial<Type>

构造一个拥有`Type`所有的属性的类型，并且都设置为可选。这个工具将返回一个类型，表示给定类型的所有子集。

例子
```ts
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};

const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```


### Readonly<Type>

构造一个拥有`Type`所有属性的类型，并且都设置为`readonly`，意味着构造的类型的属性不能被重新赋值。

例子

```
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};

todo.title = "Hello";
Cannot assign to 'title' because it is a read-only property.

```

这个工具对表示运行时将会失败的赋值表达非常有用（比如，当尝试去重新赋值一个[冰冻对象]()的属性）

### Record<Keys, Type>

构造一个类型，拥有一系列的属性集合`Keys`，`Keys`属于`Type`。这个工具可以用于映射一个类型的属性到另一个类型。

例子
```
interface PageInfo {
  title: string;
}

type Page = "home" | "about" | "contact";

const nav: Record<Page, PageInfo> = {
  about: { title: "about" },
  contact: { title: "contact" },
  home: { title: "home" },
};

nav.about;
//      ^ = Could not get LSP result: bou>t<;
/
```

### Pick<Type, Keys>

构造一个类型，从`Type`选择一个集合的属性`Keys`

例子

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};

todo;
// ^ = const todo: Pick
```

### Omit<Type, Keys>

构造一个类型，通过从`Type`选择所有的属性，让后移除`Keys`。

例子：
```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, "description">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};

todo;
// ^ = const todo: Pick
```

### Exclude<Type, ExcludedUnion>

构造一个类型，通过从`Type`排除所有可以被赋值给`ExcludedUnion`的联合成员。
```
type T0 = Exclude<"a" | "b" | "c", "a">;
//    ^ = type T0 = "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;
//    ^ = type T1 = "c"
type T2 = Exclude<string | number | (() => void), Function>;
//    ^ = type T2 = string | number
```

### Extract<Type, Union>

构造一个类型，通过从`Type`提取所有可以被赋值给`Union`的联合成员。

例子
```
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
//    ^ = type T0 = "a"
type T1 = Extract<string | number | (() => void), Function>;
//    ^ = type T1 = () => void
```
### NonNullable<Type>
构造一个类型，通过从`Type`排除`null`和`undefined`。

例子
```
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
//    ^ = type T0 = "a"
type T1 = Extract<string | number | (() => void), Function>;
//    ^ = type T1 = () => void
```

### Parameters<Type>

从用在函数类型`Type`的参数构造一个元组类型。

例子
```ts
declare function f1(arg: { a: number; b: string }): void;

type T0 = Parameters<() => string>;
//    ^ = type T0 = []
type T1 = Parameters<(s: string) => void>;
//    ^ = type T1 = [s: string]
type T2 = Parameters<<T>(arg: T) => T>;
//    ^ = type T2 = [arg: unknown]
type T3 = Parameters<typeof f1>;
//    ^ = type T3 = [arg: {
    a: number;
    b: string;
}]
type T4 = Parameters<any>;
//    ^ = type T4 = unknown[]
type T5 = Parameters<never>;
//    ^ = type T5 = never
type T6 = Parameters<string>;
Type 'string' does not satisfy the constraint '(...args: any) => any'.
//    ^ = type T6 = never
type T7 = Parameters<Function>;
Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  Type 'Function' provides no match for the signature '(...args: any): any'.
//    ^ = type T7 = never
```

### ConstructorParameters<Type>

从一个构造器函数类型构造一个元组或者数组类型。它产生一个带所有参数类型的元组类型（或者，如果`Type`不是一个函数，则是`never`）。

例子
```
type T0 = ConstructorParameters<ErrorConstructor>;
//    ^ = type T0 = [message?: string]
type T1 = ConstructorParameters<FunctionConstructor>;
//    ^ = type T1 = string[]
type T2 = ConstructorParameters<RegExpConstructor>;
//    ^ = type T2 = [pattern: string | RegExp, flags?: string]
type T3 = ConstructorParameters<any>;
//    ^ = type T3 = unknown[]

type T4 = ConstructorParameters<Function>;
Type 'Function' does not satisfy the constraint 'new (...args: any) => any'.
  Type 'Function' provides no match for the signature 'new (...args: any): any'.
//    ^ = type T4 = never
```

### ReturnType<Type>

构造一个由`Type`返回类型组成的类型

例子
```ts
declare function f1(): { a: number; b: string };

type T0 = ReturnType<() => string>;
//    ^ = type T0 = string
type T1 = ReturnType<(s: string) => void>;
//    ^ = type T1 = void
type T2 = ReturnType<<T>() => T>;
//    ^ = type T2 = unknown
type T3 = ReturnType<<T extends U, U extends number[]>() => T>;
//    ^ = type T3 = number[]
type T4 = ReturnType<typeof f1>;
//    ^ = type T4 = {
    a: number;
    b: string;
}
type T5 = ReturnType<any>;
//    ^ = type T5 = any
type T6 = ReturnType<never>;
//    ^ = type T6 = never
type T7 = ReturnType<string>;
Type 'string' does not satisfy the constraint '(...args: any) => any'.
//    ^ = type T7 = any
type T8 = ReturnType<Function>;
Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  Type 'Function' provides no match for the signature '(...args: any): any'.
//    ^ = type T8 = any
```

### InstanceType<Type>

构造一个类型，由`Type`的构造器函数的实例类型组成。

例子
```ts
class C {
  x = 0;
  y = 0;
}

type T0 = InstanceType<typeof C>;
//    ^ = type T0 = C
type T1 = InstanceType<any>;
//    ^ = type T1 = any
type T2 = InstanceType<never>;
//    ^ = type T2 = never
type T3 = InstanceType<string>;
Type 'string' does not satisfy the constraint 'new (...args: any) => any'.
//    ^ = type T3 = any
type T4 = InstanceType<Function>;
Type 'Function' does not satisfy the constraint 'new (...args: any) => any'.
  Type 'Function' provides no match for the signature 'new (...args: any): any'.
//    ^ = type T4 = any
```

### Required<Type>

构造一个类型，由`Type`所有个属性组成，并设为必须。于此相反的是[Partial]()

例子
```
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 };

const obj2: Required<Props> = { a: 5 };
Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```

### ThisParameterType<Type>

为一个函数类型提取[this]()参数的类型，如果函数类型没有`this`参数，就是`unknow`。

例子
```ts
function toHex(this: Number) {
  return this.toString(16);
}

function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```

### OmitThisParameter<Type>

从`Type`移除[this]()参数。如果`Type`没有明确的声明`this`参数，结果是简单的`Type`。否则，一个新的没有`this`参数的函数类型从`Type`创建。泛型被移除，只有最后一个重载签名被传播到新的函数类型。

例子
```
function toHex(this: Number) {
  return this.toString(16);
}

const fiveToHex: OmitThisParameter<typeof toHex> = toHex.bind(5);

console.log(fiveToHex());
```

### ThisType<Type>

这个工具不返回一个转化的类型。相反，它作为一个上下文[this]()类型标记。主要以，`--noImplicitThis`标志必须打开此岸鞥使用这个工具。

例子
```
type ObjectDescriptor<D, M> = {
  data?: D;
  methods?: M & ThisType<D & M>; // Type of 'this' in methods is D & M
};

function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {};
  let methods: object = desc.methods || {};
  return { ...data, ...methods } as D & M;
}

let obj = makeObject({
  data: { x: 0, y: 0 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx; // Strongly typed this
      this.y += dy; // Strongly typed this
    },
  },
});

obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```

在前面的例子中，给`makeObject`的`methods`对象有一个上下文类型，包含`ThisType<D & M>`，因此，`methods`对象中的方法的[this]()是`{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`。注意`methods`属性是如何是一个接口目标和`this`的源。

`ThisType<T>`标记接口是一个简单的空的接口，声明在`lib.d.ts`。超出一个对象字面量的上下文类型识别，接口表现的想任何空的接口。