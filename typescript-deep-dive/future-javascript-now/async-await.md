[已校对]
# Async Await

> [一个专业 egghead 视频课程覆盖这相同的物料](https://egghead.io/courses/async-await-using-typescript)

作为一个思想实验，想象下面：一个方式去告诉 JavaScript 运行时在`await`关键字是去暂停代码执行，当用在一个 promise 并只恢复一次（并且如果）promise 从被处理的函数返回：
```ts
// Not actual code. A thought experiment
async function foo() {
    try {
        var val = await getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
}
```

当 promise 完成的时候，执行继续，
- 如果它是 fulfilled，则等待返回的值
- 如果他是 rejected，一个错误将会同步抛出，这样我们可以捕获

这突然（并且魔幻的）让异步程序和同步程序一样简单。这个思想实验需要三个东西：
- 任意暂停函数执行。
- 任意在一个函数内部放置一个值
- 任意一个函数内部抛出一个异常

这就是生成器允许我们做的！这个思想实验非常真实，他就是`async`/`await`。在下面，它只是使用生成器。

### 生成的 JavaScript

你不需要去理解这个，但是这很简单，如果你[读过生成器](https://basarat.gitbook.io/typescript/future-javascript/generators)。函数`foo`可以简单如下包裹：
```ts
const foo = wrapToReturnPromise(function* () {
    try {
        var val = yield getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
});
```
`wrapToReturnPromise`只是执行设给你撑起函数去获取`generator`然后使用`generator.next()`，如果值是一个`promise`，他可能`then`+`catch` promise并取决于结果调用`generator.next(result)`或`generator.throw(error)`。感谢它。

### TypeScript 中支持的 Async Await

Async - Await 已经被[TypeScript 从版本 1.7](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-7.html)开始支持。异步函数使用异步关键字作为前缀；await 暂停执行，直到一个异步函数返回 promise 被 fullfilled，并从 Promise 返回解包的值。它值支持目标 es6 直接转义到 ES6 生成器。

TypeScript 2.1 [添加了 ES3 和 ES5 运行时的能力，](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html)意味着你可以自由使用它，不用关心你使用的环境。注意到我们可以使用 async/await 很重要，很多浏览器都你吃，当然，有全局添加的 Promise 的 polyfill。

来看一个例子，看看这个代码，指出 TypeScript async / await 声明如何工作：
```ts
function delay(milliseconds: number, count: number): Promise<number> {
    return new Promise<number>(resolve => {
            setTimeout(() => {
                resolve(count);
            }, milliseconds);
        });
}

// async function always returns a Promise
async function dramaticWelcome(): Promise<void> {
    console.log("Hello");

    for (let i = 0; i < 5; i++) {
        // await is converting Promise<number> into number
        const count: number = await delay(500, i);
        console.log(count);
    }

    console.log("World!");
}

dramaticWelcome();
```

转化为 ES6（--taget es6）
```ts
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : new P(function (resolve) { resolve(result.value); }).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
function delay(milliseconds, count) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(count);
        }, milliseconds);
    });
}
// async function always returns a Promise
function dramaticWelcome() {
    return __awaiter(this, void 0, void 0, function* () {
        console.log("Hello");
        for (let i = 0; i < 5; i++) {
            // await is converting Promise<number> into number
            const count = yield delay(500, i);
            console.log(count);
        }
        console.log("World!");
    });
}
dramaticWelcome();
```

你可以在[这里](https://cdn.rawgit.com/basarat/typescript-book/705e4496/code/async-await/es6/asyncAwaitES6.js)查看完整例子

转化为 ES5（--target es5）
```ts
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : new P(function (resolve) { resolve(result.value); }).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
var __generator = (this && this.__generator) || function (thisArg, body) {
    var _ = { label: 0, sent: function() { if (t[0] & 1) throw t[1]; return t[1]; }, trys: [], ops: [] }, f, y, t, g;
    return g = { next: verb(0), "throw": verb(1), "return": verb(2) }, typeof Symbol === "function" && (g[Symbol.iterator] = function() { return this; }), g;
    function verb(n) { return function (v) { return step([n, v]); }; }
    function step(op) {
        if (f) throw new TypeError("Generator is already executing.");
        while (_) try {
            if (f = 1, y && (t = y[op[0] & 2 ? "return" : op[0] ? "throw" : "next"]) && !(t = t.call(y, op[1])).done) return t;
            if (y = 0, t) op = [0, t.value];
            switch (op[0]) {
                case 0: case 1: t = op; break;
                case 4: _.label++; return { value: op[1], done: false };
                case 5: _.label++; y = op[1]; op = [0]; continue;
                case 7: op = _.ops.pop(); _.trys.pop(); continue;
                default:
                    if (!(t = _.trys, t = t.length > 0 && t[t.length - 1]) && (op[0] === 6 || op[0] === 2)) { _ = 0; continue; }
                    if (op[0] === 3 && (!t || (op[1] > t[0] && op[1] < t[3]))) { _.label = op[1]; break; }
                    if (op[0] === 6 && _.label < t[1]) { _.label = t[1]; t = op; break; }
                    if (t && _.label < t[2]) { _.label = t[2]; _.ops.push(op); break; }
                    if (t[2]) _.ops.pop();
                    _.trys.pop(); continue;
            }
            op = body.call(thisArg, _);
        } catch (e) { op = [6, e]; y = 0; } finally { f = t = 0; }
        if (op[0] & 5) throw op[1]; return { value: op[0] ? op[1] : void 0, done: true };
    }
};
function delay(milliseconds, count) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            resolve(count);
        }, milliseconds);
    });
}
// async function always returns a Promise
function dramaticWelcome() {
    return __awaiter(this, void 0, void 0, function () {
        var i, count;
        return __generator(this, function (_a) {
            switch (_a.label) {
                case 0:
                    console.log("Hello");
                    i = 0;
                    _a.label = 1;
                case 1:
                    if (!(i < 5)) return [3 /*break*/, 4];
                    return [4 /*yield*/, delay(500, i)];
                case 2:
                    count = _a.sent();
                    console.log(count);
                    _a.label = 3;
                case 3:
                    i++;
                    return [3 /*break*/, 1];
                case 4:
                    console.log("World!");
                    return [2 /*return*/];
            }
        });
    });
}
dramaticWelcome();
```
你可以在[这里](https://cdn.rawgit.com/basarat/typescript-book/705e4496/code/async-await/es5/asyncAwaitES5.js)查看完整例子

注意：对于两个目标场景，我们需要去确保我们的运行时有一个 ECMAScript-兼容的 Promise 全局可用。这可能会为 Primise 获取一个 polyfill。我们也需要去确保 TypeScript 知道 Promise 存在，通过这只我们的 lib 标签为一些类似“dom”，“es2015”或者“dom”，“es2015.promise”，“es5”。我们可以在[这里](https://kangax.github.io/compat-table/es6/#test-Promise)看到浏览器做了什么支持 Priomise（原生和垫片）。
