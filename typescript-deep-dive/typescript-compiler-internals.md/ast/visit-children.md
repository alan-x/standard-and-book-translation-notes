# 提示：遍历子孙

有一个工具函数`ts.forEachChild`允许你去访问 AST 中任何节点的子节点

这是一个源代码简化的片段，展示他的功能：
```ts
export function forEachChild<T>(node: Node, cbNode: (node: Node) => T, cbNodeArray?: (nodes: Node[]) => T): T {
        if (!node) {
            return;
        }
        switch (node.kind) {
            case SyntaxKind.BinaryExpression:
                return visitNode(cbNode, (<BinaryExpression>node).left) ||
                    visitNode(cbNode, (<BinaryExpression>node).operatorToken) ||
                    visitNode(cbNode, (<BinaryExpression>node).right);
            case SyntaxKind.IfStatement:
                return visitNode(cbNode, (<IfStatement>node).expression) ||
                    visitNode(cbNode, (<IfStatement>node).thenStatement) ||
                    visitNode(cbNode, (<IfStatement>node).elseStatement);

            // .... lots more
```

基本上，它检测`node.kind`，并基于`node`题哦那个一个接口的假设，并在子孙节点上调用`cbNode`。然而，注意这个函数不再所有的子孙节点上调用`visitNode`（）。如果你 想要 AST 节点的所有子孙就调用`node`侧`.getChildren`成员函数。

比如，这是一个函数，打印一个节点详细的`AST`：
```ts
function printAllChildren(node: ts.Node, depth = 0) {
    console.log(new Array(depth+1).join('----'), ts.syntaxKindToName(node.kind), node.pos, node.end);
    depth++;
    node.getChildren().forEach(c=> printAllChildren(c, depth));
}
```

当我们深入讨论转化器的时候，我们将看到这个函数一个简单的使用例子。