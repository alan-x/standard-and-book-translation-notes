高级类型

这个页面列出了一些你可以建模类型更高级的方式，他和工具类型文档协同，包含包含在 TypeScript 并且全局可用的类型。

### 类型守卫和类型区分

值在可以采用的类型可以交叠的时候，联合类型在这种建模场景非常有用。当我们需要明确知道我们是否有一个`Fish`的时候会发生什么？在 JavaScript 中，区分两个可能的值的一个常见的方式是检查成员是否存在。正如我们提到的，你只可以访问保证在联合类型所有的组成都存在的成员。

```ts
let pet = getSmallPet();

// You can use the 'in' operator to check
if ("swim" in pet) {
  pet.swim();
}
// However, you cannot use property access
if (pet.fly) {
Property 'fly' does not exist on type 'Fish | Bird'.
  Property 'fly' does not exist on type 'Fish'.
  pet.fly();
Property 'fly' does not exist on type 'Fish | Bird'.
  Property 'fly' does not exist on type 'Fish'.
}
```

为了让相同的代码可以通过属性访问器工作，我们将需要使用类型判定：
```
let pet = getSmallPet();
let fishPet = pet as Fish;
let birdPet = pet as Bird;

if (fishPet.swim) {
  fishPet.swim();
} else if (birdPet.fly) {
  birdPet.fly();
}
```

然而这不是你想要在你的代码库出现的代码。

### 用户定义的类型守卫

如果一旦我们执行了检测，我们可以在每一个分支知道`pet`的类型会更好。

正好 TypeScript 有一个叫做类型守卫的东西。一个类型守卫使一些表达式，执行一个运行时检查，确保类型在一些范围。

### 使用类型判定

为了定一个类型守卫，我们只需要去定一个函数，他的返回值是一个类型判定：
```
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```
在这个例子中，`pet is Fish`是我们的类型判定。一个类型判定采用`parameterName is Type`的形式，`parameterName`必须是当前函数签名的一个参数名字。

任何时候，`isFish`和一些变量被调用的时候，TypeScript 将会向下转型变量到指定类型，如果原始类型是兼容的。
```
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

注意 TypeScript 不仅仅知道在`if`分支中，`pet`是一个`Fish`；它也知道在`else`分支中，你不需要一个`Fish`，因此你必须是一个`Bord`。

### 使用 in 操作符

`in`操作符也表现的像一个类型向下转型表达式。

对于一个`n in x`表达式，`n`是一个字符串字面量或者字符串字面量类型并且`x`是一个联合类型，“true”分支向下转型为有一个可选的或者必须属性`n`的类型，“false”分支向下转型为有一个可选的或者没有属性`n`的类型。
```
function move(pet: Fish | Bird) {
  if ("swim" in pet) {
    return pet.swim();
  }
  return pet.fly();
}
```

### typeof 类型守卫

现在回去编写一个`padLeft`使用联合类型的代码。我们可以如下编写类型判定：
```
function isNumber(x: any): x is number {
  return typeof x === "number";
}

