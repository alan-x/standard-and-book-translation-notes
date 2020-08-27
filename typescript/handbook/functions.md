函数

函数是 JavaScript 中构建任何引用的基本块。他们是你构建抽象层，模拟类，信息隐藏，和模块的方式。在 TypeScript 中，尽管有类，命名空间，和模块，函数依旧扮演描述所做的事情的核心角色。TypeScript 也添加了一些能力到标准 JavaScript 函数去确保他们更容易操作。

### 函数

一开始，就像 JavaScript，TypeScript 函数可以被创建为具名函数或者匿名函数。这允许你为你的应用去选择最适合的方式，不管你是在为一个 API 构建一个函数列表，或者传递一个一次性函数到另一个函数。

为了快速概括这两种方式在 JavaScript 中的样子：
```
// Named function
function add(x, y) {
  return x + y;
}

// Anonymous function
let myAdd = function (x, y) {
  return x + y;
};
```

就像在 JavaScript 中，函数可以引用函数体外的变量。当他们这么做的时候，他们说是补货这些变量。尽管理解这是怎么工作的（以及当使用这种技术的权衡）是在本篇文章范围之外，对这个机制的工作原理的稳定理解是操作 JavaScript 和 TypeScript 很重要的一块。

```
let z = 100;

function addToZ(x, y) {
  return x + y + z;
}
```

### 函数类型


### 为函数添加类型

让我们给前面的简单例子添加类型：
```
function add(x: number, y: number): number {
  return x + y;
}

let myAdd = function (x: number, y: number): number {
  return x + y;
};
```

我们可以给每一个参数添加类型，然后给函数自身添加返回类型。TypeScript 可以通过返回语句指出返回类型，因此我们在很多场景下可以可选的关闭这个。

### 编写函数类型

现在我们为函数添加了类型，现在来编写函数的完整类型，通过查看函数类型的每一个部分。

```
let myAdd: (x: number, y: number) => number = function (
  x: number,
  y: number
): number {
  return x + y;
};
```

当参数类型开始的时候，就只考虑给函数的有效类型，无视你给函数类型参数的名字。

第二部分是返回类型。我们清晰的表明哪一个是返回类型，通过使用一个箭头（`=>`）在参数和返回类型之间。就像前面提到的，这是函数类型必须的一部分，因此，如果函数不需要返回值，你可以使用`void`替代离开他。

值得注意的是，只有参数和返回类型构成函数类型，捕获变量不反映在类型上。实际上，捕获变量是“隐藏状态”的一部分，不构成他的 API。

### 类型推理

从这些例子可以看出，你可能主要到 TypeScript 编译器可以指出类型，就选你只指出等号的一边的类型：
```
// The parameters 'x' and 'y' have the type number
let myAdd = function (x: number, y: number): number {
  return x + y;
};

// myAdd has the full function type
let myAdd2: (baseValue: number, increment: number) => number = function (x, y) {
  return x + y;
};
```

这叫做“上下文类型”，类型推断的一种。这帮助减少保持你的程序类型化的工作量。


### 可选和默认参数

在 TypeScript，每一个参数每一个桉树都假设被函数需要。这不意味着他不能给定`null`或者`undefined`，相反，当函数被调用，编译器将检查用户用户提供一个值给每一个参数。编译器也假设这些参数是唯一的参数，将会传递给函数。简而言之，给一个函数的参数数量匹配函数期待的数量。
```
function buildName(firstName: string, lastName: string) {
  return firstName + " " + lastName;
}

let result1 = buildName("Bob"); // error, too few parameters
Expected 2 arguments, but got 1.
let result2 = buildName("Bob", "Adams", "Sr."); // error, too many parameters
Expected 2 arguments, but got 3.
let result3 = buildName("Bob", "Adams"); // ah, just right
```

在 JavaScript，每一个参数都是可选的，用户可能停止使用，只要他们觉得适合。当他们这么做的时候，他们的值是`undefined`。我们可以在 TypeScript 中得到这个功能，通过在我们想要可选的参数后面添加`?`。比如，假设我们想要前面的 lastName 参数可选：
```
function buildName(firstName: string, lastName?: string) {
  if (lastName) return firstName + " " + lastName;
  else return firstName;
}

let result1 = buildName("Bob"); // works correctly now
let result2 = buildName("Bob", "Adams", "Sr."); // error, too many parameters
Expected 1-2 arguments, but got 3.
let result3 = buildName("Bob", "Adams"); // ah, just right
```

任何可选参数必须跟随在必须参数之后。如果我们想要让 firstname 可选，而不是 lastName，我们需要改变参数在函数中的顺序，将 firstName 放在列表的最后。

在 TypeScript，我们也可以设置一个值，如果用户没有赋值，参数将会被赋值。或者如果用户传递`undefined`在他的位置。这叫做默认初始值。使用前面的例子并more lastName 为`"Smith"`。
```
function buildName(firstName: string, lastName = "Smith") {
  return firstName + " " + lastName;
}

let result1 = buildName("Bob"); // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined); // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr."); // error, too many parameters
Expected 1-2 arguments, but got 3.
let result4 = buildName("Bob", "Adams"); // ah, just right
```

