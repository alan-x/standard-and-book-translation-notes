从 ECMAScript 2015 开始，`sumbol`是一个原子数据类型，就像`number`和`string`。

`symbol`值是通过调用`symbol`构造器创建的。
```
let sym1 = Symbol();

let sym2 = Symbol("key"); // optional string key
```

symbol 是不可变的，并且是唯一的。
```
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, symbols are unique
```

就像字符串，symbol 也可以作为对象属性的一部分
```
const sym = Symbol();

let obj = {
  [sym]: "value",
};

console.log(obj[sym]); // "value"
```

symbol 也可以和计算属性声明结合在一起声明对象属性和类成员。

```
const getClassNameSymbol = Symbol();

class C {
  [getClassNameSymbol]() {
    return "C";
  }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

### 总所周知的 symbol

除了用户定义的 symbol，还有总所周知的内置 symbol。内置 symbol 用于标示内部语言行为。

这是总所周知 symbol 的列表：


#### Symbol.hasInstance

一个方法，决定一个构造器对象承认一个对象是构造器的实例之一。通过 instanceof 操作符调用的语义。

#### Symbol.isConcatSpreadable

一个布尔值，指示一个对象应该映射为他的数组元素，通过 Array.prototype.concat。

#### Symbol.iterator

一个方法，返回一个对象的默认迭代器。通过 for-of 语句的调用的语义。
 
#### Symbol.match

一个正则表达式方法，匹配字符串和正则表达式。通过`String.prototype.match`方法调用

#### Symbol.replace
一个正则表达式方法，替代命中的字符串的子串。通过`String.prototype.replace`方法调用。


#### Symbol.search

一个正则表达式方法，返回命中正则表达式的字符串在字符串内的索引。通过`String.prototype.search`方法调用。

#### Symbol.species

一个函数值属性，是构造器函数，用于创建分离的对象。

#### Symbol.split

一个正则表达式方法，在命中正则表达式的索引分割字符串。通过`String.prototype.split`方法调用。

#### Symbol.toPrimitive

一个方法，转化一个对象到一个对应的原子值。通过`ToPrimitive`抽象操作调用。

#### Symbol.toStringTag

一个字符串值，用于创造一个对象的默认字符串描述。通过内建的`Object.prototype.toString`方法调用。

#### Symbol.unscopables

一个对象，他的自有属性名字是属性名字，被排除到'with'管来呢对象的环境绑定。