function isString(x: any): x is string {
  return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
  if (isNumber(padding)) {
    return Array(padding + 1).join(" ") + value;
  }
  if (isString(padding)) {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

然而，必须去定一个函数去指出一个类型是原子类型是一种痛。幸运的是，你不需要去抽象`typeof x === "number"`到它自己的函数，因为 TypeScript 将会意识到它是一个类型守卫。这意味着我们可以编写行内检查：
```
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

typeof 类型守卫认识两种不同的形式：`typeof v ==="typename"`和`typeof v !== "typename"`，`"typename"`必须是`"number"`，`"string"`，`"boolean"`，或者`"symbol"`。然而 TypeScript 不会阻止你去和其他字符串比较，语言不会认为这些表达式是类型守卫。

### instanceof 类型守卫

如果你已经阅读了`typeof`类型守卫，并且对 JavaScript 中的`instanceof`操作符很熟悉，你可能对这个章节的内容有些想法。

instanceof 类型守卫是一个使用他们构造器函数向下转型的方式。比如，让我们借用之前工业实力串线机的例子：

```
interface Padder {
  getPaddingString(): string;
}

class SpaceRepeatingPadder implements Padder {
  constructor(private numSpaces: number) {}
  getPaddingString() {
    return Array(this.numSpaces + 1).join(" ");
  }
}

class StringPadder implements Padder {
  constructor(private value: string) {}
  getPaddingString() {
    return this.value;
  }
}

function getRandomPadder() {
  return Math.random() < 0.5
    ? new SpaceRepeatingPadder(4)
    : new StringPadder("  ");
}

let padder: Padder = getRandomPadder();
//       ^ = let padder: Padder

if (padder instanceof SpaceRepeatingPadder) {
  padder;
  //     ^?
}
if (padder instanceof StringPadder) {
  padder;
  //     ^?
}
```

`instanceof`的右侧需要是一个构造器函数，TypeScript 将会向下转型：
1. 函数的`property`的类型，如果这个类型不是`any`
2. 类型的构造签名返回的联合类型

按这个顺序。


### 可空类型

TypeScript 有两个特殊的类型，`null`和`undefined`，他们各自有对应的值 null 和 undefined。我们在[基本类型章节]()有稍微提到。

默认情况下，类型检测认为`null`和`undefined`可以赋值给任何东西。实际上，`null`和`undefined`是每种类型的有效值。这意味着不可能组阻止他们被赋值于任何类型，就算当你想要阻止它。`null`的发明家，Tony Hoare，称这个为他的“[百万美金失误]()”。

[--srictNullChecks]()标志修复了这个：当你声明一个变量，它不自动包含`null`或者`undefined`。你可以明确的使用一个联合类型包含他们：
```ts
let examapleString = "foo";
examapleString = null;
Type 'null' is not assignable to type 'string'.

let stringOrNull: string | null = "bar";
stringOrNull = null;

stringOrNull = undefined;
Type 'undefined' is not assignable to type 'string | null'.
```

注意，为了匹配 JavaScript 语义，TypeScript 对待`null`和`undefined`不同。`string | null`和`string | undefined` 和`string | undefined | null` 不同。

从 TypeScript 3.7 开始，你可以使用[可选链]()简化可空类型的使用。

### 可选参数和属性

使用`--strictNullChecks`，一个可选的参数自动添加`| undefined`：
```
function f(x: number, y?: number) {
  return x + (y || 0);
}

f(1, 2);
f(1);
f(1, undefined);
f(1, null);
Argument of type 'null' is not assignable to parameter of type 'number | undefined'.
```

对于可选属性也是一样的：
```
class C {
  a: number;
  b?: number;
}

let c = new C();

c.a = 12;
c.a = undefined;
Type 'undefined' is not assignable to type 'number'.
c.b = 13;
c.b = undefined;
c.b = null;
Type 'null' is not assignable to type 'number | undefined'.
```

### 类型守卫和类型断言

因为可空类型使用联合实现，你需要使用类型守卫去摆脱`null`。幸运的是，这和你在 JavaScript 中写的代码一样：
```
function f(stringOrNull: string | null): string {
  if (stringOrNull === null) {
    return "default";
  } else {
    return stringOrNull;
  }
}
```

这里`null`的消除十分明显，但是你可以使用更简单的操作符：
```
function f(stringOrNull: string | null): string {
  return stringOrNull || "default";
}
```
在某些场景下，编译器无法消除`null`或者`undefined`，你可以使用类型断言操作符去手动移除他们。语法是后缀`!`：`identifier!`从`identifier`移除`null`和`undefined`：
```
interface UserAccount {
  id: number;
  email?: string;
}

const user = getUser("admin");
user.id;
Object is possibly 'undefined'.

if (user) {
  user.email.length;
Object is possibly 'undefined'.
}

// Instead if you are sure that these objects or fields exist, the
// postfix ! lets you short circuit the nullability
user!.email!.length;
```

### 类型别名

类型别名为类型创建一个新的名字。类型别名有时候和接口很像，但是可以明明原子，联合，元组，和任何其他不怎么做你就得手写的类型。

```
type Second = number;

let timeInSecond: number = 10;
let time: Second = 10;
```

别名不真实创建一个新的类型 - 它创建一个名字去引用类型。别名一个原语不是非常有用，尽管它可以用做文档的一种形式。

就像接口，类型别名可以泛型 -- 我们可以添加类型参数并在别名声明右侧使用他们：
```
type Container<T> = { value: T };
```

我们也可以使用一个类型别名在属性中去引用自身：
```
type Tree<T> = {
  value: T;
  left?: Tree<T>;
  right?: Tree<T>;
};
```

和[交叉类型]()一起，我们可以创建一些十分古怪的类型：
```ts
type LinkedList<Type> = Type & { next: LinkedList<Type> };

interface Person {
  name: string;
}

let people = getDriversLicenseQueue();
people.name;
people.next.name;
people.next.next.name;
people.next.next.next.name;
//                  ^ = (property) next: LinkedList
```

### 接口 vs 类型别名

正如我们提到的，类型别名可以表现的类似接口；然而，他们有一点不同。

几乎一个`interface`所有的特性都可以在`type`中可用，关键不同是一个类型不能被重新打开去添加新的属性，而接口总是可扩展的。

接口

扩展一个接口

```
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```

通过交叉扩展一个类型：
```
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: Boolean 
}

const bear = getBear();
bear.name;
bear.honey;
```

添加一个新的域到一个存在的接口：
```ts
interface Window {
  title: string
}

interface Window {
  ts: import("typescript")
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});