默认初始参数跟随在所有必须参数之后，被认为是可选的，就像可选参数，可以被省略，当调用他们根子的函数。这意味着可选参数和后缀默认参数将共享通用的类型，因此：
```
function buildName(firstName: string, lastName?: string) {
  // ...
}
```
和
```
function buildName(firstName: string, lastName = "Smith") {
  // ...
}

```
共享相同的类型`(firstName: string, lastName?: string) => string`。`lastName`的默认值出现在类型，只有在参数可选的时候。

和空白可选参数不一样，默认初始参数不需要出现在必须参数之后。如果一个默认初始参数出现在必须参数之前，用户需要明确传递`undefined`去获取默认初始值。比如，我们可能编写我们最新的例子，只在`fistName`上初始化默认值：
```
function buildName(firstName = "Will", lastName: string) {
  return firstName + " " + lastName;
}

let result1 = buildName("Bob"); // error, too few parameters
Expected 2 arguments, but got 1.
let result2 = buildName("Bob", "Adams", "Sr."); // error, too many parameters
Expected 2 arguments, but got 3.
let result3 = buildName("Bob", "Adams"); // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams"); // okay and returns "Will Adams"
```

### 剩余参数

必须的，可选的，和默认的参数都有一个相同的东西：他们一次只能同时沟通一个参数。有时候，你像作为组操作多个参数，或者你可能不知道一个函数将会接受多少参数。在 JavaScript，你可以直接操作参数，使用`arguments`变量，它可以在任何函数题内访问。

在 TypeScript，你可以将这些变量聚合到一个变量：
```
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

// employeeName will be "Joseph Samuel Lucas MacKinzie"
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

剩余参数可以看作是可选参数集合。当传递参数给一个剩余参数，你可以使用你想要的数量，你甚至可以不传递。编译器将会构建一个参数数组给省略号（`...`）后面的名字，允许你去在你的函数内使用。
```
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;Try

```

### this

学习怎么在 JavaScript 中使用`this`是一道坎。因为 TypeScript 是 JavaScript 的超集，TypeScript 开发者也需要知道怎样使用`this`和识别什么时候使用错误。幸好，TypeScript 使用两种技术让你捕获不正确shying的`this`。然而，如果你需要学习怎么在 JavaScript 中使用`this`，先阅读 Yehuda Katz's 的[理解 JavaScript 函数调用和“this”]()。Yehuda 的文章解释`this`内部原理解释的很棒，因此我们只在这里覆盖基本的。

### this 和箭头函数

在 JavaScript 中，`this`是一个变量，当函数被调用的时候被设置。这让他成为非常强大并切灵活的特性，但是它来自于总是需要知道函数执行的上下文的成本。这是总所周知的混乱，特别是当返回一个函数或者传递一个函数作为参数的时候。

来看看一个例子：
```
let deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  createCardPicker: function () {
    return function () {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);

      return { suit: this.suits[pickedSuit], card: pickedCard % 13 };
    };
  },
};

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

注意`createCardPicker`是一个函数，它返回一个函数。如果我们尝试运行这个例子，我们将会得到一个错误，而不是期待的弹出窗。这是因为`createCardPicker`创造的函数中使用的`this`将会被设置为`window`，而不是我们的`deck`对象。因为我们在它自己身上调用`cardPicker()`。一个顶级非方法语法像这样调用将会为`this`使用`window`。（注意：在严格模式下，`this`将会是`undefined`而不是`window`）。

我们可以修复这个，通过确认函数函数包裹到正确的`this`，在我们返回函数到之后时候。这种方式，无视之后它怎么用，它将会依旧能够看到原始的`deck`对象。为了做到这个，我们改变函数的表达去使用 ECMAScript 6 箭头语法。箭头函数在函数创建的时候捕获`this`，而不是它被调用的shih：
```
let deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  createCardPicker: function () {
    // NOTE: the line below is now an arrow function, allowing us to capture 'this' right here
    return () => {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);

      return { suit: this.suits[pickedSuit], card: pickedCard % 13 };
    };
  },
};

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

甚至更好，TypeScript 将会警告你，当你犯这个错的时候，如果你传递`--noImplicitThis`标志给编译器。它将会指出`this`在`this.suits[pickedSuit] `是`any`类型。

### this 参数
不幸的是，`this.suits[pickedSuit]`的类型依旧是`any`。这是因为`this`来自对象字面量的函数表达式。为了修复这个，你剋提供一个明确的`this`参数。`this`参数是一个伪装参数，来自函数参数列表的第一个：
```
function f(this: void) {
  // make sure `this` is unusable in this standalone function
}

```

添加一对的借口到我们前面的例子，`Card`和`Deck`确保类型清晰并且重用简单：
```
interface Card {
  suit: string;
  card: number;
}

interface Deck {
  suits: string[];
  cards: number[];
  createCardPicker(this: Deck): () => Card;
}

