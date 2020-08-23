Promise/A+

**一个可靠的，可交互的 JavaScript promise 开放标准，由实现者实现，提供给实现者**

一个 pormise 表示一个异步操作的最终结果。和 primise 交互的最主要方式是通过他的 `then` 方法，它注册一个回调去接受 promise 最终的值，或者是 promise 不能被满足的原因。

这个规范详细描述了 `then` 方法的行为，提供一个可交互的基础，所有符合 Promise/A+ 的实现可以基于它去提供。因此，这个规范应该被认为非常稳定。尽管 Promise/A+ 组织可能偶尔通过向后兼容的改变去修订规范，来解决新发现的极端情况，我们只会在仔细考虑、讨论和测试之后才会集成大的或者不向后兼容的改变。

从历史上看，Promise/A+ 声明了早期 Promise/A 提案的行为，扩展它去覆盖实际行为，并缺省不足或者有问题的部分。

最终，核心的 Promise/A+ 规范没有解决怎样去创建，满足，拒绝 promise，而是聚焦于提供一个可交互的 `then` 方法。未来的配套规范中可能会涉及到这些主题。

### 1. 术语

1.1. "promise" 是一个对象或者函数，它有一个 `then` 方法，它的行为符合这个标准。
1.2. "thenable" 是一个对象或者函数，它定义了一个 `then` 方法。
1.3. "value" 是任意合法的 JavaScript 值（包含 `undefined`，一个 thenable，或者一个 promise）。
1.4. "exception" 是一个使用 `throw` 语句抛出的值。
1.5. "reason" 是一个值，指示为什么 promise 被拒绝。

### 2. 需求

#### 2.1 Promise 状态

一个 promise 必须在三个状态之一：pending，fullfilled，或者 rejected。

2.1.1. 当 pending 的时候，一个 promise：
    2.1.1.1. 可能转化为 fullfilled 或者 rejected 状态

2.1.2. 当 fullfilled 的时候，一个 promise：
    2.1.2.1. 必须不转化为任何其他状态。
    2.1.2.2. 必须有一个值，必须不能改变。

2.1.3. 当 rejected 的时候，一个 promise：
    2.1.3.1. 必须不转化为其他状态
    2.1.3.2. 必须有一个原因，必须不能改变。

这里，“必须不改变”意味着不能改变标识（比如，===），但是不意味着深层不可变。


### 2.2. then 方法

一个 promise 必须提供一个 then 方法去访问它当前或者最终的值或者原因。

一个 promise 的 then 方法接受两个参数：

```
promise.then(onFullfilled, onRejected)
```

2.2.1. onFullfilled 和 onRejected 都是可选的参数：
    2.2.1.1. 如果 onFullfilled 不是一个函数，他必须被忽略。
    2.2.1.2. 如果 onRejected 不是一个函数，他必须被忽略。
2.2.2. 如果 onFullfilled 是一个函数：
    2.2.2.1. 它必须在 promise 是 fullfilled 的时候调用，将 promise 的值作为它的第一个参数。
    2.2.2.2. 他必须不能在 promise 被 fullfilled 前调用。
    2.2.2.3. 它必须不能被调用超过一次。
2.2.3. 如果 onRejected 是函数：
    2.2.3.1. 它必须在 promise 被 rejected 之后被调用，将 promise 的原因作为他的第一个参数。
    2.2.3.2. 他必须不能在 promise 被 rejected 前被调用。
    2.2.3.3. 它必须不能调用超过一次。
2.2.4. onFullfilled 和 onRejected 必须不能被调用，直到异常上下文栈只包含平台代码。
2.2.5. onFullfilled 和 onRejected 必须作为一个函数被调用（比如，没有 this 值）。
2.2.6. then 在相同的 promise 中可能调用的多次。
    2.2.6.1. 如果/当 promise 是 fullfilled，所有的每个 onFullfilled 回调必须以他们发起对 then 的调用的顺序执行。
    2.2.6.2. 如果/当 promise 是 rejected，所有的每个 onRejected 回调必须以他们发起对 then 的调用的顺序执行。
2.2.7. then 必须返回一个 promise
    ```
    promise2 = promise1.then(onFullfilled, onRejected);
    ```
    2.2.7.1. 如果 onFullfilled 或者 onRejected 返回一个值 x，执行 Promise Resolution Procedure。
    2.2.7.2. 如果 onFullfilled 或者 onRejected 抛出一个异常 e，promise2 必须使用 e 作为原因 rejected。
    2.2.7.3. 如果 onFullfiiled 不是一个函数，并且 promise1 是 fullfilled，promise2 必须必须使用和 promise1 相同的值 fullfilled。
    2.2.7.4. 如果 onRejected 不是一个函数，并且 promise1 是 rejected，promise2 必须使用和 prmise1 相同的原因 rejected。


