泛形

大部分的软件工程师构建的组件不仅仅有定义良好和一致的 API，还能重用。能够和今天的数据工作的组件，也能和明天的数据工作，将会给你最大的弹性能力去构建大型软件系统。

在类似 C# 和 Java 的语言中，工具箱中创建可重用组件的主要工具之一是泛型，也就是，可以创建一个组件可以和一系列的类型工作而不是单一的一个。这允许用户去消费这些组件并使用他们自己的类型。

### 对泛型说好

为了开始，执行泛型的“hello world”：identity 函数。identity 函数是一个函数，将会返回传递进来的任何东西。你可以认为这个和`echo`命令很像。

没有泛型，我们将需要给定 identity 函数一个指定的类型：
```ts
function identity(arg: number): number {
  return arg;
}
```

或者，我们可以使用`any`miaoshu identity 函数：
```ts
function identity(arg: any): any {
  return arg;
}
```

尽管使用`any`是否中解决方案 ，它将会导致函数的`arg`接受任何类型，我们实际上在函数返回的时候丢失了类型的信息。如果我们传递一个数字，我们拥有的信息只有返回 any。

相反，我们需要一种方式去捕获参数的类型，在这种方式，我们可以使用它的指示返回什么。这里，我们将使用一个类型变量，一个特定类型的变量，可以在类型上工作而不是值：

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

我们现在添加了一个类型变量`T`到 idenfity 函数。这个`T`允许我们去捕获用户提供的类型（比如，`number`），这样我们可以在之后使用这些信息。这里，我们再次使用`T`作为返回类型。在检查中，我们仙子啊可以看到相同的类型被用于参数和返回类型。这允许我们在函数的一侧去运输类型信息到另一侧。

我们说这个版本的`identity`函数是泛型，因为它和一些列的类型工作。不像使用`any`，它和第一个为参数和返回值类型使用数字的`identity`函数一样精确（比如，它不丢失任何信息）。

一旦我们已经写了泛型 identity 函数，我们可以以两种方式调用。第一种方式是传递所有的参数，包括类型参数，到函数：
```ts
let output = identity<string>("myString");
//       ^ = let output: string
```
这里，我们明确的设置`T`为`string`作为函数调用的参数之一，使用`<>`包围参数表明而不是`()`。

第二种方式肯能是最常见的，我们使用类型参数推断 -- 也就是，我们想要编译器自动去为我们设置`T`的值，基于船渡的参数类型：
```ts
let output = identity("myString");
//       ^ = let output: string
```

注意我们没有明确传递类型到角括号（`<>`）；编译器只是查看`"myString"`，并设置`T`为它的类型。尽管类型参数推断可以成为一个有用的工具去保持代码简短并且更容易阅读，当编译器无法推断类型，你可能需要去明确的传递类型参数，就像我们在其哪一个例子中做的，这可能发生在更加复杂的例子。

### 使用泛型类型变量

当你使用泛型，你将会注意到，当你创建像`identity`一样的泛型函数的时候，编译器将会强制你在函数体内正确的使用任意泛型类型的参数。也就是，你实际对待这些参数，就像他们可以是任何和所有类型。

用我们前面的`identity`函数：
```ts
function identity<T>(arg: T): T {
  return arg;
}
```

如果我们想要在每次调用的时候输出参数`arg`的长度到控制台我们要怎么做？我们可能这么写：
```ts
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length);
Property 'length' does not exist on type 'T'.
  return arg;
}
```

当我们这么做的时候，编译器将会给我们一个错误，那就是我们使用`arg`的`.length`成员，但是我们没有什么地方说`arg`有这个成员。记住，我们前面说这些参数类型代替任意和所有类型，因此使用这个函数的某个人可以传递`number`，它没有`.length`成员。

也就是说其实我们实际上想要这个函数运行在`T`数组上而不是直接在`T`上。因为我们使用数组，`.length`成员应该可以被访问。我们可以描述这个，就像我们可以创建其他类型的数组：
```ts
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length);
  return arg;
}
```

你可以读取`loggingIdentity`的类型作为“泛型函数`loggingIdentity`接受一个类型参数`T`，和一个参数`arg`，他是一个`T`数组，并返回一个数组的`T`“。如果我传递数字到一个数组，我们将得到一个数字数组，`T`将会被绑定到`number`。这允许我们去使用我们泛型类型变量`T`作为我们操作的类型的一部分，而不是整个类型，给了我们很大的灵活性。

