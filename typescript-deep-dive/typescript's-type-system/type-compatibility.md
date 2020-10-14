# 类型兼容
- [类型兼容]()
- [无声]()
- [构造]()
- [泛型]()
- [可变]()
- [函数]()
    - [返回类型]()
    - [参数的数量]()
    - [可选和剩余参数]()
    - [参数类型]()
- [枚举]()
- [类]()
- [泛型]()
- [脚注：不变性]()


### 类型兼容

类型兼容性（正是我们这里讨论的）决定一个东西能否赋值给另一个。比如，`string`和`number`是否兼容：
```ts
let str: string = "Hello";
let num: number = 123;

str = num; // ERROR: `number` is not assignable to `string`
num = str; // ERROR: `string` is not assignable to `number`
```

### 无声

TypeScript 的类型系统是为了方便和允许无声的行为而设计的，比如，任何东西都可以赋值给`any`，意味着告诉编译器允许你去做你想做的：
```ts
let foo: any = 123;
foo = "Hello";

// Later
foo.toPrecision(3); // Allowed as you typed it as `any`
```

### 解构

TypeScript  对象是结构化类型。这意味着名字无所谓，只要解构匹配
```ts
interface Point {
    x: number,
    y: number
}

class Point2D {
    constructor(public x:number, public y:number){}
}

let p: Point;
// OK, because of structural typing
p = new Point2D(1,2);
```

这允许你去自有创建对象（）并依旧安全，只要它可以被推断。

当然太多数据也被认为是安全的：
```ts
interface Point2D {
    x: number;
    y: number;
}
interface Point3D {
    x: number;
    y: number;
    z: number;
}
var point2D: Point2D = { x: 0, y: 10 }
var point3D: Point3D = { x: 0, y: 10, z: 20 }
function iTakePoint2D(point: Point2D) { /* do something */ }

iTakePoint2D(point2D); // exact match okay
iTakePoint2D(point3D); // extra information okay
iTakePoint2D({ x: 0 }); // Error: missing information `y`
```

### 可变

可变是类型兼容分析的一个简单理解并且重要的概念。

对于简单类型`Base`和`Child`，如果`Child`是`Base`的子孙，`Child`的实例可以被赋值给类型是`Base`的变量。

> 这是多态 101

在`Base`和`Child`构成的类型兼容的复杂类型取决于`Base`和`Child`在类似场景中取决于差异。

- 协变：（）只在相同的方向
- 异变：（）只在不透明的方向
- 双变：正面和背面
- 不变：如果类型不是完全相同，他们就不兼容

> 注意：对于一个完全兼容的类型系统，存在类似 JavaScript 

### 函数

当对比两个函数的时候，有一些不易察觉的东西需要考虑。

#### 返回类型

`协变`：返回值必须至少包含足够的数据。
```ts
/** Type Hierarchy */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Two sample functions */
let iMakePoint2D = (): Point2D => ({ x: 0, y: 0 });
let iMakePoint3D = (): Point3D => ({ x: 0, y: 0, z: 0 });

/** Assignment */
iMakePoint2D = iMakePoint3D; // Okay
iMakePoint3D = iMakePoint2D; // ERROR: Point2D is not assignable to Point3D
```

#### 参数数量

更少的参数是可以的（比如，函数可以选择去忽略额外参数）。只要保证至少可以调用的足够的参数。

```ts
let iTakeSomethingAndPassItAnErr
    = (x: (err: Error, data: any) => void) => { /* do something */ };

iTakeSomethingAndPassItAnErr(() => null) // Okay
iTakeSomethingAndPassItAnErr((err) => null) // Okay
iTakeSomethingAndPassItAnErr((err, data) => null) // Okay

// ERROR: Argument of type '(err: any, data: any, more: any) => null' is not assignable to parameter of type '(err: Error, data: any) => void'.
iTakeSomethingAndPassItAnErr((err, data, more) => null);
```

#### 可选和剩余参数

可选的（预决定的数量）和剩余参数（任意参数数量）是兼容的，为了便利。
```ts
let foo = (x:number, y: number) => { /* do something */ }
let bar = (x?:number, y?: number) => { /* do something */ }
let bas = (...args: number[]) => { /* do something */ }

foo = bar = bas;
bas = bar = foo;
```

> 注意：可选的（在我们的例子中是`bar`）和非可选的（在我们的例子中是`foo`）只有在`strictNullChecks`是 false 的时候

