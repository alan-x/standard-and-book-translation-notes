# 绑定器容器

一个 AST 节点可以是一个容器。这决定了节点和关联的符号将会拥有的`SymbolTables`。容器是一个抽象的概念（比如，没有关联的数据结构）。这个概念被一些东西驱动，一个是`ContainerFlags`枚举。函数`getContainerFlags`（在`binder.ts`）驱动这个标志并展示在下面：
```ts
function getContainerFlags(node: Node): ContainerFlags {
    switch (node.kind) {
        case SyntaxKind.ClassExpression:
        case SyntaxKind.ClassDeclaration:
        case SyntaxKind.InterfaceDeclaration:
        case SyntaxKind.EnumDeclaration:
        case SyntaxKind.TypeLiteral:
        case SyntaxKind.ObjectLiteralExpression:
            return ContainerFlags.IsContainer;

        case SyntaxKind.CallSignature:
        case SyntaxKind.ConstructSignature:
        case SyntaxKind.IndexSignature:
        case SyntaxKind.MethodDeclaration:
        case SyntaxKind.MethodSignature:
        case SyntaxKind.FunctionDeclaration:
        case SyntaxKind.Constructor:
        case SyntaxKind.GetAccessor:
        case SyntaxKind.SetAccessor:
        case SyntaxKind.FunctionType:
        case SyntaxKind.ConstructorType:
        case SyntaxKind.FunctionExpression:
        case SyntaxKind.ArrowFunction:
        case SyntaxKind.ModuleDeclaration:
        case SyntaxKind.SourceFile:
        case SyntaxKind.TypeAliasDeclaration:
            return ContainerFlags.IsContainerWithLocals;

        case SyntaxKind.CatchClause:
        case SyntaxKind.ForStatement:
        case SyntaxKind.ForInStatement:
        case SyntaxKind.ForOfStatement:
        case SyntaxKind.CaseBlock:
            return ContainerFlags.IsBlockScopedContainer;

        case SyntaxKind.Block:
            // do not treat blocks directly inside a function as a block-scoped-container.
            // Locals that reside in this block should go to the function locals. Otherwise 'x'
            // would not appear to be a redeclaration of a block scoped local in the following
            // example:
            //
            //      function foo() {
            //          var x;
            //          let x;
            //      }
            //
            // If we placed 'var x' into the function locals and 'let x' into the locals of
            // the block, then there would be no collision.
            //
            // By not creating a new block-scoped-container here, we ensure that both 'var x'
            // and 'let x' go into the Function-container's locals, and we do get a collision
            // conflict.
            return isFunctionLike(node.parent) ? ContainerFlags.None : ContainerFlags.IsBlockScopedContainer;
    }

    return ContainerFlags.None;
}
```

它只在绑定器的`bindChildren`函数内部调用，它设置一个节点为一个`container`，并且/或者一个`blockScopedContainer`取决于`getContainerFlags`函数的求值。函数`bindChildren`展示在下面：
```ts
// All container nodes are kept on a linked list in declaration order. This list is used by
// the getLocalNameOfContainer function in the type checker to validate that the local name
// used for a container is unique.
function bindChildren(node: Node) {
    // Before we recurse into a node's children, we first save the existing parent, container
    // and block-container.  Then after we pop out of processing the children, we restore
    // these saved values.
    let saveParent = parent;
    let saveContainer = container;
    let savedBlockScopeContainer = blockScopeContainer;

    // This node will now be set as the parent of all of its children as we recurse into them.
    parent = node;

    // Depending on what kind of node this is, we may have to adjust the current container
    // and block-container.   If the current node is a container, then it is automatically
    // considered the current block-container as well.  Also, for containers that we know
    // may contain locals, we proactively initialize the .locals field. We do this because
    // it's highly likely that the .locals will be needed to place some child in (for example,
    // a parameter, or variable declaration).
    //
    // However, we do not proactively create the .locals for block-containers because it's
    // totally normal and common for block-containers to never actually have a block-scoped
    // variable in them.  We don't want to end up allocating an object for every 'block' we
    // run into when most of them won't be necessary.
    //
    // Finally, if this is a block-container, then we clear out any existing .locals object
    // it may contain within it.  This happens in incremental scenarios.  Because we can be
    // reusing a node from a previous compilation, that node may have had 'locals' created
    // for it.  We must clear this so we don't accidentally move any stale data forward from
    // a previous compilation.
    let containerFlags = getContainerFlags(node);
    if (containerFlags & ContainerFlags.IsContainer) {
        container = blockScopeContainer = node;

        if (containerFlags & ContainerFlags.HasLocals) {
            container.locals = {};
        }

        addToContainerChain(container);
    }

    else if (containerFlags & ContainerFlags.IsBlockScopedContainer) {
        blockScopeContainer = node;
        blockScopeContainer.locals = undefined;
    }

    forEachChild(node, bind);

    container = saveContainer;
    parent = saveParent;
    blockScopeContainer = savedBlockScopeContainer;
}
```

正如你想起的绑定器函数那个章节：`bindChildren`从`bind`函数调用，因此我们有递归的绑定设置：`bind`调用`bindChildren`为每个子节点调用`bind`。