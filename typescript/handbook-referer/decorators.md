装饰器

### 介绍

随着 TypeScript 和 ES6 中类的引入，存在某些场景需要额外的特性去支持注解或者修改类和类的成员。注解支持一种方式去添加声明，为类声明和成员提供一个元数据编程语法。注解是 JavaScript 的一个[阶段2提案]()，作为 TypeScript 的一个体验性特性引入。

注意：注解是一个体验性特性，可能在未来的发行改变。

为了启用注解的体验性支持，你必须在命令行或者在你的`tsconfig,json`启用`experimentalDecorators`编译器选项：

命令行：
```
tsc --target ES5 --experimentalDecorators
```

tsconfig.json
```
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
```

### 注解

一个注解是一个特殊类型的声明，可以被绑定到一个[类声明]()，[方法]()，[访问器]()，[属性]()，或者[参数]()。注解使用`@expression`的形式，`expression`必须求值为一个函数，将会在运行时使用修饰的声明的相关信息调用。

比如，修饰器`@sealed`可能如下编写`sealed`函数：
```
function sealed(target) {
  // do something with 'target' ...
}

```

注意：你可以在[类修饰器]()查阅更详细的例子。

### 修饰器工厂

如果我们想要自定义一个修饰器如何应用于一个声明，我们可以编写一个修饰起工厂。一个修饰器共产只是一个简单的函数，返回表达式，将会在运行时调用修饰器。


我们可以以下面的风格编写一个修饰起工厂：
```
function color(value: string) {
  // this is the decorator factory
  return function (target) {
    // this is the decorator
    // do something with 'target' and 'value'...
  };
}

```

注意：你可以在[方法修饰器]()了解更多详细例子。

### 装饰器组成

多个修饰起可以应用于一个声明，就像下面的例子：

- 在单独的一行：
```
@f @g x
```

- 在多行：
```
@f
@g
x
```

当多个声明应用于一个单独的声明，他们的求值类似于[数学中的函数组合]()。在这个模型下，当组合函数 f 和 g，(f * g)(x) 的组合结果和 f(g(x)) 相同。

同样的，TypeScript 中，当在单独的声明上求值多个注解的时候，执行下面步骤：

1. 每一个注解的表达式从上至下执行
2. 然后结果从下到上作为函数调用

如果我们使用[装饰器工厂]()，我们可以从下面的例子观察到这个求值顺序：
```
function f() {
  console.log("f(): evaluated");
  return function (
    target,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log("f(): called");
  };
}

function g() {
  console.log("g(): evaluated");
  return function (
    target,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log("g(): called");
  };
}

class C {
  @f()
  @g()
  method() {}
}
```

这将会输出结果到控制台：
```
f(): evaluated
g(): evaluated
g(): called
f(): called
```

### 装饰器求值

以下很好的定义了应用于类内的多个声明装饰器是怎样被应用的顺序：

1. 参数修饰器，还有方法，访问器，或者属性装饰器，应用于每一个实例成员
2. 参数修饰器，还有方法，访问器，或者属性修饰器，应用于每一个静态成员
3. 参数修饰器，应用于构造器
4. 类修饰器应用于类

### 类修饰器

一个类声明在类声明之前声明。类装饰器应用于类的构造器，可以用于观察，修改，或者替换一个类定义。一个类修饰符必须用在宇哥声明文件，或者任何其他环境（比如在一个`declare`类）


如果类装饰器返回一个值，它将会使用提供的构造器函数替代类声明。

注意：如果你选择返回一个新的构造函数，你必须注意维护原始的原型。运行时应用装饰器的逻辑将不会为你做这个。

下面是一个类装饰器的例子（`@sealed`）应用于`Greeter`类：
```
@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

当`@sealed`被执行，它将会冻结构造器和原型。

下一步，我们有一个如何覆盖构造器的例子，
```
function classDecorator<T extends { new (...args: any[]): {} }>(
  constructor: T
) {
  return class extends constructor {
    newProperty = "new property";
    hello = "override";
  };
}

@classDecorator
class Greeter {
  property = "property";
  hello: string;
  constructor(m: string) {
    this.hello = m;
  }
}

