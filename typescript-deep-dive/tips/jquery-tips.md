# jQuery 提示

注意：你需要为这些提示去安装`jquery.d.ts`文件


### 快速定义个新的插件

创建一个`jquery-foo.d.ts`：

```ts
interface JQuery {
  foo: any;
}
```

现在你可以使用`$('something').foo({whateverYouWant:'hello jquery plugin'})`。