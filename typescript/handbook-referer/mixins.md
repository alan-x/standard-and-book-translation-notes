Mixins

除了传统面向对象层级，从可重用组件构建类的另一种流行的方式是通过绑定简单的部分类去构建。你可能对类似像 Scala 语言中的混入和特性的想法很熟悉，这个模式在 JavaScript 社区也得到一些流行。

### 一个 mixin 是如何工作的

这个模式依赖于使用泛型类型继承去扩展一个基本类。TypeScript 的最好的 mixin 支持方式是通过类表达语法。你可以在[JavaScript 这里]()了解更多关于这个模式的东西。

为了开始，我们将需要一个类，用来应用 minxin：
```ts
class Sprite {
  name = "";
  x = 0;
  y = 0;

  constructor(name: string) {
    this.name = name;
  }
}
```

然后你需要一个类型和一个类
```ts
// To get started, we need a type which we'll use to extend
// other classes from. The main responsibility is to declare
// that the type being passed in is a class.

type Constructor = new (...args: any[]) => {};

// This mixin adds a scale property, with getters and setters
// for changing it with an encapsulated private property:

function Scale<TBase extends Constructor>(Base: TBase) {
  return class Scaling extends Base {
    // Mixins may not declare private/protected properties
    // however, you can use ES2020 private fields
    _scale = 1;

    setScale(scale: number) {
      this._scale = scale;
    }

    get scale(): number {
      return this._scale;
    }
  };
}
```

在全部设置好之后，你可以创建一个类，表示基本类，mixin 将会被应用：
```ts
// Compose a new class from the Sprite class,
// with the Mixin Scale applier:
const EightBitSprite = Scale(Sprite);

const flappySprite = new EightBitSprite("Bird");
flappySprite.setScale(0.8);
console.log(flappySprite.scale);
```

### 强制混入
在前面的例子中，mixin 没有关于类的底层设计很难让你作出你想要的设计。

为了建模这个，我们修改了原始的构造器类型去接受一个通用的参数。
```ts
// This was our previous constructor:
type Constructor = new (...args: any[]) => {};
// Now we use a generic version which can apply a constraint on
// the class which this mixin is applied to
type GConstructor<T = {}> = new (...args: any[]) => T;
```
这允许创建一个类，值作用于受限于基本类：
```ts
type Positionable = GConstructor<{ setPos: (x: number, y: number) => void }>;
type Spritable = GConstructor<typeof Sprite>;
type Loggable = GConstructor<{ print: () => void }>;
```

然后你就可以创建 minxin，只在你有部分基础时候工作：
```ts
function Jumpable<TBase extends Positionable>(Base: TBase) {
  return class Jumpable extends Base {
    jump() {
      // This mixin will only work if it is passed a base
      // class which has setPos defined because of the
      // Positionable constraint.
      this.setPos(0, 20);
    }
  };
}
```

### 替代模式

这个文档的前面版本推荐一个方式去编写 mixin，分别创建运行时和类型层级，然和在最后合并：
```ts
// Each mixin is a traditional ES class
class Jumpable {
  jump() {}
}

class Duckable {
  duck() {}
}

// Including the base
class Sprite {
  x = 0;
  y = 0;
}

// Then you create an interface which merges
// the expected mixins with the same name as your base
interface Sprite extends Jumpable, Duckable {}
// Apply the mixins into the base class via
// the JS at runtime
applyMixins(Sprite, [Jumpable, Duckable]);

let player = new Sprite();
player.jump();
console.log(player.x, player.y);

// This can live anywhere in your codebase:
function applyMixins(derivedCtor: any, constructors: any[]) {
  constructors.forEach((baseCtor) => {
    Object.getOwnPropertyNames(baseCtor.prototype).forEach((name) => {
      Object.defineProperty(
        derivedCtor.prototype,
        name,
        Object.getOwnPropertyDescriptor(baseCtor.prototype, name)
      );
    });
  });
}
```

这个模式较少的依赖于编译器，更多依赖于你的代码库去确保运行时和类型系统同时保持正确。

### 约束

mixin 模式在 TypeScript 编译器中是原声支持的，通过代码流分析。这是一些场景，你可以命中原生支持的边缘。

### 装饰器和 mixin [#4881]()

你不能通过代码流分析使用装饰器去提供 mixin：

```ts
// A decorator function which replicates the mixin pattern:
const Pausable = (target: typeof Player) => {
  return class Pausable extends target {
    shouldFreeze = false;
  };
};

@Pausable
class Player {
  x = 0;
  y = 0;
}

// The Player class does not have the decorator's type merged:
const player = new Player();
player.shouldFreeze;
Property 'shouldFreeze' does not exist on type 'Player'.

// It the runtime aspect could be manually replicated via
// type composition or interface merging.
type FreezablePlayer = typeof Player & { shouldFreeze: boolean };

const playerTwo = (new Player() as unknown) as FreezablePlayer;
playerTwo.shouldFreeze;
```

### 静态属性 迷信[#17829]()

与其说是一个约束，不如说是一个陷阱。类表达模式创建单例，因此他们不能在类型系统中映射去支持不同的变量类型。

你可以通过使用函数去返回你的类去使用它，而不是基于一个泛型。
```ts
function base<T>() {
  class Base {
    static prop: T;
  }
  return Base;
}

function derived<T>() {
  class Derived extends base<T>() {
    static anotherProp: T;
  }
  return Derived;
}

class Spec extends derived<string>() {}

Spec.prop; // string
Spec.anotherProp; // string
```