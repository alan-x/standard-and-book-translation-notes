[已校对]
# Mixins

TypeScript（和 JavaScript）类支持严格的单一继承。因此你不能：
```ts
class User extends Tagged, Timestamped { // ERROR : no multiple inheritance
}
```
从可重用组件构建类的其他方式是通过结合简单的部分类去构建他们，这叫做 mixins。

想法很简单，与其让类 A 继承 类 B 去获取他的功能，函数 B 接受一个类 A 返回一个拥有这个新增的功能的新的类。函数`B`是一个 mixin。

> [A mixin 是]一个函数
>  1. 接受一个构造器
>  2. 创建一个类，使用新功能扩展这个构造器
>  3. 返回一个新的类


一个完整的例子
```ts
// Needed for all mixins
type Constructor<T = {}> = new (...args: any[]) => T;

////////////////////
// Example mixins
////////////////////

// A mixin that adds a property
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

// a mixin that adds a property and methods
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActivated = false;

    activate() {
      this.isActivated = true;
    }

    deactivate() {
      this.isActivated = false;
    }
  };
}

////////////////////
// Usage to compose classes
////////////////////

// Simple class
class User {
  name = '';
}

// User that is Timestamped
const TimestampedUser = Timestamped(User);

// User that is Timestamped and Activatable
const TimestampedActivatableUser = Timestamped(Activatable(User));

////////////////////
// Using the composed classes
////////////////////

const timestampedUserExample = new TimestampedUser();
console.log(timestampedUserExample.timestamp);

const timestampedActivatableUserExample = new TimestampedActivatableUser();
console.log(timestampedActivatableUserExample.timestamp);
console.log(timestampedActivatableUserExample.isActivated);
```

来剖析这个例子

### 接受一个构造器

mixins 接受一个类并使用新功能扩展它。因此我们需要去定义什么是一个构造器，比如：
```ts

// Needed for all mixins
type Constructor<T = {}> = new (...args: any[]) => T;
```

### 扩展类并返回它

非常简单：
```ts
// A mixin that adds a property
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```
就是这了