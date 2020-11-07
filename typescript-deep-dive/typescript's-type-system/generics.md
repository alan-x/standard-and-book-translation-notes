[已校对]
# 泛型

### 泛型 

泛型的关键动机是在成员之间记录有意义的类型依赖。这些成员可以是：
- 类实例成员
- 类方法
- 函数参数
- 函数返回值

### 动机和例子

考虑简单的`Queue`（先入先出）数据结构实现。一个 TypeScript / JavaScriot 的例子看起来像：
```ts
class Queue {
  private data = [];
  push(item) { this.data.push(item); }
  pop() { return this.data.shift(); }
}
```

这个实现的问题是它允许人们添加任何东西到队列，当他们弹出的时候可以是任何东西。这展示在下面，某个人可以推送一个`string`到队列，然而在实际使用的时候，假设只有`numers`被推送：

```ts
class Queue {
  private data = [];
  push(item) { this.data.push(item); }
  pop() { return this.data.shift(); }
}

const queue = new Queue();
queue.push(0);
queue.push("1"); // Oops a mistake

// a developer walks into a bar
console.log(queue.pop().toPrecision(1));
console.log(queue.pop().toPrecision(1)); // RUNTIME ERROR
```

一个解决方案（并且实际上在不支持泛型的语言是唯一的解决方案）是前进并为这些约束创建特殊的类。比如，快速和脏的数字队列：
```ts
class QueueNumber extends Queue {
  push(item: number) { super.push(item); }
  pop(): number { return this.data.shift(); }
}

const queue = new QueueNumber();
queue.push(0);
queue.push("1"); // ERROR : cannot push a string. Only numbers allowed

// ^ if that error is fixed the rest would be fine too
```

当然，这很快就会变成痛苦，比如，如果你想要一个字符串队列，你需要再搞一遍。实际你想要的是无论推入什么类型，返回的类型应该是一样的。这可以通过泛型参数简单实现（在这个场景，在类级别）：
```ts
/** A class definition with a generic parameter */
class Queue<T> {
  private data = [];
  push(item: T) { this.data.push(item); }
  pop(): T | undefined { return this.data.shift(); }
}

/** Again sample usage */
const queue = new Queue<number>();
queue.push(0);
queue.push("1"); // ERROR : cannot push a string. Only numbers allowed

// ^ if that error is fixed the rest would be fine too
```

另一个例子我们已经见过了，就是反转函数，这里的约束在传入函数和函数返回之间：
```ts
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// Safety!
reversed[0] = '1';     // Error!
reversed = ['1', '2']; // Error!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

在这个章节，你会看到定义在类级别和函数级别的泛型的例子。值得一提的是，你可以只为一个成员函数常见泛型。一个简单的例子是下面我们将`reverse`移动到一个`Utility`类：
```ts
class Utility {
  reverse<T>(items: T[]): T[] {
      var toreturn = [];
      for (let i = items.length - 1; i >= 0; i--) {
          toreturn.push(items[i]);
      }
      return toreturn;
  }
}
```

> 提示：你可以随你喜欢命名泛型参数。当你的泛型很简单的时候，通常约定使用`T`，`U`，`V`。如果你有对于一个泛型参数，尝试使用类似`TKey`和`TValue`之类有意义的名字（约定去添加`T`前缀，因为泛型在其他类似 C++ 的语言也叫做模板）。

### 设计模式：便利泛型

考虑函数：
```ts
declare function parse<T>(name: string): T;
```

在这个场景，你可以看到类型`T`只用在一个地方。因此在成员之间没有约束。就类型安全而言，这和类型断言相同：
```ts
declare function parse(name: string): any;

const something = parse('something') as TypeOfSomething;
```

就类型安全而言，只用一次的泛型不会比一个断言好。也就是说，他们都为你的 API 提供了便利。

一个更明显的例子是加载一个 json 响应的函数。他返回一个你传入的类型的 promise：
```ts
const getJSON = <T>(config: {
    url: string,
    headers?: { [key: string]: string },
  }): Promise<T> => {
    const fetchConfig = ({
      method: 'GET',
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      ...(config.headers || {})
    });
    return fetch(config.url, fetchConfig)
      .then<T>(response => response.json());
  }
```

注意你依旧需要声明你想要的，但是`getJSON<T>`签名`(config) => Promise<T>`节约你几个按键（你不需要声明`loadUsers`的返回类型，因为他可以被推断）：

```ts
type LoadUsersResponse = {
  users: {
    name: string;
    email: string;
  }[];  // array of user objects
}
function loadUsers() {
  return getJSON<LoadUsersResponse>({ url: 'https://example.com/users' });
}
```

当然`Promise<T>`作为一个返回值绝对比替代的`Promise<any>`好：
```ts
declare function send<T>(arg: T): void;
```

另一个例子是一个泛型只用作一个参数：
```ts
declare function send<T>(arg: T): void;
```

这里，泛型`T`可以用于声明你想要参数匹配的类型，比如：
```ts
send<Something>({
  x:123,
  // Also you get autocomplete  
}); // Will TSError if `x:123` does not match the structure expected for Something
```