```
一个类型被创建之后不能被改变。
```ts
type Window = {
  title: string
}

type Window = {
  ts: import("typescript")
}

// Error: Duplicate identifier 'Window'.


```
因为一个接口更接近于 JavaScript 对象[通过开放成为可扩展]()，我们推荐尽可能通过类型别名使用接口。

换句话说，如果你无法使用接口表达一个外型，并且你需要去使用一个联合或者元组类型，类型别名通常是实现的方式。

### 枚举成员类型

就像在[我们关于枚举的章节]()提到的，当每一个成员是字面量-初始化的，枚举成员有类型。

很多时候，当我们讨论“单例类型”的时候，我们说的是枚举成员类型和数字/字符串字面量类型，尽管很多用户将使用“单例类型”和“字面量类型”交换。

### 多态的 this 类型

一个多态的`this`类型表示一个类型，它是包含类或者接口的子类型。这叫做 F-bounded 多态性，很多人知道它是因为[fluent API]()模式。这使得分层连贯接口更容易去表达，比如。采取一个简单的计算，在每个操作之后返回`this`：
```ts
class BasicCalculator {
  public constructor(protected value: number = 0) {}
  public currentValue(): number {
    return this.value;
  }
  public add(operand: number): this {
    this.value += operand;
    return this;
  }
  public multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
  // ... other operations go here ...
}

let v = new BasicCalculator(2).multiply(5).add(1).currentValue();
```

因为类使用`this`类型，你可以扩展它，新的类可以水用旧的方法而不需要改变。

```ts
class ScientificCalculator extends BasicCalculator {
  public constructor(value = 0) {
    super(value);
  }
  public sin() {
    this.value = Math.sin(this.value);
    return this;
  }
  // ... other operations go here ...
}

let v = new ScientificCalculator(2).multiply(5).sin().add(1).currentValue();
```
没有`this`类型，`ScientificCalculator`无法扩展`BasicCalculator`并保持 fluent 接口。`multiply`将会返回`BasicCalculator`，没有`sin`方法。然而，使用`this`类型，`multiply`返回`this`，这里是`ScientificCalculator`。

### 索引类型

使用索引类型，你可以获得编译器去检测使用动态属性名的代码。比如，一个常见 JavaScript 模式是去对象中获取子集属性：
```
function pluck(o, propertyNames) {
  return propertyNames.map((n) => o[n]);
}
```

这是你在 TypeScript 中编写和使用这个函数的方式，使用索引类型查询和索引的访问操作符：
```ts
function pluck<T, K extends keyof T>(o: T, propertyNames: K[]): T[K][] {
  return propertyNames.map((n) => o[n]);
}

interface Car {
  manufacturer: string;
  model: string;
  year: number;
}

let taxi: Car = {
  manufacturer: "Toyota",
  model: "Camry",
  year: 2014,
};

// Manufacturer and model are both of type string,
// so we can pluck them both into a typed string array
let makeAndModel: string[] = pluck(taxi, ["manufacturer", "model"]);