#### 参数的类型

`双变`：这设计用于支持常见事件处理场景
```ts
/** Event Hierarchy */
interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

/** Sample event listener */
enum EventType { Mouse, Keyboard }
function addEventListener(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common. Works as function argument comparison is bivariant
addEventListener(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Undesirable alternatives in presence of soundness
addEventListener(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + "," + (<MouseEvent>e).y));
addEventListener(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + "," + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
addEventListener(EventType.Mouse, (e: number) => console.log(e));
```

当然`Array<Child>`是可以赋值给`Array<Base>`（协变），因为函数是兼容的。数组协变需要所有的`Array<Child>`函数都能赋值给`Array<Base>`，比如`push(t:Child)`可以赋值给`push(t:Base)`，通过函数参数协变可以做到。

**这对于从其他语言来的人很难理解**，他们期待下面报错，但是在 TypeScript 却不会：
```ts
/** Type Hierarchy */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Two sample functions */
let iTakePoint2D = (point: Point2D) => { /* do something */ }
let iTakePoint3D = (point: Point3D) => { /* do something */ }

iTakePoint3D = iTakePoint2D; // Okay : Reasonable
iTakePoint2D = iTakePoint3D; // Okay : WHAT
```

### 枚举

- 枚举和数字是兼容的，数字和枚举是兼容的。
```ts
enum Status { Ready, Waiting };

let status = Status.Ready;
let num = 0;

status = num; // OKAY
num = status; // OKAY
```

- 不同枚举类型的枚举值被认为是不兼容的。这让枚举正常可用（而不死结构上）
```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
let color = Color.Red;

status = color; // ERROR
```

### 类

- 只有实例成员和方法才被比较。构造器和静态没有作用。
```ts
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { /** do something */ }
}

class Size {
    feet: number;
    constructor(meters: number) { /** do something */ }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```

- `private`和`protected`成员必须从相同的类。这类成员的建立让类正常。
```ts
/** A class hierarchy */
class Animal { protected feet: number; }
class Cat extends Animal { }

let animal: Animal;
let cat: Cat;

animal = cat; // OKAY
cat = animal; // OKAY

/** Looks just like Animal */
class Size { protected feet: number; }

let size: Size;

animal = size; // ERROR
size = animal; // ERROR
```

### 泛型

因为 TypeScript 有一个结构化的类型系统，类型参数只影响兼容性，当被一个成员使用。比如，下面`T`不影响兼容。
```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x
```

然而，如果`T`被使用，他将在兼容性中扮演一个角色，基于它的实例，正如下面显示：
```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x and y are not compatible
```

为了放置泛型参数没有被实例化，他们被`any`替代，在检查兼容性之前：
```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
```

泛型调用类命中，通过关联类兼容性，就像之前提到的。比如
```ts
class List<T> {
  add(val: T) { }
}

class Animal { name: string; }
class Cat extends Animal { meow() { } }

const animals = new List<Animal>();
animals.add(new Animal()); // Okay 
animals.add(new Cat()); // Okay 

const cats = new List<Cat>();
cats.add(new Animal()); // Error 
cats.add(new Cat()); // Okay
```


### 脚注：不可变

我们说不可变是唯一可见的选项。这是一个例子，`contra`和`co`可变显示为不安全的数组：
```ts
/** Hierarchy */
class Animal { constructor(public name: string){} }
class Cat extends Animal { meow() { } }

/** An item of each */
var animal = new Animal("animal");
var cat = new Cat("cat");

/**
 * Demo : polymorphism 101
 * Animal <= Cat
 */
animal = cat; // Okay
cat = animal; // ERROR: cat extends animal

/** Array of each to demonstrate variance */
let animalArr: Animal[] = [animal];
let catArr: Cat[] = [cat];

/**
 * Obviously Bad : Contravariance
 * Animal <= Cat
 * Animal[] >= Cat[]
 */
catArr = animalArr; // Okay if contravariant
catArr[0].meow(); // Allowed but BANG 🔫 at runtime


/**
 * Also Bad : covariance
 * Animal <= Cat
 * Animal[] <= Cat[]
 */
animalArr = catArr; // Okay if covariant
animalArr.push(new Animal('another animal')); // Just pushed an animal into catArr!
catArr.forEach(c => c.meow()); // Allowed but BANG 🔫 at runtime
```