console.log(new Greeter("world"));
```

### 方法装饰器

一个方法装饰器就在方法声明之前。装饰器应用于方法的属性装饰器，并且可以用于观察，修改，或者修改方法定义。一个方法装饰器不能用于声明文件，重载，或者任何其他环境上下文（比如在一个`declare`类）。

方法装饰器的表达式将会在运行时作为函数调用，使用下面的三个参数：

1. 一个类的静态成员的构造函数，或者类实例成员的的原型。

2. 成员的名字。

3. 成员的属性装饰器。

注意：如果你的脚本目标小于 ES5，属性装饰器将会是`undefined`。

下面是一个例子，一个方法装饰器（`@enumerable`）应用于一个`Greeter`类的方法：
```ts
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }

  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```
我们可以使用下面的函数声明定义`@enumerbale`：
```
function enumerable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.enumerable = value;
  };
}
```

这里的`@enumberable(false)`装饰器是一个[装饰器工厂]()。当`@enumberabale(false)`装饰器被调用的时候，它修改属性描述符的`enumerabale`属性。


### 访问器装饰器

一个访问器装饰器声明在访问器声明之前。访问器装饰器应用于访问器的属性修饰符，并且可以用于观察，定义，或者替代一个访问器的定义。一个访问器装饰器不能用在一个声明文件，或者任何其他环境上下文（比如在一个`declare`类）。

注意：TypeScript 不允许为一个单独的成员装饰`get`和`set`访问器。相反，成员的所有的装饰器必须应用于文档顺序的第一个访问器。这是因为应用于属性描述符的装饰器，结合了`get`和`set`访问器，不是分开声明。

这个访问器声明的表达式将会在运行时作为函数调用，使用下面的三个参数：

1. 一个类的静态成员的构造函数，或者类实例成员的的原型。

2. 成员的名字。

3. 成员的属性装饰器。

注意：如果你的脚本目标小于 ES5，属性装饰器将会是`undefined`。

如果访问器装饰器返回一个值，它将会作为成员的属性装饰器。

注意：返回值将会被忽略，如果你的脚本目标小于`ES5`。

下面是一个例子，关于访问器装饰器（`@configurable`）应用于`Point`类的成员：
```ts
class Point {
  private _x: number;
  private _y: number;
  constructor(x: number, y: number) {
    this._x = x;
    this._y = y;
  }

  @configurable(false)
  get x() {
    return this._x;
  }

  @configurable(false)
  get y() {
    return this._y;
  }
}
```

我们可以使用下面函数声明定义`@configurable`装饰器：
```ts
function configurable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.configurable = value;
  };
}
```

### 属性装饰器

一个属性装饰器声明在属性声明之前。一个属性装饰器不能用在一个声明文件，或者在任何其他环境上下文（比如在一个`declare`类）。

这个访问器声明的表达式将会在运行时作为函数调用，使用下面的两个参数：

1. 一个类的静态成员的构造函数，或者类实例成员的的原型。

2. 成员的名字。

注意：一个属性描述符不提供为一个属性装饰器的参数是因为属性装饰器的方式是初始化在 TypeScript 重的。这是因为现在没有机制去描述一个实例属性，当定义一个原型的成员的死后，并且没有方式去观察或者定义属性的初始器。返回值也被忽略了。比如，一个属性装饰器只能用于观察一个指定名字被声明在类中的属性。

我们可以使用这个信息去记录原型的元信息，就像下面的例子：

```ts
class Greeter {
  @format("Hello, %s")
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    let formatString = getFormat(this, "greeting");
    return formatString.replace("%s", this.greeting);
  }
}

```

我们可以使用下面的函数声明定义`@format`装饰器和`getFormat`函数：
```ts
import "reflect-metadata";

const formatMetadataKey = Symbol("format");

function format(formatString: string) {
  return Reflect.metadata(formatMetadataKey, formatString);
}

function getFormat(target: any, propertyKey: string) {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}

