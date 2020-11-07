[已校对]
# 新鲜度

- [新鲜度](https://basarat.gitbook.io/typescript/type-system/freshness#freshness)
- [允许额外的属性](https://basarat.gitbook.io/typescript/type-system/freshness#allowing-extra-properties)
- [使用场景： React](https://basarat.gitbook.io/typescript/type-system/freshness#use-case-react-state)

### 新鲜度

TypeScript 提供了**新鲜度**的概念（也叫做严格对象字面量检查），让对象字面量与结构化的类型是否兼容的类型检查更简单。

结构化类型非常方便。考虑下面的代码块。这允许你非常方便的升级你的 JavaScript 到 TypeScript，还保留一定程度的类型安全：
```ts
function logName(something: { name: string }) {
    console.log(something.name);
}

var person = { name: 'matt', job: 'being awesome' };
var animal = { name: 'cow', diet: 'vegan, but has milk of own species' };
var random = { note: `I don't have a name property` };

logName(person); // okay
logName(animal); // okay
logName(random); // Error: property `name` is missing
```

然而，结构化类型有一个缺点，它允许你去错误的认为一些东西接受的数据比它实际的多。这显示在下面的代码，这时候 TypeScript 将会显示一个错误：
```ts
function logName(something: { name: string }) {
    console.log(something.name);
}

logName({ name: 'matt' }); // okay
logName({ name: 'matt', job: 'being awesome' }); // Error: object literals must only specify known properties. `job` is excessive here.
```

注意这个错误只发生在对象字面量。没有这个错误，一个人可能查看调用`logName({ name: 'matt', job: 'being awesome' })`，并认为 logName 将使用`job`，然而实际上它完全被忽略。

另一个非常大的使用场景是有可选的成员的接口，没有这类的对象字面量检测，一个错误输入将会通过类型检测。这显示在下面：
```ts
function logIfHasName(something: { name?: string }) {
    if (something.name) {
        console.log(something.name);
    }
}
var person = { name: 'matt', job: 'being awesome' };
var animal = { name: 'cow', diet: 'vegan, but has milk of own species' };

logIfHasName(person); // okay
logIfHasName(animal); // okay
logIfHasName({neme: 'I just misspelled name to neme'}); // Error: object literals must only specify known properties. `neme` is excessive here.
```

为什么只有对象字面量使用这种方式检测类型的原因是因为在这种场景，多余的实际没有被使用的属性总是一个错误输入或者对 API 的错误理解。


### 允许额外的属性

一个可以包含索引签名的类型显示的只是允许多余的属性：
```ts
var x: { foo: number, [x: string]: any };
x = { foo: 1, baz: 2 };  // Ok, `baz` matched by index signature
```

### 使用场景： React 状态

[Facebook ReactJS](https://facebook.github.io/react/)为对象新鲜度提供了一个很好的使用场景。在一个组件，非常常见的场景，你使用少量的属性而不是传递所有的属性调用`setState`，比如：
```ts
// Assuming
interface State {
    foo: string;
    bar: string;
}

// You want to do: 
this.setState({foo: "Hello"}); // Error: missing property bar

// But because state contains both `foo` and `bar` TypeScript would force you to do: 
this.setState({foo: "Hello", bar: this.state.bar});
```
使用新鲜度的方法，你可以标记所有的成员为可选的，你依旧可以捕获输入错误：
```ts
// Assuming
interface State {
    foo?: string;
    bar?: string;
}

// You want to do: 
this.setState({foo: "Hello"}); // Yay works fine!

// Because of freshness it's protected against typos as well!
this.setState({foos: "Hello"}); // Error: Objects may only specify known properties

// And still type checked
this.setState({foo: 123}); // Error: Cannot assign number to a string
```