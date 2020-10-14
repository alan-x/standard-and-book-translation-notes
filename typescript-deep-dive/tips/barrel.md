### barrel

一个 barrel 是将多个模块的导出卷到一个单独的模块的方式。这个 barrel 他自己是一个模块文件，然后重新导出从其他模块选中的导出。

想象一个库中下面的类解构：
```ts
// demo/foo.ts
export class Foo {}

// demo/bar.ts
export class Bar {}

// demo/baz.ts
export class Baz {}
```

不实用 barrel，一个消费者可能需要三个导入语句：
```ts
import { Foo } from '../demo/foo';
import { Bar } from '../demo/bar';
import { Baz } from '../demo/baz';
```

你可以选择添加一个 barrel `demo/index.ts`，包含下面：
```ts
// demo/index.ts
export * from './foo'; // re-export all of its exports
export * from './bar'; // re-export all of its exports
export * from './baz'; // re-export all of its exports
```

现在消费者可以从 barrel 导入它需要的：
```ts
import { Foo, Bar, Baz } from '../demo'; // demo/index.ts is implied
```

### 具名导出

与其导出`*`，你可以选择导出在一个名字下导出模块。比如，假设`baz.ts`有一个函数：
```ts
// demo/foo.ts
export class Foo {}

// demo/bar.ts
export class Bar {}

// demo/baz.ts
export function getBaz() {}
export function setBaz() {}
```

如果你更愿意从 demo 导出`getBaz`/`setBaz`，你可以将他们放在一个变量中，通过导入他们到一个名字，然后像下面一样导出这个名字：
```ts
// demo/index.ts
export * from './foo'; // re-export all of its exports
export * from './bar'; // re-export all of its exports

import * as baz from './baz'; // import as a name
export { baz }; // export the name
```

现在消费者看起来像：
```ts
import { Foo, Bar, baz } from '../demo'; // demo/index.ts is implied

// usage
baz.getBaz();
baz.setBaz();
// etc. ...
```