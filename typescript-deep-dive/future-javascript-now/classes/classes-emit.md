[已校对]
# 类生成

### IIFE 怎么了

类生成的 js 可能是：
```ts
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.add = function (point) {
    return new Point(this.x + point.x, this.y + point.y);
};
```
它被包裹在一个立即执行函数表达死后（IIFE）的原因是，比如
```ts
(function () {

    // BODY

    return Point;
})();
```
需要继承。它允许 TypeScript 捕获基础类作为一个变量`_super`，比如
```ts
var Point3D = (function (_super) {
    __extends(Point3D, _super);
    function Point3D(x, y, z) {
        _super.call(this, x, y);
        this.z = z;
    }
    Point3D.prototype.add = function (point) {
        var point2D = _super.prototype.add.call(this, point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    };
    return Point3D;
})(Point);
```

注意 IIFE 允许 TypeScript 去简单捕获基础类`Point`在一个`_super`变量，在类体内使用是一致的。


### `__extends`

你将会注意到当你继承一个类的时候，TypeScript 也生成了下面的函数：
```ts
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```

这里`d`引用了分发的类，并且`b`引用基础类。这个函数做两件事：

- 复制基础类的静态成员到子类，比如`for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];`
- 设置子类的函数的原型为可选的查找父`proto`的成员，比如有效的`d.prototype.__proto__ = b.prototype`

人们很少对理解 1 有困难，但是很多人在 2 挣扎。因此按顺序解释。

### `d.prototype.__proto__ = b.prototype`

在辅导了很多人之后，我发现下面的解释最简单。首先，我们将解释`__extends`的代码是怎样和简单的`d.prototype.__proto__ = b.prototype`相同，然后，为什么这行有意义。为了理解这里的全部，你需要知道这些东西：
1. `__proto__`
2. `prototype`
3. `new`在`this`内部调用的函数的效果
4. `new`在`prototype`阿和`__proto__`的效果

JavaScipt 中的所有对象包含一个`__proto__`成员。这个成员通常在旧的浏览器无法访问（有时候文档引用这个魔法属性为`[[prototype]]`）。它有一个目标：如果一个属性在对象查找的时候没有找到（比如，`obj.property`），则它会在`obj.__proto__.property`查找。如果它依旧没有找到，则`obj.__proto__.__proto__.property`，直到：它被找到或者最新的`.__proto__`它是 null。这解释为什么 JavaScript 被称为开箱支持原型继承。这显示在下面，你可以运行在 chrome 的控制台或者 Node.js：
```ts
var foo = {}

// setup on foo as well as foo.__proto__
foo.bar = 123;
foo.__proto__.bar = 456;

console.log(foo.bar); // 123
delete foo.bar; // remove from object
console.log(foo.bar); // 456
delete foo.__proto__.bar; // remove from foo.__proto__
console.log(foo.bar); // undefined
```

现在你理解了`__proto__`。另一个有用的事实是 JavaScript 所有`function`都有一个属性叫做`prototype`，它有一个成员`constructor`指向这个函数。这显示在下面：
```ts
function Foo() { }
console.log(Foo.prototype); // {} i.e. it exists and is not undefined
console.log(Foo.prototype.constructor === Foo); // Has a member called `constructor` pointing back to the function
```

现在来看看`new`在`this`内部调用的函数的效果吧。基本上调用的函数内部的`this`用于指出将会从函数返回的新创建的对象。很简单就能看到，如果你在函数内部的`this`操作一个属性：
```ts
function Foo() {
    this.bar = 123;
}

// call with the new operator
var newFoo = new Foo();
console.log(newFoo.bar); // 123
```

现在，你需要知道的其他东西只有在函数上调用`new`赋值函数的`prototype`到函数调用返回的新创建的对象的`__proto__`。这是你可以运行去完全理解它的代码：
```ts
function Foo() { }

var foo = new Foo();

console.log(foo.__proto__ === Foo.prototype); // True!
```

就是它，现在看看下面更直接的`__extends`。我已经直接为这些行编了号：
```ts
1  function __() { this.constructor = d; }
2   __.prototype = b.prototype;
3   d.prototype = new __();
```

按相反的方向阅读这个函数，第三行的`d.prototype = new __()`效果意味着`d.prototype = {__proto__ : __.prototype}`（因为`new`在`prototype`和`__proto__`的效果），将它和前面那行联系起来（比如，第二行`__.prototype = b.prototype;`）你得到`d.prototype = {__proto__ : b.prototype}`。

但是等等，我们想要`d.prototype.__proto__`，比如，只是 proto 改变，并维护旧的`d.prototype.constructor`。这是第一行有意义的地方（比如，`function __() { this.constructor = d; }`）来了。这里，我们将有效的拥有`d.prototype = {__proto__ : __.prototype, constructor : d}`。因此，因为我们重新存储了`d.prototype.constructor`，我们真实操作的唯一的东西是`__proto__`，因此`d.prototype.__proto__ = b.prototype`。

### `d.prototype.__proto__ = b.prototype`意义

意义是它允许你去添加成员函数到一个子类，并从基础类继承。这通过下面简单的例子展示：
```ts
function Animal() { }
Animal.prototype.walk = function () { console.log('walk') };

function Bird() { }
Bird.prototype.__proto__ = Animal.prototype;
Bird.prototype.fly = function () { console.log('fly') };

var bird = new Bird();
bird.walk();
bird.fly();
```

基本上，`bird.fly`将会从`bird.__proto__.fly`（记住，`new`让`bird.__proto__`指向`Bird.prototype`），`bird.walk`（一个继承的成员）将会从`bird.__proto__.__proto__.walk`查找（因为`bird.__proto__ == Bird.prototype`和`bird.__proto__.__proto__`==`Animal.prototype`）。