```

这里的`@format("Hello, %s")`装饰器是一个[装饰器工厂]()。当`@format("Hello, %s")`被调用，它为属性添加一个元信息入口，使用`reflect-metadata`库的`Reflect.metadata`函数。当`getFormat`被调用，它从格式读取元信息值。

注意：这个例子需要`reflect-metadata`库。查阅[Meatadata]()了解更多关于`reflect-metadata`库。


### 参数装饰器

一个参数装饰器声明在参数声明之前。参数装饰器应用于函数，比如类构造器，或者方法声明。一个参数装饰器不能用于声明文件，一个重载，或者在其他环境上下文（比如在一个`decleare`类）。

这个访问器声明的表达式将会在运行时作为函数调用，使用下面的三个参数：

1. 一个类的静态成员的构造函数，或者类实例成员的的原型。

2. 成员的名字。

3. 参数在函数参数列表中的顺序索引。

注意：一个参数装饰器只能用于观察一个蚕食是否被声明在方法中。

参数装饰器的返回值会被忽略。

下面是一个例子，关于参数装饰器（`@required`）应用于`Greeter`类的成员的参数：
```
class Greeter {
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  @validate
  greet(@required name: string) {
    return "Hello " + name + ", " + this.greeting;
  }
}
```

我们可以使用下面的函数声明定义`@required`和`@validate`：
```ts
import "reflect-metadata";

const requiredMetadataKey = Symbol("required");

function required(
  target: Object,
  propertyKey: string | symbol,
  parameterIndex: number
) {
  let existingRequiredParameters: number[] =
    Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata(
    requiredMetadataKey,
    existingRequiredParameters,
    target,
    propertyKey
  );
}

function validate(
  target: any,
  propertyName: string,
  descriptor: TypedPropertyDescriptor<Function>
) {
  let method = descriptor.value;
  descriptor.value = function () {
    let requiredParameters: number[] = Reflect.getOwnMetadata(
      requiredMetadataKey,
      target,
      propertyName
    );
    if (requiredParameters) {
      for (let parameterIndex of requiredParameters) {
        if (
          parameterIndex >= arguments.length ||
          arguments[parameterIndex] === undefined
        ) {
          throw new Error("Missing required argument.");
        }
      }
    }

    return method.apply(this, arguments);
  };
}
```

`@required`装饰器添加一个元信息入口，标记参数是必须的。`@validate`装饰器包裹存在的`greet`方法到一个函数，在调用原始方法之前验证参数。

注意：这个例子需要`reflect-metadata`库。查阅[Meatadata]()了解更多关于`reflect-metadata`库。


### Metadata

一些例子使用`relfect-metadata`库去为[体验性的元信息 API]()添加垫片。这个库还不是 ECMAScript(JavaScript)标准的一部分。然而，一旦装饰器官方接受成为 ECMAScript 标砖的一部分，这些存在将提议采用。

你可以通过 npm 安装这个库：
```
npm i reflect-metadata --save

```
TypeScript 包含为有某种装饰器的发送某种类型的元信息的体验性支持。为了启用这个体验性支持，你必须设置`emitDecoratorMetadata`编译器选项，在命令行或者在你的`tsconfig.json`：

命令行：
```
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata

```

tsconfig.json:
```
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

当启用的时候，只要`reflect-metadata`库被引入，额外的设计实践类型信息将会在运行时被暴露。

```ts
import "reflect-metadata";

class Point {
  x: number;
  y: number;
}

class Line {
  private _p0: Point;
  private _p1: Point;

  @validate
  set p0(value: Point) {
    this._p0 = value;
  }
  get p0() {
    return this._p0;
  }

  @validate
  set p1(value: Point) {
    this._p1 = value;
  }
  get p1() {
    return this._p1;
  }
}

function validate<T>(
  target: any,
  propertyKey: string,
  descriptor: TypedPropertyDescriptor<T>
) {
  let set = descriptor.set;
  descriptor.set = function (value: T) {
    let type = Reflect.getMetadata("design:type", target, propertyKey);
    if (!(value instanceof type)) {
      throw new TypeError("Invalid type.");
    }
    set.call(target, value);
  };
}
```


TypeScript 编译器将使用`@Reflect。metadata`注入设计时类型信息。你应该考虑它和后续相等的 TypeScript：

```ts
class Line {
  private _p0: Point;
  private _p1: Point;

  @validate
  @Reflect.metadata("design:type", Point)
  set p0(value: Point) {
    this._p0 = value;
  }
  get p0() {
    return this._p0;
  }

  @validate
  @Reflect.metadata("design:type", Point)
  set p1(value: Point) {
    this._p1 = value;
  }
  get p1() {
    return this._p1;
  }
}
```

注意：装饰器元信息是一个体验性特性，可能在未来发行引入破坏性改变。