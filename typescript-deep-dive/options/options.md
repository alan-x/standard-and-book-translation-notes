# 选项

TypeScript 有时候会阻止你做一些开箱即用的东西，比如，使用还没有的声明的变量（当然你可以为外部系统使用一个声明文件）。

也就是说，传统编程语言通过类型系统，在允许和不允许之间有一个严格的边界。TypeScript 在这一点不同，它允许你控制这个滑块。这真的允许你尽可能的安全使用你知道的和你喜欢的 JavaScript。有很多的编译器选项去精确控制这个滑块，来看一看吧，

### 布尔选项
`boolean`的`compilerOptions`可以指定为`tsconfig.json`的`compilerOptions`：
```ts
{
    "compilerOptions": {
        "someBooleanOption": true
    }
}
```
或者在命令行中：
```ts
tsc --someBooleanOption
```
> 这些默认都是`false`

点击[这里]()查看所有的编译器选项。