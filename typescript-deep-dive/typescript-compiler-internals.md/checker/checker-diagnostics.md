[已校对]
# 检查器诊断

`initializeTypeChecker`内存在下面的代码：
```ts
// Initialize global symbol table
forEach(host.getSourceFiles(), file => {
    if (!isExternalModule(file)) {
        mergeSymbolTable(globals, file.locals);
    }
});
```

基本上合并了所有的`global`符号到`let globals: SymbolTable = {};`（在`createTypeChecker`）SymbolTable。`mergeSymbolTable`主要调用`mergeSymbol`。