// If we try to pluck model and year, we get an
// array of a union type: (string | number)[]
let modelYear = pluck(taxi, ["model", "year"]);
```

编译器检查`manufacturer`和`model`是否真实存在在`Car`。这个例子引入了一堆新的类型操作符。首先是`keyof T`，索引类型查询操作符。对于任何类型`T`，`keyof T`是未知的联合，叫做`T`的公共属性。比如：
```ts
let carProps: keyof Car;
//         ^ = let carProps: "manufacturer" | "model" | "year"
```

`keyof Car` 完全可以和`"manufacturer" | "model" | "year"`交换。不同的是，如果你添加其他属性到`Car`，叫做`ownersAddress: string`，则`keyof Car`将会自动更新为`"manufacturer" | "model" | "year" | "ownersAddress"`。你可以使用`keyof`在泛型上下文，比如`pluck`，你不可能知道提前知道属性名字。这意味着编译器将会检查你传递正确的属性名集合给`pluck`：
```
// error, Type '"unknown"' is not assignable to type '"manufacturer" | "model" | "year"'
pluck(taxi, ["year", "unknown"]);
```

第二个操作符是`T[K]`，索引的访问操作符。这里，类型语法反射表达式语法。这意味着`taxi["manufacturer"]`有类型`Car["manufacturer"]` -- 在我们的例子中只是`string`。然而，就像索引类型查询，你可以在泛型上下文使用`T[K]`，这使它的真正威力降临。你只需要确保类型变量` K extends keyof T`。这是另一个例子，他的函数名字是`getProperty`：
```ts
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
  return o[propertyName]; // o[propertyName] is of type T[K]
}
```

在`getProperty`，`o: T`和`propertyName: K`，也意味着`o[propertyName]: T[K]`。一旦i返回`T[K]`结果，编译器将会实例化键的真实类型，因此`getProperty`的返回类型根据你请求的属性的不同有所变化。

```
let manufacturer: string = getProperty(taxi, "manufacturer");
let year: number = getProperty(taxi, "year");

let unknown = getProperty(taxi, "unknown");
Argument of type '"unknown"' is not assignable to parameter of type '"manufacturer" | "model" | "year"'.
```


索引类型和索引签名

`keyof`和`T[K]`和索引签名交互。一个索引签名参数类型必须是‘string’或者‘number’。如果你有一个类型，他的索引签名是字符串，`keyof T`将会是`string | number`（不只是 string，因为 JavaScript ，你可以使用字符串（`object["42"]`或者数字（`object[42]`）访问一的对象属性））。`T[string]`是索引签名的类型：
```
interface Dictionary<T> {
  [key: string]: T;
}
let keys: keyof Dictionary<number>;
//     ^ = let keys: string | number
let value: Dictionary<number>["foo"];
//      ^ = let value: number
```

如果你有一个类型，有一个数字索引签名，`keyof T`将会是`number`。

```ts
interface Dictionary<T> {
  [key: number]: T;
}

let keys: keyof Dictionary<number>;
//     ^ = let keys: number
let numberValue: Dictionary<number>[42];
//     ^ = let numberValue: number
let value: Dictionary<number>["foo"];
Property 'foo' does not exist on type 'Dictionary<number>'.

```

### 映射类型

一个常见任务使用让一个已存在的类型，并让每一个属性可选：
```
interface PersonSubset {
  name?: string;
  age?: number;
}
```

或者我们想要只读版本：
```
interface PersonReadonly {
  readonly name: string;
  readonly age: number;
}

```

这在 JavaScript 中太常见，TypeScript 提供了一个方式去基于旧的类型创建新的类型 -- 映射类型。在一个映射类型，新的类型以相同的方式转化旧的类型的每一个属性。比如，你可以让所有的额属性可选或者只读。这是一组例子：
```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

使用它：
```
type PersonPartial = Partial<Person>;
//   ^ = type PersonPartial = {
    name?: string | undefined;
    age?: number | undefined;
}
type ReadonlyPerson = Readonly<Person>;
//   ^ = type ReadonlyPerson = {
    readonly name: string;
    readonly age: number;
}
```

