Classes

传统 JavaScript 使用函数和原型 -- 基于继承去构建重用组件，但是这对于习惯于面向对象的实现的程序员来说可能有点笨拙，类继承功能和对象是从这些类构建的。从 ECMAScript 2015 开始，熟知的 ECMAScript 6，JavaScript 程序员可以使用面向对象基于类的是想方式去构建他们的应用。在 TypeScript，为我们允许开发者使用这个技术，并编译他们到 JavaScript，造大部分主流浏览器和平台使用，不需要等待下一个版本的 JavaScript。

### 类

来看看简单的基于类的例子：

```
class Greeter {
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter = new Greeter("world");
```

如果你使用过 C# 或者 Java，这语法你可能看起来很熟悉。我们声明一个新的类`Greeter`。这个类有三个成员：一个属性叫做`greeting`，一个构造器，和一个方法`greet`。这个类有三个成员：一个属性叫做


你将会主要到在类中，当我们引用类的任何成员，我们前面添加了一个`this.`。这表明这是一个成员访问。

在最后一行，我们使用`new`构建了一个`Greeter`类的实例。这调用了我们前面定义的构造器，创建了一个有`Greeter`形状的新对象，并运行构造器去初始化它。

### 继承

在 TypeScript，我们可以使用普通的面向对象模式。在基于对象编程中，其中一个最基础的模式是可以使用继承去扩展存在的类去创建新的一个。

看一个例子：

```
class Animal {
  move(distanceInMeters: number = 0) {
    console.log(`Animal moved ${distanceInMeters}m.`);
  }
}

class Dog extends Ani=
mal {
  bark() {
    console.log("Woof! Woof!");
  }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

这个例子展示了最基本的继承特性：类从基本的类继承属性和方法。这里，`Dog`是一个衍生类，来自`Animal`基础类，使用`extends`关键字。衍生类通常叫做子类，基础类一般叫做父类。

因为`Dog`继承`Animal`的功能，我们可以去创建一个`Dog`实例，拥有`bark()`和`move()`。这个类有三个成员：一个属性叫做

来看看一个更复杂的例子

```
class Animal {
  name: string;
  constructor(theName: string) {
    this.name = theName;
  }
  move(distanceInMeters: number = 0) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}

class Snake extends Animal {
  constructor(name: string) {
    super(name);
  }
  move(distanceInMeters = 5) {
    console.log("Slithering...");
    super.move(distanceInMeters);
  }
}

