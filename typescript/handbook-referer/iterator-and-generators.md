迭代和生成器

### 可迭代的

一个对象被认为是可迭代的，如果它实现了[Symbol.iterator]()属性。一些像内置类型`Array`，`Map`，`Set`，`String`，`Int32Array`，`Uint32Array`，等，已经实现了自己的`Symbol.iterator`。

### for..of 语句

`for..of`遍历一个可遍历对象，调用对象的`Symbol.iterator`属性。这里是一个`for..of`遍历一个数组的例子。
```
let someArray = [1, "string", false];

for (let entry of someArray) {
  console.log(entry); // 1, "string", false
}

```

### for..of vs for..in 语句

`for..of`和`for..in`语句都遍历列表；然而他们遍历不同的值，`for..in`在遍历的时候返回一个键的列表，然而`for..of`返回被遍历的对象的一个数字属性的值的列表。

这是一个变现这个不同的例子：
```
let list = [4, 5, 6];

for (let i in list) {
  console.log(i); // "0", "1", "2",
}

for (let i of list) {
  console.log(i); // "4", "5", "6"
}
```

另一个不同是`for..in`操作在任何对象；它表现为检查这个对象上的属性的方式。`for..of`相反，主要对可遍历的对象的值感兴趣。`Map`和`Set`之类的内建的对象实现`Symbol.iterator`属性允许访问存储的值。

```ts
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
  console.log(pet); // "species"
}

for (let pet of pets) {
  console.log(pet); // "Cat", "Dog", "Hamster"
}

```

代码生成

目标是 ES5 和 ES3

当目标时候 ES5 或者 ES3 兼容的引擎的时候。迭代器只允许在`Array`类型的值上。使用`for..of`遍历非数组值是一种错误，即使没有数组值是想`Symbol.iterator`属性。

编译器会为`for..of`遍历生成一个简单的`for`遍历，比如：
```
let numbers = [1, 2, 3];
for (let num of numbers) {
  console.log(num);
}

```

将会生成为：
```ts
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
  var num = numbers[_i];
  console.log(num);
}
```

### 目标是 ECMAScript 2015 或者更高

当目标是一个 ECMAScript 2015 兼容的引擎，编译器将会生成`for..of`遍历去命中引擎内建的迭代器实现。
