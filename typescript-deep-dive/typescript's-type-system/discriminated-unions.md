# 区分联合

### 区分联合

如果你有一个具有[字面量成员]()的类，则你可以使用这个成员去区分联合成员。

作为一个例子，假设有`Sequare`和`Rectangle`的联合，这里我们有一个成员`king`存在于联合成员，是一个特殊的字面量类型：
```ts
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
type Shape = Square | Rectangle;
```

如果你使用类型守卫风格检测（`==`，`===`，`!=`，`!==`）或者`switch`去区分属性（这里是`kind`），TypeScript 将会知道对象必须是这个类型，有指定的直面量并为你向下转型
```ts
function area(s: Shape) {
    if (s.kind === "square") {
        // Now TypeScript *knows* that `s` must be a square ;)
        // So you can use its members safely :)
        return s.size * s.size;
    }
    else {
        // Wasn't a square? So TypeScript will figure out that it must be a Rectangle ;)
        // So you can use its members safely :)
        return s.width * s.height;
    }
}
```

### 全面检测

经常你想要保证联合的所有成员有一些代码（行为）贯穿他们。
```ts
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}

// Someone just added this new `Circle` Type
// We would like to let TypeScript give an error at any place that *needs* to cater for this
interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;
```

作为一个例子，有些东西变化了：
```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    // Would it be great if you could get TypeScript to give you an error?
}
```
你可以通过简单添加一个落空并确保这个块中的类型推断和`never`类型兼容。比如，如果你添加了一个全面检测，你将得到一个漂亮的错误：
```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else {
        // ERROR : `Circle` is not assignable to `never`
        const _exhaustiveCheck: never = s;
    }
}
```
这强制你处理这个新的场景：
```ts
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else if (s.kind === "circle") {
        return Math.PI * (s.radius **2);
    }
    else {
        // Okay once more
        const _exhaustiveCheck: never = s;
    }
}
```

### switch

提示：当然你也可以在一个`switch`语句这么做：
```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        case "circle": return Math.PI * s.radius * s.radius;
        default: const _exhaustiveCheck: never = s;
    }
}
```

### strictNullChecks

如果使用 strictNullChecks 并执行全面检测，TypeScript kennel抱怨“并不是所有的代码路径都有返回值”。你可以通过返回`_exhaustiveCheck`变量沉默它（`never`类似）。因此：
```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        case "circle": return Math.PI * s.radius * s.radius;
        default:
          const _exhaustiveCheck: never = s;
          return _exhaustiveCheck;
    }
}
```

### 在全面检查中抛出

你可以编写一个接受`never`的函数（因此可以被推断为`never`的变量调用）然后抛出一个错误，如果他的内部已经执行了：
```ts
function assertNever(x:never): never {
    throw new Error('Unexpected value. Should have been never.');
}
```
使用 area 函数的例子：
```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
type Shape = Square | Rectangle;

function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        // If a new case is added at compile time you will get a compile error
        // If a new value appears at runtime you will get a runtime error
        default: return assertNever(s);
    }
}
```

### 回顾版本

假设你有一个这个形式的数据解构：
```ts
type DTO = {
  name: string
}
```

在你有一串的`DTO`之后，你意识到`name`是一个很坏的选择。你可以添加版本回顾，通过创建一个新的 DTO 使用数字字面量的联合（或者你使用的字符串）。标着为版本 0 作为`undefined`，并且如果你有`strictNullChecks`启用他，它将会工作：
```ts
type DTO = 
| { 
   version: undefined, // version 0
   name: string,
 }
| {
   version: 1,
   firstName: string,
   lastName: string, 
}
// Even later 
| {
    version: 2,
    firstName: string,
    middleName: string,
    lastName: string, 
} 
// So on
```

这类 DTO 的使用例子：
```ts
function printDTO(dto:DTO) {
  if (dto.version == null) {
      console.log(dto.name);
  } else if (dto.version == 1) {
      console.log(dto.firstName,dto.lastName);
  } else if (dto.version == 2) {
      console.log(dto.firstName, dto.middleName, dto.lastName);
  } else {
      const _exhaustiveCheck: never = dto;
  }
}
```

### Redux

使用这个的流行库是 redux。

这是[redux 的 gist]()使用 TypeScript 类型声明：
```ts
import { createStore } from 'redux'

type Action
  = {
    type: 'INCREMENT'
  }
  | {
    type: 'DECREMENT'
  }

/**
 * This is a reducer, a pure function with (state, action) => state signature.
 * It describes how an action transforms the state into the next state.
 *
 * The shape of the state is up to you: it can be a primitive, an array, an object,
 * or even an Immutable.js data structure. The only important part is that you should
 * not mutate the state object, but return a new object if the state changes.
 *
 * In this example, we use a `switch` statement and strings, but you can use a helper that
 * follows a different convention (such as function maps) if it makes sense for your
 * project.
 */
function counter(state = 0, action: Action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counter)

// You can use subscribe() to update the UI in response to state changes.
// Normally you'd use a view binding library (e.g. React Redux) rather than subscribe() directly.
// However, it can also be handy to persist the current state in the localStorage.

store.subscribe(() =>
  console.log(store.getState())
)

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

TypeScript 使用它让你安全的和输入错误打交道，增加重构能力并且自文档化代码。