class Horse extends Animal {
  constructor(name: string) {
    super(name);
  }
  move(distanceInMeters = 45) {
    console.log("Galloping...");
    super.move(distanceInMeters);
  }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

这个例子覆盖了一些其他我们之前没有意识到的特性。我们再一次看到`extends`官架子，用来创建`Animal`的连个新的子类：`Horse`和`Snake`。这个类有三个成员：一个属性叫做

和前一个例子不同的是每一个衍生类包含的构造器函数必须调用`super()`，它将会执行基类的构造器。甚至，在构造器中，我们访问`this`的任何成员之前，都必须调用`super()。`这是一个很重要的规则，TypeScript 将会强制。

例子也显示了怎样使用子类特定的方法去覆盖基类的方法。`Snake`和`Horse`创建了一个`move`方法覆盖来自`Animal`的`move`，给他特定于每一个类的功能。注意尽管`tom`声明为一个`Animal`，因为他的值是一个`Horse`，调用`tom.move(34)`将会调用`Horse`覆盖的方法：

```
Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m.
```

### 公共，私有，和保护的修饰

### 默认公共

在我们的例子中，我们已经可以自由访问我们声明的成员。如果你在其他语言对类很熟悉。你可能注意到前面的例子我们没有使用单词`public`去完成这个；比如，C# 需要每一个成员明确的标记`public`。在 TypeScript，每一个成员默认是`public`。

你可能依旧明确标记`public`。我们可以以下面的方式去编写`Animal`：
```
class Animal {
  public name: string;

  public constructor(theName: string) {
    this.name = theName;
  }

  public move(distanceInMeters: number) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}
```

### ECMAScript 私有域

在 TypeScript 3.8，TypeScript 支持新的 JavaScript 私有域语法：
```
class Animal {
  #name: string;
  constructor(theName: string) {
    this.#name = theName;
  }
}

new Animal("Cat").#name;
Property '#name' is not accessible outside class 'Animal' because it has a private identifier.
```

这个语法内置到了 JavaScript 运行时，有更好的保证每一个私有域的隔离。现在，关于私有域最好的文档在 TypeScript 3.8 [发布日志]()。

### 理解 TypeScript 的私有

TypeScript 当然也有它自己的方式去声明被标记为`private`的成员，它不能从包含它的类的外部访问。比如：
```
class Animal {
  private name: string;

  constructor(theName: string) {
    this.name = theName;
  }
}

new Animal("Cat").name;
Property 'name' is private and only accessible within class 'Animal'.
```
TypeScript 是一个结构化类型系统。当我们比较两个不同的类型，无视他们来自哪里，如果所有成员的类型是兼容的，那么我们说类型自身也是兼容的。

然而，当比较`private` he ·protected`成员的时候，我们对待这些类型不同。因为两种类型被认为是兼容的，如果其中一个是`private`成员，这其他必须是`private`成员，来自相同的声明。同样适用于`protected`成员。

```
class Animal {
  private name: string;
  constructor(theName: string) {
    this.name = theName;
  }
}

class Rhino extends Animal {
  constructor() {
    super("Rhino");
  }
}

class Employee {
  private name: string;
  constructor(theName: string) {
    this.name = theName;
  }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee;
Type 'Employee' is not assignable to type 'Animal'.
  Types have separate declarations of a private property 'name'.
Try
```

在这个例子中，我们有`Animal`和一个`Rhino`，`Rhino`是`Animal`的一个子类。我们有一个新的类`Employee`，在外形方面看起来和`Animal`相同。我们创建了这些类的一些实例，然后尝试去给他们相互赋值，看看会发生什么。因为`Animal`和`Rhino`共享他们外形的`private`部分，从`Animal`的`private name:string`声明，他们是兼容的。然而，这不是`Employee`的场景。但我们尝试从一个`Employee`赋值给`Animal`，我们得到一个类型不兼容的错误。尽管`Employee`也有一个`private`成员叫做`name`。但是这不是我们声明在`Animal`的声明。


### 理解受保护的

`protected`修饰符表现的和`private`修饰符很像，除了`protect`声明的成员也可以被衍生类访问。比如，

```
class Person {
  protected name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Employee extends Person {
  private department: string;

  constructor(name: string, department: string) {
    super(name);
    this.department = department;
  }

  public getElevatorPitch() {
    return `Hello, my name is ${this.name} and I work in ${this.department}.`;
  }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name);
Property 'name' is protected and only accessible within class 'Person' and its subclasses.

```


注意，尽管我们不能在`Person`外面使用`name`，我们依旧可以在`Employee`的实例方法中使用它，因为`Employee`衍生自`Person`。

一个构造器可能被标记为`protected`。这意味着类不能在包含链外部被实例化，但是可以被继承。比如，
```
class Person {
  protected name: string;
  protected constructor(theName: string) {
    this.name = theName;
  }
}

// Employee can extend Person
class Employee extends Person {
  private department: string;

  constructor(name: string, department: string) {
    super(name);
    this.department = department;
  }

  public getElevatorPitch() {
    return `Hello, my name is ${this.name} and I work in ${this.department}.`;
  }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John");
Constructor of class 'Person' is protected and only accessible within the class declaration.
```

### 只读修饰符

你可以使用`readonly`关键字标记属性只读。只读属性必须初始化在他们声明的时候，或者在构造器中。
```
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;

