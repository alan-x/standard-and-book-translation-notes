# Number

当你在任何编程语言需要处理数字，你需要小心语言特性是如何处理数字的。这是关于 JavaScript 的一些批评信息，你需要小心 

### 核心类型

JavaScript 只有一个数字类型。它是一个双精度 64 位`Number`。下面，我们将讨论他的局限性，和一个推荐的解决方案。

### 十进制

对于那些在其他语言中熟悉 double/float 的，你应该知道二进制浮点数并没有正确的映射十进制数。JavaScript 使用数字的一个小（和著名）例子显示在下面：
```ts
console.log(.1 + .2); // 0.30000000000000004
```
> 对于真的使用十进制数学，使用下面提到的`big.js`

### 整形

整形限制通过`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`建立的数字类型表示。
```ts
console.log({max: Number.MAX_SAFE_INTEGER, min: Number.MIN_SAFE_INTEGER});
// {max: 9007199254740991, min: -9007199254740991}
```

这个上下文的暗转指的是值不能被舍入错误的结果。

不安全的值是这些安全值`+1 / -1`，和任何加/减将会舍入的结果。
```ts
console.log(Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2); // true!
console.log(Number.MIN_SAFE_INTEGER - 1 === Number.MIN_SAFE_INTEGER - 2); // true!

console.log(Number.MAX_SAFE_INTEGER);      // 9007199254740991
console.log(Number.MAX_SAFE_INTEGER + 1);  // 9007199254740992 - Correct
console.log(Number.MAX_SAFE_INTEGER + 2);  // 9007199254740992 - Rounded!
console.log(Number.MAX_SAFE_INTEGER + 3);  // 9007199254740994 - Rounded - correct by luck
console.log(Number.MAX_SAFE_INTEGER + 4);  // 9007199254740996 - Rounded!
```

为了检测安全，可以使用 ES6 的`Number.isSafeInteger`：
```ts
// Safe value
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true

// Unsafe value
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)); // false

// Because it might have been rounded to it due to overflow
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 10)); // false
```

> JavaScript 最终将会支持[BigInt]()。现在，如果你想要任何精度的整形，使用下面提到的`big.js`

### big.js

当你为金融计算使用数学的时候（比如。GST 计算，金额的分，等），使用类似[big.js]()类似的库，他是为了以下设计：

- 完美的十进制数学
- 安全的整形值之外

安装很简单：
```ts
npm install big.js @types/big.js
```

快速使用例子：
```ts
import { Big } from 'big.js';

export const foo = new Big('111.11111111111111111111');
export const bar = foo.plus(new Big('0.00000000000000000001'));

// To get a number:
const x: number = Number(bar.toString()); // Loses the precision
```

> 不要为 UI/性能敏感的目的使用这个库，比如图标，画布绘制等。

### NaN

桑一些数字计算不能被有效数字表示的时候，JavaScript 返回一个特定的`NaN`值。一个典型的例子是虚数。
```ts
console.log(Math.sqrt(-1)); // NaN
```

注意：相等检测无法用在`NaN`值。使用`Number.isNaN`替代：
```ts
// Don't do this
console.log(NaN === NaN); // false!!

// Do this
console.log(Number.isNaN(NaN)); // true
```

### Infinity

Number 中范围之外的值可以作为静态`Number.MAX_VALUE`和`-Number.MAX_VALUE`值访问。
```ts
console.log(Number.MAX_VALUE);  // 1.7976931348623157e+308
console.log(-Number.MAX_VALUE); // -1.7976931348623157e+308
```

精度不变的范围之外的值被限制，比如
```ts
console.log(Number.MAX_VALUE + 1 == Number.MAX_VALUE);   // true!
console.log(-Number.MAX_VALUE - 1 == -Number.MAX_VALUE); // true!
```

精度变化的范围之外的值被解析为特定的值`Infinity`/`-Infinity`，比如：
```ts
console.log(Number.MAX_VALUE + 10**1000);  // Infinity
console.log(-Number.MAX_VALUE - 10**1000); // -Infinity
```

当然，这些指定的无限的值也出现在任何需要它的地方，比如：
```ts
console.log( 1 / 0); // Infinity
console.log(-1 / 0); // -Infinity
```

你可以手动使用这些`Infinity`值或者使用`Number`类的静态成员，如下显示：
```ts
console.log(Number.POSITIVE_INFINITY === Infinity);  // true
console.log(Number.NEGATIVE_INFINITY === -Infinity); // true
```

幸运的是比较符（`<` / `>`）可以可靠的额处理无限值：
```ts
console.log( Infinity >  1); // true
console.log(-Infinity < -1); // true
```

### 无穷小
Number 中最小的非0值可以通过静态`Number.MIN_VALUE`可用：
```ts
console.log(Number.MIN_VALUE);  // 5e-324
```
小于`MIN_VALUE`（“下溢值”）的值被转化为 0。
```ts
console.log(Number.MIN_VALUE / 10);  // 0
```
> 深入直觉：就像大于`Number.MAX_VALUE`的值被限制到 INFINITY，小于`Number.MIN_VALUE`的值被限制到`0`