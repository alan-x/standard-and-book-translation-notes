# 生成器

`function *`是用于创建一个生成器函数的语法。调用一个生成器函数返回一个生成器对象。生成器对象只是实现[迭代器]()接口（比如，`next`，`return`和`throw`函数）。

生成器函数背后有两个主要动机：

### 懒迭代器

生成器函数可以用于创建懒迭代器，比如，下面的函数按需返回一个无穷的整数列表：
```ts
function* infiniteSequence() {
    var i = 0;
    while(true) {
        yield i++;
    }
}

var iterator = infiniteSequence();
while (true) {
    console.log(iterator.next()); // { value: xxxx, done: false } forever and ever
}
```

当然，如果迭代器完成了，你将会得到结果`{ done: true}`，正如下面展示：
```ts
function* idMaker(){
  let index = 0;
  while(index < 3)
    yield index++;
}

let gen = idMaker();

console.log(gen.next()); // { value: 0, done: false }
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { done: true }
```

### 外部受控执行

这是生成器函数最令人激动的一部分。它基本上允许一个函数暂停他的执行并传递函数执行的剩余部分的控制（命运）给调用者。

一个生成器函数在你调用它的时候不执行。他只是创建一个生成器对象。考虑下面的例子，使用一个例子执行：
```ts
function* generator(){
    console.log('Execution started');
    yield 0;
    console.log('Execution resumed');
    yield 1;
    console.log('Execution resumed');
}

var iterator = generator();
console.log('Starting iteration'); // This will execute before anything in the generator function body executes
console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

如果你运行这个，你将等到下面的输出：
```ts
$ node outside.js
Starting iteration
Execution started
{ value: 0, done: false }
Execution resumed
{ value: 1, done: false }
Execution resumed
{ value: undefined, done: true }
```

- 函数只执行一次，一旦生成器对象傻姑娘的`next`被调用
- 函数暂停，一旦遇到`yield`语句
- 当`next`被调用，函数恢复

> 因此，基本上生成器函数的执行可以通过生成器对象控制。

我们沟通使用生成器基本上使用生成器为迭代器返回的值。JavaScript 生成器一个非常强大的特性是他们允许双向沟通（需要注意）

- 你可以控制`yield`表达式的结果，通过使用`iterator.next(valueToInject)`
- 你可以使用`iterator.throw(error)`抛出一个异常，在`yield`表达式的点。

下面的例子展示了`iterator.next(valueToInject)`：
```ts
function* generator() {
    const bar = yield 'foo'; // bar may be *any* type
    console.log(bar); // bar!
}

const iterator = generator();
// Start execution till we get first yield value
const foo = iterator.next();
console.log(foo.value); // foo
// Resume execution injecting bar
const nextThing = iterator.next('bar');
```
因为`yield`返回传递到迭代器的`next`函数的参数，所有迭代器的`next`函数接受任何类型的参数，TypeScript 总是赋值`any`类型给`yield`操作符的结果（前面是`bar`）。

> 你需要自己强制将结果转化为你期待的，并确保这个类型的值被传递给 next（比如通过架构一个额外的类型强制层为你调用`call`）。如果强类型对你很重要，Nike 鞥想要完全避免双向通信，和验证依赖它的包（比如，redux-saga）。

下面的例子展示`iterator.throw(error)`：
```ts
function* generator() {
    try {
        yield 'foo';
    }
    catch(err) {
        console.log(err.message); // bar!
    }
}

var iterator = generator();
// Start execution till we get first yield value
var foo = iterator.next();
console.log(foo.value); // foo
// Resume execution throwing an exception 'bar'
var nextThing = iterator.throw(new Error('bar'));
```
因此，这里是总结

- `yield`允许一个生成器函数去暂停它的交流并传递控制到一个外部系统
- 一个外部心痛可以推送一个值到生成器函数体
- 一个外部系统可以抛出一个异常到生成器函数体

这有什么用？跳到[async/await]()章节，并指出来。