  constructor(theName: string) {
    this.name = theName;
  }
}

let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit";
Cannot assign to 'name' because it is a read-only property.
```

### 参数属性

在我们最新的例子中，我们在Octopus`类中定义了一个只读成员`name`和一个构造器参数`theName`。这是需要的，为了让`theName`有值，在`Octopus`构造器被执行之后可以被访问。参数属性让你在一个地方创建和初始化一个成员。这是前面`Octopus`类使用参数属性的更深入版本：
```
class Octopus {
  readonly numberOfLegs: number = 8;
  constructor(readonly name: string) {}
}

let dad = new Octopus("Man with the 8 strong legs");
dad.name;
```

注意我们如何完全清理`theName`，并只在构造器使用短的`readonly name:string`参数去创建和初始化`name`成员。我们已经吧声明和赋值合并到了同一个地方。

参数属性通过在构造器参数前面使用可访问性修饰符或者`readonly`,或者全部来声明。使用`private`为一个参数属性声明和初始化一个私有成员；同样的，`public`，`protected`和`readonly`也做了同样的事情。

### 访问器

TypeScript 支持 getter/setter 作为拦截对一个对象成员访问的方式。这给你一个更细力度的控制每一个对象怎样互相访问成员。

转变一个简单的类去使用`get`和`set`。首先，从没有 getter 和 setter 的例子开始。

```
class Employee {
  fullName: string;
}

let employee = new Employee();
employee.fullName = "Bob Smith";

if (employee.fullName) {
  console.log(employee.fullName);
}
```

然而允许人们直接去自由的设置`fullName`太灵活了，当`fullname`被设置的时候，我们可能想要强制一些约束。

在这个版本，我们添加 setter 去检查`newName`确保它和后端数据库域的最大长度兼容。如果他不是，同门抛出一个错误通知客户端代码有些东西发生了错误。

为了保留已存在的功能，我们也添加一个简单的 getter 获取未修饰的`fullName`。这个类有三个成员：一个属性叫做

```
const fullNameMaxLength = 10;

class Employee {
  private _fullName: string;

  get fullName(): string {
    return this._fullName;
  }

  set fullName(newName: string) {
    if (newName && newName.length > fullNameMaxLength) {
      throw new Error("fullName has a max length of " + fullNameMaxLength);
    }

    this._fullName = newName;
  }
}

let employee = new Employee();
employee.fullName = "Bob Smith";

if (employee.fullName) {
  console.log(employee.fullName);
}
```

为了证明我们的访问器现在检查值的长度，我们可以尝试去赋值一个超过 10 个字符的名字，并验证我们得到一个错误。

关于访问器，有两点需要注意：

首先，访问起需要你去设置编译器输出 ECMAScript 5 或者更高。ECMAScript 3 以下的不支持。其次，有`get`的和没有`set`的访问器自定索引为`readonly`。这在从你的代码生成一个`.d.ts`文件的时候很有帮助，因为你的属性的用户可以可以看到他们但是不能改变他们。


### 静态属性

到目前未知，我们只讨论了类的实例成员，那些在它实例化之后出现在对象的。我们也可以创建类的静态成员，可以在类自身看见而不是在实例上。在这个例子中，我们在 origin 使用`static`，他是一个所有 grid 的通用值。每一个实例访问这个值通过前面拼接类的名字。和前面拼接`this.`到实例访问前面类似，这里我们拼接`Grid.`到静态访问前面。

```
class Grid {
  static origin = { x: 0, y: 0 };

  calculateDistanceFromOrigin(point: { x: number; y: number }) {
    let xDist = point.x - Grid.origin.x;
    let yDist = point.y - Grid.origin.y;
    return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
  }

  constructor(public scale: number) {}
}

let grid1 = new Grid(1.0); // 1x scale
let grid2 = new Grid(5.0); // 5x scale

console.log(grid1.calculateDistanceFromOrigin({ x: 10, y: 10 }));
console.log(grid2.calculateDistanceFromOrigin({ x: 10, y: 10 }));
```


### 抽象类

抽象类是其他类派生的基础类。他们可能无法直接实例化。。不想一个接口，一个抽象类可能包含他的成员的实现细节。`abstract`关键字用来定义抽象类，和抽象类内的抽象方法。

```
abstract class Animal {
  abstract makeSound(): void;

  move(): void {
    console.log("roaming the earth...");
  }
}
```
抽象类内被标记为抽象的方法不包含实现，并且必须在派生类中实现。抽象方法共享和接口方法相似的外形。都定义了一个方法的签名，但是不包含一个方法体。然而，抽象方法必须包含`abstract`关键字，可能可选的包含访问修饰符。

```
abstract class Department {
  constructor(public name: string) {}

  printName(): void {
    console.log("Department name: " + this.name);
  }

  abstract printMeeting(): void; // must be implemented in derived classes
}

class AccountingDepartment extends Department {
  constructor() {
    super("Accounting and Auditing"); // constructors in derived classes must call super()
  }

  printMeeting(): void {
    console.log("The Accounting Department meets each Monday at 10am.");
  }

  generateReports(): void {
    console.log("Generating accounting reports...");
  }
}

let department: Department; // ok to create a reference to an abstract type
department = new Department(); // error: cannot create an instance of an abstract class
Cannot create an instance of an abstract class.
department = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
department.printName();
department.printMeeting();
department.generateReports();
Property 'generateReports' does not exist on type 'Department'.
```

### 高级的技术

### 构造器函数

当你在 TypeScript 声明了一个类，你实际上同时创建了多个声明。首先是类的实例的类型。
```
class Greeter {
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

这里，当我们说`let greeter: Greeter`，我们使用`Greeter`作为`Greeter`类的实例的类型。这几乎是从其他面向对象语言来的程序员的第二天性。

我们也创建了其他的值，我们叫做构造器函数。这是我们`new`一个类的时候调用的函数。为了看看 实际中是怎么样的，看看前面例子中 JavaScript 创建的例子：

```
let Greeter = (function () {
  function Greeter(message) {
    this.greeting = message;
  }

  Greeter.prototype.greet = function () {
    return "Hello, " + this.greeting;
  };

  return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

这里，`let Greeter`将被构造器函数赋值。当我们调用`new`并运行这个函数，我们这个类的实例。构造器函数也包含所有的类的静态成员。思考每一种类的另一种方式是有一个实例端和静态端。

修改一点例子去显示这个不同：

```
class Greeter {
  static standardGreeting = "Hello, there";
  greeting: string;
  greet() {
    if (this.greeting) {
      return "Hello, " + this.greeting;
    } else {
      return Greeter.standardGreeting;
    }
  }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet()); // "Hello, there"

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet()); // "Hey there!"
```

在这个例子中，`greeterl`和之前类似。我们初始化`Greeter`类，并使用这个对象。这是我们之前看到的。

接下来，我们直接使用类。我们创建了一个新的变量叫做`greeterMaker`。这个变量将会持有类本身，或者说是它的构造器函数。这里我们使用`typeof Greeter`，也就是“给我 `Grrter`类自身的类型”，而不是实例类型，或者，更精确，“给我叫做`Greeter`的符号”，也就是构造器函数的类型。这个类型将会包含所有的 Greeter 静态成员，还有构再起创建`Greeter`类实例。我们显示这个通过在`greeterMaker`使用`new`，创建`Greeter`新的实例就像之前那样调用。

### 像接口一样使用类。

就像我们在之前的章节说的，一个类声明创建两个东西：一个表示类实例的类型，和一个构造器函数。因为类创建类型，你可以在可以使用接口的地方使用他们。

```
class Point {
  x: number;
  y: number;
}

interface Point3d extends Point {
  z: number;
}

let point3d: Point3d = { x: 1, y: 2, z: 3 };
```