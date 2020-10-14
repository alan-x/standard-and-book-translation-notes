### 创建数组

创建一个空的数组非常简单：
```ts
const foo: string[] = [];
```

如果你想要去创建一个预填充一些内容的数组，可以使用 ES6`Array.prototype.fill`：
```ts
const foo: string[] = new Array(3).fill('');
console.log(foo); // ['','',''];
```

如果你想要使用调用创建一个预定义长度的数组，你可以使用展开操作符：
```ts
const someNumbers = [...new Array(3)].map((_,i) => i * 10);
console.log(someNumbers); // [0,10,20];
```