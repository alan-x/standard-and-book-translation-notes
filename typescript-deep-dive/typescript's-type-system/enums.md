# 枚举


- [枚举]()
- [数字枚举和数字]()
- [数字枚举和字符串]()
- [改变关联一个数字枚举的数字]()
- [枚举是开放的]()
- [数字枚举作为标志]()
- [字符串枚举]()
- [常量枚举]()
- [有静态函数的枚举]()


### 枚举

一个枚举是组织一个集合的关联的值的方式。很多其他的编程语言（C/C#/Java）都有一个`enum`数据类型，但是 JavaScript 没有。然而，TypeScript 有。这是一个 TypeScript 枚举定义的例子：
```ts
enum CardSuit {
    Clubs,
    Diamonds,
    Hearts,
    Spades
}

// Sample usage
var card = CardSuit.Clubs;

// Safety
card = "not a member of card suit"; // Error : string is not assignable to type `CardSuit`
```

这些枚举值是`numbser`，因此今后我将叫他们数字枚举。

### 数字枚举和数字

TypeScript 是是基于数字的。这意味着数字可以赋值于枚举的一个实例，并且可以兼容于`number`。

```ts
enum Color {
    Red,
    Green,
    Blue
}
var col = Color.Red;
col = 0; // Effectively same as Color.Red
```

### 数字枚举和字符串

在我们深入查看了枚举之前，来看看它生成的 javaScript 吧，这里是一个 TypeScript 例子：
```ts
enum Tristate {
    False,
    True,
    Unknown
}
```
生成下列的 JavaScript：
```ts
var Tristate;
(function (Tristate) {
    Tristate[Tristate["False"] = 0] = "False";
    Tristate[Tristate["True"] = 1] = "True";
    Tristate[Tristate["Unknown"] = 2] = "Unknown";
})(Tristate || (Tristate = {}));
```

现在聚焦于`Tristate[Tristate["False"] = 0] = "False";`这一行。`Tristate["False"] = 0`是可以自解释的，比如设置`Traistate`变量的`"False"`为`0`。注意在 javaScript 中，赋值操作符返回所赋的值（这个场景是`0`）。因此 JavaScript 运行时下一个执行的东西是`Tristate[0] = "False"`。这意味着你可以使用`Tristate`变量去转化一个字符串版本的枚举为一个数字或者数字版本的枚举到一个字符串。这展示在下面：
```ts
enum Tristate {
    False,
    True,
    Unknown
}
console.log(Tristate[0]); // "False"
console.log(Tristate["False"]); // 0
console.log(Tristate[Tristate.False]); // "False" because `Tristate.False == 0`
```

### 改变关联一个数字枚举的数字

默认，枚举是基于`0`的，之后的子序列值自动递增 1。假设有如下例子：
```ts
enum Color {
    Red,     // 0
    Green,   // 1
    Blue     // 2
}
```

然而，你可以改变关联的任何枚举成员，通过赋值给他。这显示在下面，我们从 3 开始递增：
```ts
enum Color {
    DarkRed = 3,  // 3
    DarkGreen,    // 4
    DarkBlue      // 5
}
```

> 提示：我通常使用`=1`初始化第一个枚举值，它允许我对枚举值做安全的真实性检测。

### 枚举是开放的

枚举的一个极好的使用是作为`Flag`的能力。标志允许你去检测一个集合的条件中的某个条件是否为真。思考下面的例子，我们有关于动物一个集合的属性：
```ts
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3
}
```

这里，我们使用左移操作符去移动 1 到某个为止的比特得到位不相交的数字`0001`，`0010`，`0100`和`1000`（这是十进制的`1`，`2`，`4`，`8`，如果i好奇）。位操作符`|`（或）/`&`（和）/`～`（非）是你最好的朋友，当和标志一起用的时候。这展示在下面：
```ts
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
}
type Animal = {
    flags: AnimalFlags
}

function printAnimalAbilities(animal: Animal) {
    var animalFlags = animal.flags;
    if (animalFlags & AnimalFlags.HasClaws) {
        console.log('animal has claws');
    }
    if (animalFlags & AnimalFlags.CanFly) {
        console.log('animal can fly');
    }
    if (animalFlags == AnimalFlags.None) {
        console.log('nothing');
    }
}

let animal: Animal = { flags: AnimalFlags.None };
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws;
printAnimalAbilities(animal); // animal has claws
animal.flags &= ~AnimalFlags.HasClaws;
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws | AnimalFlags.CanFly;
printAnimalAbilities(animal); // animal has claws, animal can fly
```

这里：
- 我们使用`|=`去添加标志
- 组合`&=`和`~`去清理标志
- `|`去结合标志

