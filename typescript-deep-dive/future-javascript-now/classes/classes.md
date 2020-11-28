[已校对]
# 类

### 类

为什么在 JavaScript 中，将类作为第一类项很重要的原因是：

1. [类提供了一个有用的结构化抽象](https://basarat.gitbook.io/typescript/main-1/classesareuseful)
2. 为开发者提供一个一致的方式去使用类，而不是每一个框架（emberjs，reactjs 等）一个自己的版本。

最终 JavaScript 开发者可以有`class`。这里我们有一个基本类叫做 Point：
```ts
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```

这个类生成了下面的 JavaScript，在 ES5 生成：
```ts
var Point = (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.add = function (point) {
        return new Point(this.x + point.x, this.y + point.y);
    };
    return Point;
})();
```
这是一个非常理想的传统的 JavaScript 类模式作为第一级语言结构。

### 继承

TypeScript 中的类（就像其他语言）支持单独继承，使用`extends`关键字，如下显示：
```ts
class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```

如果你有一个构造器在你的类，你必须在你的构造器调用父级构造器（TypeScript 为你指出了它）。这确保需要设置到`this`上的东西被设置。在`super`调用之后，你可以添加任何你想要在你构造器执行的额外的东西（这里我们添加了另一个成员`z`）。

注意，你很简单就能覆盖父成员函数（这里我们覆盖`add`）并依旧在你的成员使用父类的功能（使用`super,`语法）。

### 静态

TypeScript 类支持`static`属性，在所有类的实例共享。放置（和访问）他们的一个自然的地方是在类自身，这也是 TypeScript 做的：
```ts
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```
你可以拥有静态成员，同样静态函数也可以。

### 访问修饰器

TypeScript 支持访问器修饰符`public`，`private`和`protected`，它决定`class`成员的可访问性，如下：


| 在哪可访问 | `public` | `protected` | `private` |
| -- | -- | -- | -- |
| 类 | yes | yes | yes |
| 子类 | yes | yes | yes |
| 类实例 | yes | yes | yes |

如果一个访问器修饰符没有指定，它暗示着`public`，因为它命中了 JavaScript 的约定。

注意在运行时（在生成的 JS），这不重要，但是将会给你一个编译时错误，如果你不正确的使用他们。一个例子显示在下面：
```ts
class FooBase {
    public x: number;
    private y: number;
    protected z: number;
}

// EFFECT ON INSTANCES
var foo = new FooBase();
foo.x; // okay
foo.y; // ERROR : private
foo.z; // ERROR : protected

// EFFECT ON CHILD CLASSES
class FooChild extends FooBase {
    constructor() {
      super();
        this.x; // okay
        this.y; // ERROR: private
        this.z; // okay
    }
}
```

这些修饰器对于成员属性和成员函数也有效。

### 抽象

`abstract`可以被认为是一个访问修饰器。我们分离的展示它是因为相对于前面提到的修饰器，他可以在一个`class`上，就像类上的任何成员。有一个`abstract`修饰器主要意味着这类功能不能被直接调用，一个子类必须提供功能。

- `abstract`类不能被直接实例化，用户必须创建一些类去继承`abstract clas`。
```ts
abstract class FooCommand {}

class BarCommand extends FooCommand {}

const fooCommand: FooCommand = new FooCommand(); // Cannot create an instance of an abstract class.

const barCommand = new BarCommand(); // You can create an instance of a class that inherits from an abstract class.
```

- `abstract`成员不能直接被访问，并且一个子类必须提供功能
```ts
abstract class FooCommand {
  abstract execute(): string;
}

class BarErrorCommand  extends FooCommand {} // 'BarErrorCommand' needs implement abstract member 'execute'.

class BarCommand extends FooCommand {
  execute() {
    return `Command Bar executed`;
  }
}

const barCommand = new BarCommand();

barCommand.execute(); // Command Bar executed
```

### 构造器是可选的

类不需要拥有一个构造器，比如，下面是很好的
```ts
class Foo {}
var foo = new Foo();
```

### 使用构造器定义

在一个类中拥有一个成员和初始化它像下面：
```ts
class Foo {
    x: number;
    constructor(x:number) {
        this.x = x;
    }
}
```

这是一个常见的模式，TypeScript 提供一个简短方式，你可以在成员前面添加一个访问修饰符，它自动在类上声明并从构造器复制。因此前面的例子可以重写为（注意`public x:number`）：
```ts
class Foo {
    constructor(public x:number) {
    }
}
```


### 属性初始化

这是一个有技巧的特性，被 TypeScript 支持（实际上从 ES7 开始）。你可以在构造器之外初始化类的任何成员，对于提供默认值很有用（注意`members = []`）
```ts
class Foo {
    members = [];  // Initialize directly
    add(x) {
        this.members.push(x);
    }
}
```