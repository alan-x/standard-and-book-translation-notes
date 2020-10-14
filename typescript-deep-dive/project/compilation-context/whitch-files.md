# 哪些文件？

使用`include`和`exclude`去指定文件/文件夹/全局。比如：
```ts
{
    "include":[
        "./folder"
    ],
    "exclude":[
        "./folder/**/*.spec.ts",
        "./folder/someSubFolder"
    ]
}
```

### 全局

- 对于全局：`**/*`（比如，简单使用`somefolder/**/*`）意味着所有的文件夹和任何文件（将会假设`.ts`/`.tsx`扩展，如果设置了`allowJs:true`，`.js`/`.jsx`也允许）

### `files`选项

或者，你可以使用`file`去明确：
```ts
{
    "files":[
        "./some/file.ts"
    ]
}
```

但是这不推荐，因为你需要保持更新它。使用`include`去添加包含的文件。