> 注意：你可以在枚举定义中结合标志去创建便利的捷径，比如下面的`EndangeredFlyingClawedFishEating`：
```ts
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3,

    EndangeredFlyingClawedFishEating = HasClaws | CanFly | EatsFish | Endangered,
}
```

### 字符串枚举值

我们只看过成员值是`number`的枚举值。你其实也被允许使用字符串枚举成员。比如：
```ts
export enum EvidenceTypeEnum {
  UNKNOWN = '',
  PASSPORT_VISA = 'passport_visa',
  PASSPORT = 'passport',
  SIGHTED_STUDENT_CARD = 'sighted_tertiary_edu_id',
  SIGHTED_KEYPASS_CARD = 'sighted_keypass_card',
  SIGHTED_PROOF_OF_AGE_CARD = 'sighted_proof_of_age_card',
}
```

这可以更简单的处理和调试，因为他们提供有意义的/可调试的字符串值。

你可以使用这些值去简化字符串对比，比如：
```ts
// Where `someStringFromBackend` will be '' | 'passport_visa' | 'passport' ... etc.
const value = someStringFromBackend as EvidenceTypeEnum; 

// Sample use in code
if (value === EvidenceTypeEnum.PASSPORT){
    console.log('You provided a passport');
    console.log(value); // `passport`
}
```

### 常量枚举

如果你有如下的枚举定义：
```ts
enum Tristate {
    False,
    True,
    Unknown
}

var lie = Tristate.False;
```
`var lie = Tristate.False`被编译为 JavaScript `var lie = Tristate.False`（没错，输出和输入相同）。这意味着在执行的时候，运行时需要去查找`Tristate`，然后是`Tristate.False`。为了提高性能，我们可以标记`enum`为`const enum`。这显示在下面：
```ts
const enum Tristate {
    False,
    True,
    Unknown
}

var lie = Tristate.False;
```

生成 JavaScript：
```
var lie = 0;
```
比如，编译器：
1. 内联任何对枚举的使用（`0`而不是`Tristate.False`）。
2. 不生成任何 JavaScript 枚举定义（运行时没有`Tristate`变量），因为它使用内联。

#### 常量枚举 preserveConstEnums

内联有明显的性能优势。事实是在运行时没有`Tristate`变量，至死后编译器帮助你在运行时不生成没有使用的 JavaScript。然而，你可能想要编译器依旧生成 JavaScript 版本的枚举定义，为类似数字到字符串或者字符串到数字的查找。在这种场景中，你可以使用编译标志`--preserveConstEnums`，它将会依旧生成`var Tristate`定义，这样，如果你需要你就可以在运行时手动使用`Tristate["False"]`或者`Tristate[0]`。这对内联不会有任何影响。

### 有静态函数的枚举

你可以使用`enum`+`namespace`合并去添加静态方法到一个枚举。下面展示了一个例子，我们添加了一个静态成员`isBusinessDay`到一个枚举值`Weekday`：
```ts
enum Weekday {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday
}
namespace Weekday {
    export function isBusinessDay(day: Weekday) {
        switch (day) {
            case Weekday.Saturday:
            case Weekday.Sunday:
                return false;
            default:
                return true;
        }
    }
}

const mon = Weekday.Monday;
const sun = Weekday.Sunday;
console.log(Weekday.isBusinessDay(mon)); // true
console.log(Weekday.isBusinessDay(sun)); // false
```

#### 枚举是开放的

> 注意：开放的枚举值只有当你使用模块的时候才有关系。你应该使用枚举值。因此这个章节在最后。

这是一个枚举值生成的 JavaScript：
```ts
var Tristate;
(function (Tristate) {
    Tristate[Tristate["False"] = 0] = "False";
    Tristate[Tristate["True"] = 1] = "True";
    Tristate[Tristate["Unknown"] = 2] = "Unknown";
})(Tristate || (Tristate = {}));
```

我们已经解释了`Tristate[Tristate["False"] = 0] = "False";`。现在注意周围的代码`(function (Tristate) { /*code here */ })(Tristate || (Tristate = {}));`，特别是`(Tristate || (Tristate = {}));`。这不活了本地变量`TriState`，指向已经定义的`Tristate`值，或者使用一个空的`{}`对象初始化它。

这意味着你可以跨越多个文件分离（或者扩展）一个枚举定义。比如下面，我们分离了`Color`的定义到两块：
```ts
enum Color {
    Red,
    Green,
    Blue
}

enum Color {
    DarkRed = 3,
    DarkGreen,
    DarkBlue
}
```

注意你应该重新在枚举中连续的初始化第一个成员（这里是`DarkRed = 3`），让生成的代码不删除前面一个定义（）。TypeScript 将会警告你，如果你不这么做（错误信息`In an enum with multiple declarations, only one declaration can omit an initializer for its first enum element.`）。