注意这个语法描述了一个类型而不是一个成员。如果你想要添加成员，你可以使用一个交叉类型：
```ts
// Use this:
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean }

// This is an error!
type WrongPartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
  newMember: boolean;
'boolean' only refers to a type, but is being used as a value here.
'}' expected.
}
Declaration or statement expected.
```
看看最简单的映射类型和他的部分：
```
type Keys = "option1" | "option2";
type Flags = { [K in Keys]: boolean };
```

语法和`for .. in`内部的索引签名语法很像。有三个部分：
1. 类型变量`K`，哪个依次绑定到每个属性
2. 字符串字面量联合`Keys`，包含要遍历的属性名
3. 属性的结果类型。

在这个简单的例子中，`Keys`是一个硬编码属性名列表，属性类型总是`boolean`，因此这个映射类型和下面写的相同：
```
type Flags = {
  option1: boolean;
  option2: boolean;
};
```

然而，真实的应用，就像前面的`Readonly`，或者`Partial`。他们基于一些存在的类型，他们以某种方式转化属性。这也是`keyof`和索引访问类型出现的地方：
```
type NullablePerson = { [P in keyof Person]: Person[P] | null };
//   ^ = type NullablePerson = {
    name: string | null;
    age: number | null;
}
type PartialPerson = { [P in keyof Person]?: Person[P] };
//   ^ = type PartialPerson = {
    name?: string | undefined;
    age?: number | undefined;
}
```

但是有一个通用版本会更好：
```
type Nullable<T> = { [P in keyof T]: T[P] | null };
type Partial<T> = { [P in keyof T]?: T[P] };
```

在这个例子，属性列表是`keyof T`，结果类型是`T[P]`的一些变形。这对于任何常用的映射类型是一个好的模板。这是因为这类转化是[同型]()的，这意味着映射只应用`T`的属性，没有其他的。编译器知道他可以赋值所有存在的属性修饰符，在添加新的一个之前。比如，如果`Person.name`是只读的，`Partial<Person>.name`将会只读和可选。


这是另一个例子，`T[P]`映射到一个`Proxy<T>`类：
```ts
type Proxy<T> = {
  get(): T;
  set(value: T): void;
};

type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>;
};

function proxify<T>(o: T): Proxify<T> {
  // ... wrap proxies ...
}

let props = { rooms: 4 };
let proxyProps = proxify(props);
//  ^ = let proxyProps: Proxify<{
    rooms: number;
}>
```

注意`Readonly<T>`和`Partial<T>`非常有用，他们和`Pick`和`Record`一起包含在 TypeScript 标准库：
```
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Record<K extends keyof any, T> = {
  [P in K]: T;
};

```

`Readonly`，`Partial`，和`Pick`是同型的，然而，`Record`不是。`Record`不是同型的一个线索是它不接受一个输入类型去赋值属性：
```
type ThreeStringProps = Record<"prop1" | "prop2" | "prop3", string>;
```

非同型类型本质上创建一个新的属性。因此他们无法从其他地方赋值这个属性修饰符。


### 从映射类型推断

现在，你知道怎样去包裹类型的属性，下一个东西你想要做的东西是不包裹他们，幸运的是，这很简单：
```ts
function unproxify<T>(t: Proxify<T>): T {
  let result = {} as T;
  for (const k in t) {
    result[k] = t[k].get();
  }
  return result;
}

let originalProps = unproxify(proxyProps);
//  ^ = let originalProps: {
    rooms: number;
}
```

注意这个展开推断值工作于同型映射类型。如果映射类型不是同型的，你需要给一个明确的类型参数到你的展开函数。

### 条件类型
一个条件类型选择两个可能的类型中的一个，基于一个条件表达式的一个类型关系测试：
```
T extends U ? X : Y
```

前面的类型意味着，当`T`可以赋值给`U`，类型就是`X`，否则就是`Y`。

一个条件类型`T extends U ? X : Y`要么是`X`或`Y`，或者延迟，因此条件依赖于一个或者多个类型变量。当`T`或`U`包含类型变量，是否解析为`X`或`Y`，或者延迟，取决于类型系统有足够的信息去推断`T`是否可以赋值给`U`。

作为一个一些类型立即解析的例子，我们可以看看下面的例子：
```ts
declare function f<T extends boolean>(x: T): T extends true ? string : number;

// Type is 'string | number'
let x = f(Math.random() < 0.5);
//  ^ = let x: string | number
```

