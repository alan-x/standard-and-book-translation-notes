# 动态导入表达式

动态导入表达式是 ECMAScript 新特性的一部分，允许用户去异步请求一个模块，在你的程序的任意点。TC39 JavaScript 委员会已经有了这个提案，现在在 stage 3，它叫做[JavaScript import() 提案]()。

或者，webpack 打包器有一特特性叫做[Code Splitting]()，允许你去分离你的包到块中，可以在之后的时间异步下载。比如，这允许先提供一个最小化启动包，然后去异步加载额外特性。

很自然会认为（）[TypeScript 2.4 动态导入表达式]()将自动产生包块并自动代码分离你的 JS 最终包。但是，这并不是如看到的那么简单，因为它依赖于我们正在使用的 tsconfig.json 配置。

webpack 代码分离支持两种类似的技术去达到这个目的：使用 import()（预获取，ECMAScript 提案）和 require.ensure()（遗留的，webpack 指定的）。这意味着期待 TypeScript 输出是留下 import() 语句而不是转化它到其他东西。

来看看怎样配置 webpack + TypeScript 2.4 的例子。

在下面的代码，我希望懒加载库 moment，但是我也对代码分割感兴趣，这意味着，让 moment 库在分离的 JS 块中（JavaScript 文件），至于哦当需要的时候才加载。

```ts
import(/* webpackChunkName: "momentjs" */ "moment")
  .then((moment) => {
      // lazyModule has all of the proper types, autocomplete works,
      // type checking works, code references work \o/
      const time = moment().format();
      console.log("TypeScript >= 2.4.0 Dynamic Import Expression:");
      console.log(time);
  })
  .catch((err) => {
      console.log("Failed to load moment", err);
  });
```

这是 tsconfig.json:
```ts
{
    "compilerOptions": {
        "target": "es5",                          
        "module": "esnext",                     
        "lib": [
            "dom",
            "es5",
            "scripthost",
            "es2015.promise"
        ],                                        
        "jsx": "react",                           
        "declaration": false,                     
        "sourceMap": true,                        
        "outDir": "./dist/js",                    
        "strict": true,                           
        "moduleResolution": "node",               
        "typeRoots": [
            "./node_modules/@types"
        ],                                        
        "types": [
            "node",
            "react",
            "react-dom"
        ]                                       
    }
}
```

重要笔记：

- 使用"module":"esnext" TypeScript 产生伪装的 import() 语句输入到 WebPack 代码分割。
- 了解更多信息阅读这个文章：[动态导入表达式和 webpack 2 代码分割和 TypeScript 2.4 的集成]()。

你可以在[这里查]()看完整的例子。