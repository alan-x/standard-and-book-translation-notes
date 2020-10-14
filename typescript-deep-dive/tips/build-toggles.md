# 构建切换

JavaSript 项目基于他们运行的地方切换是很常见的。你可以简单的使用 webpack 做到这个，因为它支持根据环境变量消除死代码。

在`package.json`的`scripts`添加不同目标：
```ts
"build:test": "webpack -p --config ./src/webpack.config.js",
"build:prod": "webpack -p --define process.env.NODE_ENV='\"production\"' --config ./src/webpack.config.js",
```

当然，我假设你有`npm install webpack --save-dev`，现在你可以运行`npm run build:test`，等。

使用这个变量也非常简单：
```ts
/**
 * This interface makes sure we don't miss adding a property to both `prod` and `test`
 */
interface Config {
  someItem: string;
}

/**
 * We only export a single thing. The config.
 */
export let config: Config;

/**
 * `process.env.NODE_ENV` definition is driven from webpack
 *
 * The whole `else` block will be removed in the emitted JavaScript
 *  for a production build
 */
if (process.env.NODE_ENV === 'production') {
  config = {
    someItem: 'prod'
  }
  console.log('Running in prod');
} else {
  config = {
    someItem: 'test'
  }
  console.log('Running in test');
}
```

> 我们使用`process.env.NODE_ENV`只是因为它在许多的 JavaScript 库中很方便，比如`React`。
