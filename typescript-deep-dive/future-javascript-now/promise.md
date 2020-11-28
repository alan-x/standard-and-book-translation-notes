[已校对]
# Promise

### Promise
`Promise`类是存在于很多现代 JavaScript 引擎的东西，也可以简单通过[垫片](https://github.com/stefanpenner/es6-promise)。promise 的主要动机是引入异步风格的错误处理为 Async/Await 风格代码。

### 回调风格代码

为了完全感谢 promise，来展示一个简单的例子来正面只使用回调创建可信赖的异步代码的困难。考虑简单的场景，创作一个异步版本的从一个文件加载 JSON。一个同步版本可能非常简单：
```ts
import fs = require('fs');

function loadJSONSync(filename: string) {
    return JSON.parse(fs.readFileSync(filename));
}

// good json file
console.log(loadJSONSync('good.json'));

// non-existent file, so fs.readFileSync fails
try {
    console.log(loadJSONSync('absent.json'));
}
catch (err) {
    console.log('absent.json error', err.message);
}

// invalid json file i.e. the file exists but contains invalid JSON so JSON.parse fails
try {
    console.log(loadJSONSync('invalid.json'));
}
catch (err) {
    console.log('invalid.json error', err.message);
}
```
这个简单的`loadJSONSync`函数有三个行为，一个有效的返回值，一个文件系统错误或一个 JSON 转化错误。我们处理错误，使用简单 try/catch 去捕获错误，就像其他语言中的同步编程做的一样。现在创建一个好的异步版本的这个函数，使用简单错误检查逻辑的一个不错的初始尝试如下所示：
```ts
import fs = require('fs');

// A decent initial attempt .... but not correct. We explain the reasons below
function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}
```

足够简单，它接受一个回调，传递文件系统的任何错误到回调。如果没有文件系统错误，它返回`JSON.parse`的结果。当使用基于回调的异步函数的时候，需要记住几个点：

1. 永远不要调用回调两次
2. 永远不要抛出错误

然而，这个简单的函数无法提供这两点。实际上`JSON.parse`会抛出异常，如果传递了一个坏的 JSON，回调永远不会被调用，应用会崩溃。这显示在下面的例子：
```ts
import fs = require('fs');

// A decent initial attempt .... but not correct
function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}

// load invalid json
loadJSON('invalid.json', function (err, data) {
    // This code never executes
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

一个天真的尝试修复方案是包裹`JSON.parse`到一个 try catch，如下面的例子：
```ts
import fs = require('fs');

// A better attempt ... but still not correct
function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// load invalid json
loadJSON('invalid.json', function (err, data) {
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

然而，这个代码有一个不易察觉的 bug。如果不是回调（`cb`），而是`JSON.parse`，抛出一个错误，因为我们将它包裹在一个`try`/`catch`，`catch`执行，我们再一次调用了回调，回调被调用了两次，这显示在下面的例子：
```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// a good file but a bad callback ... gets called again!
loadJSON('good.json', function (err, data) {
    console.log('our callback called');

    if (err) console.log('Error:', err.message);
    else {
        // let's simulate an error by trying to access a property on an undefined variable
        var foo;
        // The following code throws `Error: Cannot read property 'bar' of undefined`
        console.log(foo.bar);
    }
});
```

```ts
$ node asyncbadcatchdemo.js
our callback called
our callback called
Error: Cannot read property 'bar' of undefined
```

这是因为我们的`loadJSON`函数错误的被包裹在一个`try`块。这是一个简单的课需要记住

> 简单课程：包含你的所有异步代码在一个 try/catch，除非你调用回调。

遵循这个简单的课程，我们有了一个完全功能的异步版本的`loadJSON`，如下显示：
```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) return cb(err);
        // Contain all your sync code in a try catch
        try {
            var parsed = JSON.parse(data);
        }
        catch (err) {
            return cb(err);
        }
        // except when you call the callback
        return cb(null, parsed);
    });
}
```

诚然，这不是很难去遵循，一旦你花一些时间完成它，但是尽管如此，有大量的样板代码需要去编写，仅仅为了好的错误处理。现在来看看一个更好的方式去跟踪异步 JavaScript，使用 promise。

### 创建一个 Promise

一个 promsise 可以是`pending`或者`fullfilled`或`rejected`

![https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png](https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png)

在`Promise`上调用`new`是一件小事（promise 构造函数）。promise 构造器传入`resolve`和`reject`函数设置 promise 状态：
```ts
const promise = new Promise((resolve, reject) => {
    // the resolve / reject functions control the fate of the promise
});
```

### 订阅 promise 的命运

promise 命运可以使用`.then`（如果 resolved）或者`.catch`（如果 rejected）订阅。

```ts
const promise = new Promise((resolve, reject) => {
    resolve(123);
});
promise.then((res) => {
    console.log('I get called:', res === 123); // I get called: true
});
promise.catch((err) => {
    // This is never called
});
```

```ts
const promise = new Promise((resolve, reject) => {
    reject(new Error("Something awful happened"));
});
promise.then((res) => {
    // This is never called
});
promise.catch((err) => {
    console.log('I get called:', err.message); // I get called: 'Something awful happened'
});
```

> promise 捷径
> - 快速创建已经 resolved 的promise：`Promise.resolve(resulc)`
> - 快速创建已经 reject 的promise：`Promise.reject(error)`

### Promise 的链能力

promise 的链能力是 promise 提供的核心好处。一旦你有一个 promise，从这个点开始，你使用`then`函数去创建一个 promise 链。

- 如果你从链内的任何函数返回一个 promise，`.then`会在值被 resolved 的时候调用：
```ts
Promise.resolve(123)
    .then((res) => {
        console.log(res); // 123
        return 456;
    })
    .then((res) => {
        console.log(res); // 456
        return Promise.resolve(123); // Notice that we are returning a Promise
    })
    .then((res) => {
        console.log(res); // 123 : Notice that this `then` is called with the resolved value
        return 123;
    })
```

- 你可以使用单独的`catch`聚合捕获任何前面链前面的错误处理：
```ts
// Create a rejected promise
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .then((res) => {
        console.log(res); // not called
        return 123;
    })
    .then((res) => {
        console.log(res); // not called
        return 123;
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
    });
```
- `catch`时间返回一个新的 promise（有效的创建一个新的 promsie 链接）：
```ts
// Create a rejected promise
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
    })
```
- 任何在一个`then`（或者`catch`）抛出的同步错误导致返回的 promise 失败：
```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('something bad happened'); // throw a synchronous error
        return 456;
    })
    .then((res) => {
        console.log(res); // never called
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
    })
```

- 只有有关（最近的尾巴）的`catch`被一个错误调用（因为 catch 启动一个新的 promise 链）。
```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('something bad happened'); // throw a synchronous error
        return 456;
    })
    .catch((err) => {
        console.log('first catch: ' + err.message); // something bad happened
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log('second catch: ' + err.message); // never called
    })
```

- 一个`catch`只在前面的链有错误的：
```ts
Promise.resolve(123)
    .then((res) => {
        return 456;
    })
    .catch((err) => {
        console.log("HERE"); // never called
    })
```

事实是：

- 错误跳转到尾巴`catch`（并跳过中间的`then`调用）
- 同步错误也会被任何尾巴`catch`捕获

有效的为我们提供一个异步编程范例，允许更好的错误处理，比原始回调。更多在下面


### TypeScript 和 promise

TypeScript 的一个好东西是它理解值的流，通过一个 promise 链接：
```ts
Promise.resolve(123)
    .then((res) => {
         // res is inferred to be of type `number`
         return true;
    })
    .then((res) => {
        // res is inferred to be of type `boolean`

    });
```

当然他也理解未包裹的任何函数调用可能返回一个 promise：
```ts
function iReturnPromiseAfter1Second(): Promise<string> {
    return new Promise((resolve) => {
        setTimeout(() => resolve("Hello world!"), 1000);
    });
}

Promise.resolve(123)
    .then((res) => {
        // res is inferred to be of type `number`
        return iReturnPromiseAfter1Second(); // We are returning `Promise<string>`
    })
    .then((res) => {
        // res is inferred to be of type `string`
        console.log(res); // Hello world!
    });
```

### 转化一个回调风格函数去返回一个 promise

简单包裹一个函数调用到一个 promise，并且

- 如果错误出现，就`reject`
- 如果没问题，就`resolve`

比如，包裹`fs.readFile`
```ts
import fs = require('fs');
function readFileAsync(filename: string): Promise<any> {
    return new Promise((resolve,reject) => {
        fs.readFile(filename,(err,result) => {
            if (err) reject(err);
            else resolve(result);
        });
    });
}

```

最可靠的方式去做这个是手写它，不需要像前面的例子一样冗长，比如，转化`setTimeout`到一个承诺的`delay`函数非常简单：
```ts
const delay = (ms: number) => new Promise(res => setTimeout(res, ms));
```

注意，NodeJS 有一个方便的函数来为你做这个魔法`node style function => promise returning function`。
```ts
/** Sample usage */
import fs from 'fs';
import util from 'util';
const readFile = util.promisify(fs.readFile);
```

> webpack 支持`util`模块开箱即用，你也可以在浏览器中使用

如果你有一个 node 回调风格函数作为一个成员，确保`bind`它去保证有正确的`this`：
```ts
const dbGet = util.promisify(db.get).bind(db);
```

### 重新回到 JSON 例子

现在来回顾一下我们的`loadJSON`例子，使用 promise 编写异步版本。我们要做的就是作为一个 promise 读取文件内容，然后转化他们为 JSON，然后就完成了，这在下面展示：
```ts
function loadJSONAsync(filename: string): Promise<any> {
    return readFileAsync(filename) // Use the function we just wrote
                .then(function (res) {
                    return JSON.parse(res);
                });
}
```

使用（注意这和原始的在这个章节开始引入的`sync`版本如何相似）：
```ts
// good json file
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // never called
    })

// non-existent json file
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })

// invalid json file
    .then(function () {
        return loadJSONAsync('invalid.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```

这个函数更简单的原因是因为"`loadFile`（异步）+`JSON.parse`（同步）=>`catch`"结合通过 promise 链完成。当然，回调不是通过我们调用，而是被 promise 链调用，因此我们没有机会制造包裹它到一个`try/catch`的错误。

### 并行控制流

我们已经看到使用 promise 执行一个系列的异步任务多简单。基本上就是`then`的链调用。

然而，你可能潜在的想要运行一个系列的异步任务，然后使用所有这些的任务的结果。`Promise`提供了一个静态的`Promise.all`函数，你可以使用去等待`n`数量的 promise 完成。你提供它一个数组的`n`promise，它返回一个数组的`n` resolved 的值，下面我们显示同步链：
```ts
// an async function to simulate loading an item from some server
function loadItem(id: number): Promise<{ id: number }> {
    return new Promise((resolve) => {
        console.log('loading item', id);
        setTimeout(() => { // simulate a server delay
            resolve({ id: id });
        }, 1000);
    });
}

// Chained / Sequential
let item1, item2;
loadItem(1)
    .then((res) => {
        item1 = res;
        return loadItem(2);
    })
    .then((res) => {
        item2 = res;
        console.log('done');
    }); // overall time will be around 2s

// Concurrent / Parallel
Promise.all([loadItem(1), loadItem(2)])
    .then((res) => {
        [item1, item2] = res;
        console.log('done');
    }); // overall time will be around 1s
```

有时候，你想要运行一个序列的异步任务，但是你只需要一个任务完成。Promise 为这个场景提供了一个静态的`Promise.race`函数：
```ts
var task1 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 1000, 'one');
});
var task2 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 2000, 'two');
});

Promise.race([task1, task2]).then(function(value) {
  console.log(value); // "one"
  // Both resolve, but task1 resolves faster
});
```