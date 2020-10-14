比如，当你想要去使用扩展其他库的 JavaScript 代码一起使用：
```
import { greeter } from "super-greeter";

// Normal Greeter API
greeter(2);
greeter("Hello world");

// Now we extend the object with a new function at runtime
import "hyper-super-greeter";
greeter.hyperGreet();
```

"super-greeter"的定义：
```ts
/*~ This example shows how to have multiple overloads for your function */
export interface GreeterFunction {
  (name: string): void
  (time: number): void
}

/*~ This example shows how to export a function specified by an interface */
export const greeter: GreeterFunction;
```

我们可以如下扩展现有的模块：
```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module plugin template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ On this line, import the module which this module adds to */
import { greeter } from "super-greeter";

/*~ Here, declare the same module as the one you imported above
 *~ then we expand the existing declaration of the greeter function
 */
export module "super-greeter" {
  export interface GreeterFunction {
    /** Greets even better! */
    hyperGreet(): void;
  }
}
```

这使用[声明合并]()

### ES 对模块插件的影响

一些插件在存在的模块上添加或者修改顶级导出。尽管在 CommonKS 和其他加载器中这是合法的，ES6 模块被认为是不可变的，并且这个模式不太可能。因为 TypeScript 是加载器不可知的，没有编译时间实施这个策略，但是开发者想要转化到一个 ES 模块加载器将会意识到这个。