let deck: Deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  // NOTE: The function now explicitly specifies that its callee must be of type Deck
  createCardPicker: function (this: Deck) {
    return () => {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);

      return { suit: this.suits[pickedSuit], card: pickedCard % 13 };
    };
  },
};

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

现在，TypeScript 知道`createCardPicker`期待在`Deck`对象上调用。这意味着`this`是`Deck`类型，不是`any`，因此`noImplicitThis`将不会报错。

### 回调中的 this 参数

你可能在回调中遇到这样的`this`错误，当你传递函数到一个库，它将会在之后执行他们。因为库将像调用一个普通函数一样调用你的回调，`this`将会是`undefined`。使用一些工作，你可以使用`this`参数去防止回调中的错误。首先，库作者需要去使用`this`声明回调类型：

```
interface UIElement {
  addClickListener(onclick: (this: void, e: Event) => void): void;
}

```

`this:void`意味着`addClickListener`期待`onClick`是一个函数，不需要一个`this`类型，其次，使用`this`声明你的调用代码：
```ts
class Handler {
  info: string;
  onClickBad(this: Handler, e: Event) {
    // oops, used `this` here. using this callback would crash at runtime
    this.info = e.message;
  }
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // error!

Argument of type '(this: Handler, e: Event) => void' is not assignable to parameter of type '(this: void, e: Event) => void'.
  The 'this' types of each signature are incompatible.
    Type 'void' is not assignable to type 'Handler'.
```

使用`this`声明，你让这明确，`onClickBad`必须被调用在`Handler`实例上。然后，TypeScript 将会发现`addClickListener`需要一个函数，`this:void`。为了修复这个错误，改变`this`类型：
```class Handler {
  info: string;
  onClickGood(this: void, e: Event) {
    // can't use `this` here because it's of type void!
    console.log("clicked!");
  }
}

let h = new Handler();
uiElement.addClickListener(h.onClickGood);

```

因为`onClickGood`指定他的`this`类型为`void`，它可以合法的传递给`addClickListener`。当然，这也意味着不能使用`this.info`。如果你都想要，你需要使用箭头函数：
```
class Handler {
  info: string;
  onClickGood = (e: Event) => {
    this.info = e.message;
  };
}
```

这有用是因为箭头函数使用外部的`this`，因此你可以传递他们给任何期待`this:void`。缺点是每一个箭头函数都创建一个 Handler 对象。方法，另一方面，只创建一次，并绑定到 Handler 的原型。他们在所有的 Handler 对象之间共享。

### 重载

JavaScript 本质上是一个非常动态的语言。一个单一的 JavaScript 函数根据传递进来的参数的形状返回不同类型的对象是很常见的：
```
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: any): any {
  // Check to see if we're working with an object/array
  // if so, they gave us the deck and we'll pick the card
  if (typeof x == "object") {
    let pickedCard = Math.floor(Math.random() * x.length);
    return pickedCard;
  }
  // Otherwise just let them pick the card
  else if (typeof x == "number") {
    let pickedSuit = Math.floor(x / 13);
    return { suit: suits[pickedSuit], card: x % 13 };
  }
}

let myDeck = [
  { suit: "diamonds", card: 2 },
  { suit: "spades", card: 10 },
  { suit: "hearts", card: 4 },
];

let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

这里，`pickCard`函数将会返回两个不同的东西，基于用户传递进来的东西。如果用户传递了一个对象表示 deck，函数将会选择 card。如果用户选择 card，我们告诉他们选择了哪个。但是我们如何描述这个到类型系统？

答案是提供多个函数函数类型给相同的函数，作为重载列表。这是编译器将会用来处理函数调用的。创建一个重载列表去描述我们`pickCard`接受和返回什么。

```
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: { suit: string; card: number }[]): number;
function pickCard(x: number): { suit: string; card: number };
function pickCard(x: any): any {
  // Check to see if we're working with an object/array
  // if so, they gave us the deck and we'll pick the card
  if (typeof x == "object") {
    let pickedCard = Math.floor(Math.random() * x.length);
    return pickedCard;
  }
  // Otherwise just let them pick the card
  else if (typeof x == "number") {
    let pickedSuit = Math.floor(x / 13);
    return { suit: suits[pickedSuit], card: x % 13 };
  }
}

let myDeck = [
  { suit: "diamonds", card: 2 },
  { suit: "spades", card: 10 },
  { suit: "hearts", card: 4 },
];

let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

使用这个改变，重载现在给我们的`pickCard`调用类型检查。

为了让编译器选择正确的检查，它允许一个相似的进程去底层 JavaScript。它查看重载列表，处理第一个重载，尝试去使用提供的参数去调用函数。如果它发现匹配，它选择这个重载作为正确的重载。因为这个原因，从具体到最不具体的重载是一种习惯。

注意，`function pickCard(x): any`块不是重载列表的一部分，因此它只有两个重载：一个接受一个对象，一个接受一个数字。调用`pickerCard`使用其他参数类型将会导致错误。