我们可以可选的以这种方式编写例子：
```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
```

你可能已经从其他语言对这种风格的类型很熟悉。在下一个章节，我们将副高你怎样创建你自己的泛型类型，比如`Array<T>`。

### 泛型类型

在前面的章节，我们创建了泛型 identity 函数，可以和很多类型一起工作。在这个章节，我们将探索函数的类型自身和怎样创建泛型接口。

泛型函数的额类型就像这些非泛型函数函数，类型参数先列出来，和函数生类似：
```ts
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

我们可以在类型上给泛型类型参数使用不同的名字，只要类型变量和使用的类型变量数量对其就行
```ts
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```
我们也可以编写泛型函数，就像调用一个对象字面量类型的前面：
```ts
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: { <T>(arg: T): T } = identity;
```

是啥引导我们编写我们的第一个泛型接口。使用前面例子中的对象字面量，并移动它到一个对象。
```ts
interface GenericIdentityFn {
  <T>(arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

在一个相似的例子，我们可能想要移动泛型参数成为整个接口的一个桉树。这让我们看到我们泛型覆盖的 type(s) 是什么（比如，`Dictionary<string>`而不只是`Dictionary`）。这让类型参数被接口的其他成员可见。
```ts
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

注意我们的例子已经变得有点不同。与其说是描述一个泛型函数，我们现在有一个无泛型函数签名，他是泛型类型的一部分。当我们使用`GenericIdentityFn`，我们腺癌将需要去指定对饮的类型信息（这里：`number`），有效的锁定底层调用前面的使用。理解什么时候直接放置类型参数在调用前面上并且什么时候放置在接口它自己将会对类型的哪些方面是泛型很有帮助。

除了泛型接口之外，我们也可以创建泛型类。注意不可能创建泛型枚举和命名空间。

### 泛型类

一个泛型类和泛型接口有相同的外型。泛型类有一个泛型类型参数列表在角括号（`<>`），它跟随在类的名字之后。
```ts
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) {
  return x + y;
};
```

这是`GenericNumber`类非常好的直接使用，但是你可能注意到没有东西限制它只能使用`number`类型。我们可以替换为`string`或者更复杂的对象。

```ts
// @strict: false
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}
// ---cut---
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) {
  return x + y;
};

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

就像接口，将类型参数放在类自身，让我们确保类所有的属性使用相同的类型。

就像我们在[我们的类章节]()提到的，一个类的类型有两端：静态端和实例端。泛型类只泛化他们的实例端，而不是静态端，当使用类，静态成员不能使用类类型参数。

### 泛型约束

如果你记得前面的例子，你可能又是想要写一个泛型函数可以和一系列你知道有什么能力的类型工作。在我们的`loggingIdentity`例子，我们想要能够访问`arg`的`.length`属性，但是编译器不能证明每一个类型有一个`.length`属性，因此它警告我们不能做这个假设。
```ts
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length);
Property 'length' does not exist on type 'T'.
  return arg;
}
```
与其使用任意和所有类型，我们更希望去约束这个函数去和有`.length`属性的任何和所有类型工作。只要这个类型有这个成员，我们就允许它，但是它需要至少有这个成员。为了做到这个，我们必须列出我们对 T 能做什么的需求。

为了做到这个，我们将创建一个接口描述我们的约束。这里，我们将创建一个接口有一个单独的`.length`属性，让后我们将使用这个接口和`extends`关键字去表示我们的约束：
```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

因为泛型函数现在受约束，它将不再能和任何和类型工作：
```ts
loggingIdentity(3);
Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.

```
相反，我们需要传递拥有所有必须属性的类型的值：
```ts
loggingIdentity({ length: 10, value: 3 });
```

### 在泛型使用类类型


当在 TypeScript 中创建工厂使用泛型，通过他们构建起函数去索引一个类类型是必须的。比如，
```ts
function create<T>(c: { new (): T }): T {
  return new c();
}
```

一个使用原型属性去推断和约束构造器函数和他的类类型实例端的关系的更高级的的例子
```ts
class BeeKeeper {
  hasMask: boolean;
}

class ZooKeeper {
  nametag: string;
}

class Animal {
  numLegs: number;
}

class Bee extends Animal {
  keeper: BeeKeeper;
}

class Lion extends Animal {
  keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```