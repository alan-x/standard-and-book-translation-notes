# 迭代器

迭代器本身不是 TypeScript 或者 ES6 的特性，迭代器是一个行为设计模式常见于面向对象编程语言。通常，一个对象实现下面的接口：
```ts
interface Iterator<T> {
    next(value?: any): IteratorResult<T>;
    return?(value?: any): IteratorResult<T>;
    throw?(e?: any): IteratorResult<T>;
}
```

（[更多在之后的`<T>`声明]()）

这个接口允许去找回一个值，从一些术语对象的集合或者序列。

`IteratorResult`是简单的`value`+`done`对：
```ts
interface IteratorResult<T> {
    done: boolean;
    value: T;
}
```

想象有一些帧的对象，包含一个组件列表，使用迭代器接口，它可能从这个帧对象中获取组件，像下面：
```ts
class Component {
  constructor (public name: string) {}
}

class Frame implements Iterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true,
        value: null
      }
    }
  }

}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
let iteratorResult1 = frame.next(); //{ done: false, value: Component { name: 'top' } }
let iteratorResult2 = frame.next(); //{ done: false, value: Component { name: 'bottom' } }
let iteratorResult3 = frame.next(); //{ done: false, value: Component { name: 'left' } }
let iteratorResult4 = frame.next(); //{ done: false, value: Component { name: 'right' } }
let iteratorResult5 = frame.next(); //{ done: true, value: null }

//It is possible to access the value of iterator result via the value property:
let component = iteratorResult1.value; //Component { name: 'top' }
```

再一次。迭代器不是一个 TypeScript 特性，这个代码可以工作，不需要实现 Iterator 和 IteratorResult 接口。然而，为了代码一致性使用这些常见 ES6 [接口]()非常有帮助。

好，非常棒，但是更有帮助。ES6 定义了可迭代协议，包含[Symbol.iterator]`symbol`，如果 Iterable 接口可以实现：
```ts
//...
class Frame implements Iterable<Component> {

  constructor(public name: string, public components: Component[]) {}

  [Symbol.iterator]() {
    let pointer = 0;
    let components = this.components;

    return {
      next(): IteratorResult<Component> {
        if (pointer < components.length) {
          return {
            done: false,
            value: components[pointer++]
          }
        } else {
          return {
            done: true,
            value: null
          }
        }
      }
    }
  }
}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
for (let cmp of frame) {
  console.log(cmp);
}
```

不幸的是`frame.next()`无法和这个模式一起用，它看起来有点笨重。IterableIterator 接口可以拯救

```ts
//...
class Frame implements IterableIterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true,
        value: null
      }
    }
  }

  [Symbol.iterator](): IterableIterator<Component> {
    return this;
  }

}
//...
```

`frame.next()`和`for`循环现在可以和 IterableIterator 接口一起用。

Iterator 不需要迭代一个有限值。典型的例子是一个斐波那契序列：
```ts
class Fib implements IterableIterator<number> {

  protected fn1 = 0;
  protected fn2 = 1;

  constructor(protected maxValue?: number) {}

  public next(): IteratorResult<number> {
    var current = this.fn1;
    this.fn1 = this.fn2;
    this.fn2 = current + this.fn1;
    if (this.maxValue != null && current >= this.maxValue) {
      return {
        done: true,
        value: null
      } 
    } 
    return {
      done: false,
      value: current
    }
  }

  [Symbol.iterator](): IterableIterator<number> {
    return this;
  }

}

let fib = new Fib();

fib.next() //{ done: false, value: 0 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 2 }
fib.next() //{ done: false, value: 3 }
fib.next() //{ done: false, value: 5 }

let fibMax50 = new Fib(50);
console.log(Array.from(fibMax50)); // [ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34 ]

let fibMax21 = new Fib(21);
for(let num of fibMax21) {
  console.log(num); //Prints fibonacci sequence 0 to 21
}
```

### 为 ES5 目标构建有迭代器的代码

前面的代码例子需要 ES6 目标。然而，他可以和 ES5 目标一起用，如果目标 JS 引擎支持`Symbol.iterator`。这可以通过在 ES5 目标使用 ES6 库达到（添加 es6.d.ts 到你的目标）去让他编译。编译的代码可以在 node 4+，Google Chrome 和一些其他浏览器工作。