另一个例子是`TypeName`类型别名，它使用潜逃的条件类型：
```ts
type TypeName<T> = T extends string
  ? "string"
  : T extends number
  ? "number"
  : T extends boolean
  ? "boolean"
  : T extends undefined
  ? "undefined"
  : T extends Function
  ? "function"
  : "object";

type T0 = TypeName<string>;
//   ^ = type T0 = "string"
type T1 = TypeName<"a">;
//   ^ = type T1 = "string"
type T2 = TypeName<true>;
//   ^ = type T2 = "boolean"
type T3 = TypeName<() => void>;
//   ^ = type T3 = "function"
type T4 = TypeName<string[]>;
//   ^ = type T4 = "object"
```

作为一个条件类型被延迟的例子 - 他们在附近徘徊而不是选择一个分支 - 如下：
```
interface Foo {
  propA: boolean;
  propB: boolean;
}

declare function f<T>(x: T): T extends Foo ? string : number;

function foo<U>(x: U) {
  // Has type 'U extends Foo ? string : number'
  let a = f(x);

  // This assignment is allowed though!
  let b: string | number = a;
}
```

前面，变量`a`有一个条件变量还没选择一个分支。当其他块的代码调用`foo`结束，他会使用一些其他类型替代`U`，TypeScript 将会重新运行条件类型，决定是否它可以真的选择一个分支。

于此同时，我们可以指定一个田间类型去其他目标类型，只要每一个条件分支可以赋值给目标。因此，在我们前面的例子，我们可以赋值`U extends Foo ? string : number`给`string | number`，因为无聊条件如何求值，它都知道`string`或者`number`。

### 分发条件类型

选中类型是外露类型参数的条件类型被叫做分发条件类型。分发条件类型是在实例化的时候自动分发到联合类型。比如，一个使用`A | B | C`类型参数的`T extends U ? X : Y`的实例被解析为`(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`。

例子
```
type T5 = TypeName<string | (() => void)>;
//   ^ = type T5 = "string" | "function"
type T6 = TypeName<string | string[] | undefined>;
//   ^ = type T6 = "string" | "undefined" | "object"
type T7 = TypeName<string[] | number[]>;
//   ^ = type T7 = "object"
```
在分发条件类型`T extends U ? X : Y`的实例化中，条件类型中对`T`的引用被解析为联合类型中的独立的一个（比如，`T`索引独立场景在条件类型被分发到一个联合类型之后）。幸运的是，`X`中对`T`的引用有一个额外的参数类型约束`U`（比如，`T`在`X`内被认为可以赋值给`U`）。

例子
```
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T1 = Boxed<string>;
//   ^ = type T1 = {
    value: string;
}
type T2 = Boxed<number[]>;
//   ^ = type T2 = {
    array: number[];
}
type T3 = Boxed<string | number[]>;
//   ^ = type T3 = BoxedValue | BoxedArray
```

注意，在`Boxed<T>`的选中分支，`T`有一个额外的约束`any[]`，因此，它可能索引数组的元素类型为`T[number]`。同时，也注意最后一个例子条件类型怎样分发到联合类型。

条件类型的分发属性可以方便的用于联合类型过滤：
```
// Remove types from T that are assignable to U
type Diff<T, U> = T extends U ? never : T;
// Remove types from T that are not assignable to U
type Filter<T, U> = T extends U ? T : never;

type T1 = Diff<"a" | "b" | "c" | "d", "a" | "c" | "f">;
//   ^ = type T1 = "b" | "d"
type T2 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "a" | "c"
//   ^ = type T2 = "a" | "c"
type T3 = Diff<string | number | (() => void), Function>; // string | number
//   ^ = type T3 = string | number
type T4 = Filter<string | number | (() => void), Function>; // () => void
//   ^ = type T4 = () => void

// Remove null and undefined from T
type NotNullable<T> = Diff<T, null | undefined>;

type T5 = NotNullable<string | number | undefined>;
//   ^ = type T5 = string | number
type T6 = NotNullable<string | string[] | null | undefined>;
//   ^ = type T6 = string | string[]

function f1<T>(x: T, y: NotNullable<T>) {
  x = y;
  y = x;
Type 'T' is not assignable to type 'Diff<T, null | undefined>'.
}

function f2<T extends string | undefined>(x: T, y: NotNullable<T>) {
  x = y;
  y = x;
Type 'T' is not assignable to type 'Diff<T, null | undefined>'.
  Type 'string | undefined' is not assignable to type 'Diff<T, null | undefined>'.
    Type 'undefined' is not assignable to type 'Diff<T, null | undefined>'.
  let s1: string = x;
Type 'T' is not assignable to type 'string'.
  Type 'string | undefined' is not assignable to type 'string'.
    Type 'undefined' is not assignable to type 'string'.
  let s2: string = y;
}
```

