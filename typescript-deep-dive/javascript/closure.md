[已校对]
# 闭包

JavaScript 得到的最好的东西是闭包。JavaScript 中的一个函数访问任何定义在外部范围的任何变量。闭包使用例子解释最好：
```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;

    function bar() {
        console.log(variableInOuterFunction); // Access a variable from the outer scope
    }

    // Call the local function to demonstrate that it has access to arg
    bar();
}

outerFunction("hello closure"); // logs hello closure!
```
你可以看到内部函数访问了外部范围的变量。外部函数的被内部函数关闭（包裹）。因此术语闭包。它本身的定义足够简单和非常直观。


现在是最棒的部分。内部函数可以访问外部范围的变量。甚至在外部函数返回之后。这是因为变量依旧被内部函数包裹。并且不依赖外部函数。再看一个例子：
```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;
    return function() {
        console.log(variableInOuterFunction);
    }
}

var innerFunction = outerFunction("hello closure!");

// Note the outerFunction has returned
innerFunction(); // logs hello closure!
```

### 为什么这很酷的原因
它允许你去简单组合对象，比如，revealing 模块模式：
```ts
function createCounter() {
    let val = 0;
    return {
        increment() { val++ },
        getVal() { return val }
    }
}

let counter = createCounter();
counter.increment();
console.log(counter.getVal()); // 1
counter.increment();
console.log(counter.getVal()); // 2
```
从高层次来说，这是让 Node.js 可能的东西（不要担心如果它无法立马在你大脑中单击。它最终会）：
```ts
// Pseudo code to explain the concept
server.on(function handler(req, res) {
    loadData(req.id).then(function(data) {
        // the `res` has been closed over and is available
        res.send(data);
    })
});
```