### 2.3. Promise 解决程序

promise 解决程序是一个抽象操作，接受一个 prmise 和一个值，我们定位为 [[Resolve]](promise, x)。如果 x 是一个 thenable，它尝试让 promise 接受 x 的状态，给予假设 x 表现的至少像个 prmise。否则，它使用值 x fullfill promise。

这个 thenable 的对待方式允许 promise 的实现交互，只要他们暴露一个 Promise/A+ 兼容的 then 方法。它也允许 Promise/A+ 实现去“吸收”不兼容的合理的 then 方法实现。

运行 [[Resolve]](promise, x)，执行下面步骤：

2.3.1. 如果 promise 和 x 引用相同的对象，使用 TypeError 作为原因拒绝 promise。
2.3.2. 如果 x 是一个 primise，接受它的状态：
    2.3.2.1. 如果 x 是 pending，promise 必须保持 pending，直到 x 是 fullfilled 或者 rejected。
    2.3.2.2. 如果/当 x 是 fullfilled，使用相同的值满足 promise。
    2.3.2.3. 如果/当 x 是 rejected，使用相同的原因拒接 promise。
2.3.3. 否则，如果 x 是一个对象或者函数，
    2.3.3.1. 让 then 成为 x.then。
    2.3.3.2. 如果获取 x.then 属性导致抛出一个异常 e，使用 e 作为原因拒绝 promise。
    2.3.3.3. 如果 then 是一个函数，将 x 作为 this 调用它，第一个参数 resolvePromise，第二个参数 rejectPromise，如果：
        2.3.3.3.1. 如果/当 resolvePromise 使用值 y 被调用，执行 [[Resolve]](promise, y)。
        2.3.3.3.2. 如果/当 rejectPromise 使用原因 r 被调用，使用 r 拒绝 promise。
        2.3.3.3.3. 如果 resolvePromise 和 rejectPromise 被调用，或者相同参数被多次调用，第一个调用采取程序，任何更多的调用被忽略。
        2.3.3.3.4. 如果调用 then 抛出一个异常 e。
            2.3.3.3.4.1. 如果 resolvePromise 或者 rejectPromise 被调用，忽略它。
            2.3.3.3.4.2. 否则，使用 e 作为原因拒绝 promise。
    2.3.3.4. 如果 then 不是一个函数，使用 x 满足 promise。
2.3.4. 如果 x 不是一个对象或者函数，使用 x 马努在 promise。

如果 promise 使用一个参与到循环的 thenable 链的 thenable 去解决，这样的 [[Resolve]](promise, thenable) 最终会导致 [[Resolve]](promise, thenable) 被再次调用，根据上面的算法将会导致无限循环。鼓励但不强求实现者去检测这种递归并使用 TypeErrpr 作为原因拒绝 promise。


### 备注
3.1. 这里“平台代码”意味着引擎，环境，和 promise 实现代码。在实践中，这要求确保 onFiullfilled 和 onRejected 异步执行，在事件循环转到 then 被调用，并使用一个新的栈。这可以使用类似 setTimeout 或者 setImmediate 之类的“宏任务”机制去实现，或者使用类似 MutationObserver 或者 process.nextTick 之类的“微任务”机制去实现。因为 promise 实现被认为是平台代码，它自身可能包含任务调度队列或者 “trampoline” in which the handlers are called
3.2. 也就是说，在严格模式，内部的 this 会是 undefined；在马虎模式，它将会是全局对象。
3.3. 实现可能允许 promise2 === promise1，提供的实现满足所有的需求。每一个实现者应该标记它可以产生 promise2 === promise1 并且给予什么条件。
3.4. 通常，它只直到 x 是一个真正的 prmise，如果它从当前的实现出来。这个语句允许实现指定，意味着接受公共的兼容 promise 状态。
3.5. 第一个存储 x.then 的引用的程序，则测试他的引用，然后调用这个引用，避免多次访问 x.then 属性。比如预防
3.6. 实现不应该对 thenable 链条的深度设置任何限制，假设这个递归的限制是无限的。只有真实的循环应该导致一个 TypeError；如果一个无限的链是被鼓励的，永远递归是正确的行为。

基于法律的最大可能，Promise/A+ 组织已经放弃了了所有的版本和 Promise/A+ 规范想干的权利。这个工作从以下