条件类型特别有用，当和映射类型绑定：
```ts
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Part {
  id: number;
  name: string;
  subparts: Part[];
  updatePart(newName: string): void;
}

type T1 = FunctionPropertyNames<Part>;
//   ^ = type T1 = "updatePart"
type T2 = NonFunctionPropertyNames<Part>;
//   ^ = type T2 = "id" | "name" | "subparts"
type T3 = FunctionProperties<Part>;
//   ^ = type T3 = {
    updatePart: (newName: string) => void;
}
type T4 = NonFunctionProperties<Part>;
//   ^ = type T4 = {
    id: number;
    name: string;
    subparts: Part[];
}
```

和联合类型和交叉类型相似，条件类型不允许递归引用他们。比如下面的例子是一个错误。
例子：
```
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T; // Error
Type alias 'ElementType' circularly references itself.
Type 'ElementType' is not generic.
```


### 条件类型中的类型推断

在条件类型的`extends`语句中，现在可以有一个`infer`声明，引入一个类型变量去推断。这类推断的类型变量可能在条件类型的选中分支索引。可以有多个`infer`定位相同的类型变量。

比如，下面解析了一个函数类型解析返回值：
```
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

条件类型可以嵌套去组成一系列的模式匹配，按序求值：
```
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
  ? U
  : T extends Promise<infer U>
  ? U
  : T;

type T0 = Unpacked<string>;
//   ^ = type T0 = string
type T1 = Unpacked<string[]>;
//   ^ = type T1 = string
type T2 = Unpacked<() => string>;
//   ^ = type T2 = string
type T3 = Unpacked<Promise<string>>;
//   ^ = type T3 = string
type T4 = Unpacked<Promise<string>[]>;
//   ^ = type T4 = Promise
type T5 = Unpacked<Unpacked<Promise<string>[]>>;
//   ^ = type T5 = string
```

下面的例子表明对于相同类型变量有多少候选人在同型未知导致一个联合类型被推断：
```
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;

type T1 = Foo<{ a: string; b: string }>;
//   ^ = type T1 = string
type T2 = Foo<{ a: string; b: number }>;
//   ^ = type T2 = string | number
```

同样，相同类型变量的多个候选在逆反未知导致一个交叉类型被索引：
```
type Bar<T> = T extends { a: (x: infer U) => void; b: (x: infer U) => void }
  ? U
  : never;

type T1 = Bar<{ a: (x: string) => void; b: (x: string) => void }>;
//   ^ = type T1 = string
type T2 = Bar<{ a: (x: string) => void; b: (x: number) => void }>;
//   ^ = type T2 = never
```

当从有多个调用签名的类型推断的时候（），接口根据最后一个签名推断（）。不可能去执行重载解决，基于参数类型列表。
```
declare function foo(x: string): number;
declare function foo(x: number): string;
declare function foo(x: string | number): string | number;

type T1 = ReturnType<typeof foo>;
//   ^ = type T1 = string | number
```

为常规类型参数在约束语句使用`infer`声明是不可能的：
```
type ReturnedType<T extends (...args: any[]) => infer R> = R;
'infer' declarations are only permitted in the 'extends' clause of a conditional type.
Cannot find name 'R'.
```

然而，可以得到相同的效果，通过在约束中去掉类型变量并指定一个条件类型替代：
```
type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R
  ? R
  : any;
```

预定义条件类型

TypeScript 添加多个预定义条件类型，你可以在[工具类型]()找到完整列表和例子。