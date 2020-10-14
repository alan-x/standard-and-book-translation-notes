# 静态构造器

TypeScript`class`（类似 JavaScript `class`）不能有静态构造器。然而，你可以简单获得相同的效果，通过调用它：
```ts
class MyClass {
    static initialize() {
        // Initialization
    }
}
MyClass.initialize();
```