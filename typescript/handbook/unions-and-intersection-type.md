联合和交叉类型

到目前为止，手册已经覆盖了原子对象的类型。然而，随着你建模更多类型，你发现你在寻找工具去让你组合或者绑定现存类型，而不是从头开始创建他们。

交叉和联合类型是你可以组合类型的方式。

### 联合类型

有时候，你使用一个库，希望参数是`number`或者`string`。比如，私用下面的函数：
```
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: any) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // returns "    Hello world"
```

前面例子中的`padLeft`的问题是`padding`参数的类型是`any`。这意味着我们可以使用`number`或者`string`参数调用它，但是 TypeScript 可以使用它。

```
// passes at compile time, fails at runtime.
let indentedString = padLeft("Hello world", true);
```

在传统面向对象代码中，我们可能抽象两种类型，通过创建一个类型层级。尽管这更加精确，这有点过了。原始版本的`padLeft`中有一个很好的东西，那就是我们可以传递原始类型。这意味着使用简单明了。这种新的方式对我们没有帮助，如果我们只是想要使用一个已经存在的函数。

我们可以为`padding`使用联合类型而不是`any`。

```
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: string | number) {
  // ...
}

let indentedString = padLeft("Hello world", true);
Argument of type 'boolean' is not assignable to parameter of type 'string | number'.
```

一个联合类型描述一个值可以有多个类型。我们使用垂直条（`|`）去分离每一种类型，因此`number | string | boolean`是一个值的类型，可以是一个`number`，`string`，或者一个`boolean`。

### 联合常见域

如果我们有一个联合类型的值，我们只能访问在联合类型中通用的成员。

```
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

declare function getSmallPet(): Fish | Bird;

let pet = getSmallPet();
pet.layEggs();

// Only available in one of the two possible types
pet.swim();
Property 'swim' does not exist on type 'Bird | Fish'.
  Property 'swim' does not exist on type 'Bird'.
```

联合类型有点棘手，但是它只需要一点直觉来适应。如果值有类型`A | B`，我们只知道它有 `A`和`B`都有的成员。在这个例子中，`Bird`有成员`fly`。我们不能确定类型是`Bird | Fly`有一个`fly`方法。如果运行时变量是一个真实的`Fish`，则调用`pet.fly()`将会失败。

### 鉴别联合

使用联合的一个常见的技术是有一个单一的域，使用字面量类型让 TypeScript 向下转型到可能的当前类型。比如，我们将创建三种类型的联合，有一个单一的共享的域。

```
type NetworkLoadingState = {
  state: "loading";
};

type NetworkFailedState = {
  state: "failed";
  code: number;
};

type NetworkSuccessState = {
  state: "success";
  response: {
    title: string;
    duration: number;
    summary: string;
  };
};

// Create a type which represents only one of the above types
// but you aren't sure which it is yet.
type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState;
```
前面所有的类型都有一个域叫做`state`，同时他们还有自己的域：
| NetworkLoadingState |	NetworkFailedState | NetworkSuccessState |
| --- | --- | --- |
| state | state | state |
|       | code  | response |


给定的`state`域在`NetworkState`的每个类型内都非常常见 - 你的代码不需要检查就能很安全的访问他们。

因为`state`是一个字面量类型，你可以对比`state`值和相同的字符串，并且 TypeScript 将会知道哪一个类型当前被使用。

| NetworkLoadingState |	NetworkFailedState | NetworkSuccessState |
| --- | --- | --- |
| "loading" | "failed" | "success" |


在这种场景中，你可以使用`switch`语句去向下转型到运行时表示的类型：

```
type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState;

function networkStatus(state: NetworkState): string {
  // Right now TypeScript does not know which of the three
  // potential types state could be.

  // Trying to access a property which isn't shared
  // across all types will raise an error
  state.code;
Property 'code' does not exist on type 'NetworkState'.
  Property 'code' does not exist on type 'NetworkLoadingState'.

  // By switching on state, TypeScript can narrow the union
  // down in code flow analysis
  switch (state.state) {
    case "loading":
      return "Downloading...";
    case "failed":
      // The type must be NetworkFailedState here,
      // so accessing the `code` field is safe
      return `Error ${state.code} downloading`;
    case "success":
      return `Downloaded ${state.response.title} - ${state.response.summary}`;
  }
}
```

### 联合检查

我们希望编译器告诉我们当我们未覆盖所有联合变体的时候。比如，如果我们添加`NetworkFromCachedState`到`NetworkState`，我们需要去更新`logger`：
```
type NetworkFromCachedState = {
  state: "from_cache";
  id: string
  response: NetworkSuccessState["response"]
}

type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState
  | NetworkFromCachedState;

function logger(s: NetworkState) {
  switch (s.state) {
    case "loading":
      return "loading request";
    case "failed":
      return `failed with code ${s.code}`;
    case "success":
      return "got response"
  }
}
```

有两种方式可以做到这个。第一种是打开`--strictNullChecks`并且指定给一个返回值：
```
function logger(s: NetworkState): string {
Function lacks ending return statement and return type does not include 'undefined'.
  switch (s.state) {
    case "loading":
      return "loading request";
    case "failed":
      return `failed with code ${s.code}`;
    case "success":
      return "got response"
  }
}
```

因为`switch`没有穷尽，TypeScript  意识到函数有时候可能返回`undefined`。如果你有一个明确的返回值类型`string`，你就会得到一个错误，因为返回值实际上是`string | undefined`。然而，这个方法有点傻，并且，此外，[--strictNullChecks]()不总是能和旧的代码工作。

第二种方法使用`never`类型，编译器用来检测穷尽：
```
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}

function logger(s: NetworkState): string {
  switch (s.state) {
    case "loading":
      return "loading request";
    case "failed":
      return `failed with code ${s.code}`;
    case "success":
      return "got response";
    default: 
      return assertNever(s)
Argument of type 'NetworkFromCachedState' is not assignable to parameter of type 'never'.
  }
}
```

这里，`assertNever`检查`s`是`never`类型 -- 所有其他场景--其他类型已经被移除。如果你忘记了一个场景，则`s`将会有一个真实类型，你将会得到一个类型错误。这个方法要求你去定一个额外的函数，但是它在你忘记他的时候更明显，因为错误信息包含丢失的名字。

### 交叉类型

交叉类型和联合类型很像，但是他们的使用非常不同。一个交叉类型绑定多个类型到一个。这允许你去添加已存在的类型去得到单一个的类型，拥有素偶有你需要的特性。比如，`Person & Serializable & Loggable`是一个类型，是`Person`和`Serialiable`和`Loggable`。这意味着这个类型的一个对象将会有这三个类型的所有成员。

比如，如果你有网络请求，使用相同的错误处理，则你可以分离合并的类型的错误处理到它自己的类型，表示一个单一的相应类型。

```ts
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

interface ArtistsData {
  artists: { name: string }[];
}

// These interfaces are composed to have
// consistent error handling, and their own data.

type ArtworksResponse = ArtworksData & ErrorHandling;
type ArtistsResponse = ArtistsData & ErrorHandling;

const handleArtistsResponse = (response: ArtistsResponse) => {
  if (response.error) {
    console.error(response.error.message);
    return;
  }

  console.log(response.artists);
};
```