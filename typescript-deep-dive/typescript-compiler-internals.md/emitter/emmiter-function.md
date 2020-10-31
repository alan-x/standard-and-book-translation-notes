# 发射器函数

### `emitFiles`

定义在`emitter.ts`，这是函数签名：
```ts
// targetSourceFile is when users only want one file in entire project to be emitted. This is used in compileOnSave feature
export function emitFiles(resolver: EmitResolver, host: EmitHost, targetSourceFile?: SourceFile): EmitResult {
```

`EmitHost`只是一个简化（比如缩小范围）版本的`EmitHost`（并且在运行时，对于很多场景，实际是一个`CompilerHost`）。

`emitFiles`最有趣的调用栈如下：
```ts
emitFiles ->
    emitFile(jsFilePath, targetSourceFile) ->
        emitJavaScript(jsFilePath, targetSourceFile);
```

### `emitJavaScript`

有很多的好的注释在这个函数，因此我们可以在下面展示：
```ts
function emitJavaScript(jsFilePath: string, root?: SourceFile) {
    let writer = createTextWriter(newLine);
    let write = writer.write;
    let writeTextOfNode = writer.writeTextOfNode;
    let writeLine = writer.writeLine;
    let increaseIndent = writer.increaseIndent;
    let decreaseIndent = writer.decreaseIndent;

    let currentSourceFile: SourceFile;
    // name of an exporter function if file is a System external module
    // System.register([...], function (<exporter>) {...})
    // exporting in System modules looks like:
    // export var x; ... x = 1
    // =>
    // var x;... exporter("x", x = 1)
    let exportFunctionForFile: string;

    let generatedNameSet: Map<string> = {};
    let nodeToGeneratedName: string[] = [];
    let computedPropertyNamesToGeneratedNames: string[];

    let extendsEmitted = false;
    let decorateEmitted = false;
    let paramEmitted = false;
    let awaiterEmitted = false;
    let tempFlags = 0;
    let tempVariables: Identifier[];
    let tempParameters: Identifier[];
    let externalImports: (ImportDeclaration | ImportEqualsDeclaration | ExportDeclaration)[];
    let exportSpecifiers: Map<ExportSpecifier[]>;
    let exportEquals: ExportAssignment;
    let hasExportStars: boolean;

    /** Write emitted output to disk */
    let writeEmittedFiles = writeJavaScriptFile;

    let detachedCommentsInfo: { nodePos: number; detachedCommentEndPos: number }[];

    let writeComment = writeCommentRange;

    /** Emit a node */
    let emit = emitNodeWithoutSourceMap;

    /** Called just before starting emit of a node */
    let emitStart = function (node: Node) { };

    /** Called once the emit of the node is done */
    let emitEnd = function (node: Node) { };

    /** Emit the text for the given token that comes after startPos
      * This by default writes the text provided with the given tokenKind
      * but if optional emitFn callback is provided the text is emitted using the callback instead of default text
      * @param tokenKind the kind of the token to search and emit
      * @param startPos the position in the source to start searching for the token
      * @param emitFn if given will be invoked to emit the text instead of actual token emit */
    let emitToken = emitTokenText;

    /** Called to before starting the lexical scopes as in function/class in the emitted code because of node
      * @param scopeDeclaration node that starts the lexical scope
      * @param scopeName Optional name of this scope instead of deducing one from the declaration node */
    let scopeEmitStart = function(scopeDeclaration: Node, scopeName?: string) { };

    /** Called after coming out of the scope */
    let scopeEmitEnd = function() { };

    /** Sourcemap data that will get encoded */
    let sourceMapData: SourceMapData;

    if (compilerOptions.sourceMap || compilerOptions.inlineSourceMap) {
        initializeEmitterWithSourceMaps();
    }

    if (root) {
        // Do not call emit directly. It does not set the currentSourceFile.
        emitSourceFile(root);
    }
    else {
        forEach(host.getSourceFiles(), sourceFile => {
            if (!isExternalModuleOrDeclarationFile(sourceFile)) {
                emitSourceFile(sourceFile);
            }
        });
    }

    writeLine();
    writeEmittedFiles(writer.getText(), /*writeByteOrderMark*/ compilerOptions.emitBOM);
    return;

    /// BUNCH OF LOCAL FUNCTIONS
}
```

基本上它设置了一系列的本地（），然后交给本地函数`emitSourceFile`，它启动发射。`emitSourceFile`函数只是设置`currentSourceFile`，然后交给本地的`emit`函数。

```ts
function emitSourceFile(sourceFile: SourceFile): void {
    currentSourceFile = sourceFile;
    exportFunctionForFile = undefined;
    emit(sourceFile);
}
```

`emit`函数处理注释生成+真实 JavaScript 生成。真实 JavaScript 生成是`emitJavaScriptWorker`函数的功能。

### `emitJavaScriptWorker`

完整的函数：
```ts
function emitJavaScriptWorker(node: Node) {
    // Check if the node can be emitted regardless of the ScriptTarget
    switch (node.kind) {
        case SyntaxKind.Identifier:
            return emitIdentifier(<Identifier>node);
        case SyntaxKind.Parameter:
            return emitParameter(<ParameterDeclaration>node);
        case SyntaxKind.MethodDeclaration:
        case SyntaxKind.MethodSignature:
            return emitMethod(<MethodDeclaration>node);
        case SyntaxKind.GetAccessor:
        case SyntaxKind.SetAccessor:
            return emitAccessor(<AccessorDeclaration>node);
        case SyntaxKind.ThisKeyword:
            return emitThis(node);
        case SyntaxKind.SuperKeyword:
            return emitSuper(node);
        case SyntaxKind.NullKeyword:
            return write("null");
        case SyntaxKind.TrueKeyword:
            return write("true");
        case SyntaxKind.FalseKeyword:
            return write("false");
        case SyntaxKind.NumericLiteral:
        case SyntaxKind.StringLiteral:
        case SyntaxKind.RegularExpressionLiteral:
        case SyntaxKind.NoSubstitutionTemplateLiteral:
        case SyntaxKind.TemplateHead:
        case SyntaxKind.TemplateMiddle:
        case SyntaxKind.TemplateTail:
            return emitLiteral(<LiteralExpression>node);
        case SyntaxKind.TemplateExpression:
            return emitTemplateExpression(<TemplateExpression>node);
        case SyntaxKind.TemplateSpan:
            return emitTemplateSpan(<TemplateSpan>node);
        case SyntaxKind.JsxElement:
        case SyntaxKind.JsxSelfClosingElement:
            return emitJsxElement(<JsxElement|JsxSelfClosingElement>node);
        case SyntaxKind.JsxText:
            return emitJsxText(<JsxText>node);
        case SyntaxKind.JsxExpression:
            return emitJsxExpression(<JsxExpression>node);
        case SyntaxKind.QualifiedName:
            return emitQualifiedName(<QualifiedName>node);
        case SyntaxKind.ObjectBindingPattern:
            return emitObjectBindingPattern(<BindingPattern>node);
        case SyntaxKind.ArrayBindingPattern:
            return emitArrayBindingPattern(<BindingPattern>node);
        case SyntaxKind.BindingElement:
            return emitBindingElement(<BindingElement>node);
        case SyntaxKind.ArrayLiteralExpression:
            return emitArrayLiteral(<ArrayLiteralExpression>node);
        case SyntaxKind.ObjectLiteralExpression:
            return emitObjectLiteral(<ObjectLiteralExpression>node);
        case SyntaxKind.PropertyAssignment:
            return emitPropertyAssignment(<PropertyDeclaration>node);
        case SyntaxKind.ShorthandPropertyAssignment:
            return emitShorthandPropertyAssignment(<ShorthandPropertyAssignment>node);
        case SyntaxKind.ComputedPropertyName:
            return emitComputedPropertyName(<ComputedPropertyName>node);
        case SyntaxKind.PropertyAccessExpression:
            return emitPropertyAccess(<PropertyAccessExpression>node);
        case SyntaxKind.ElementAccessExpression:
            return emitIndexedAccess(<ElementAccessExpression>node);
        case SyntaxKind.CallExpression:
            return emitCallExpression(<CallExpression>node);
        case SyntaxKind.NewExpression:
            return emitNewExpression(<NewExpression>node);
        case SyntaxKind.TaggedTemplateExpression:
            return emitTaggedTemplateExpression(<TaggedTemplateExpression>node);
        case SyntaxKind.TypeAssertionExpression:
            return emit((<TypeAssertion>node).expression);
        case SyntaxKind.AsExpression:
            return emit((<AsExpression>node).expression);
        case SyntaxKind.ParenthesizedExpression:
            return emitParenExpression(<ParenthesizedExpression>node);
        case SyntaxKind.FunctionDeclaration:
        case SyntaxKind.FunctionExpression:
        case SyntaxKind.ArrowFunction:
            return emitFunctionDeclaration(<FunctionLikeDeclaration>node);
        case SyntaxKind.DeleteExpression:
            return emitDeleteExpression(<DeleteExpression>node);
        case SyntaxKind.TypeOfExpression:
            return emitTypeOfExpression(<TypeOfExpression>node);
        case SyntaxKind.VoidExpression:
            return emitVoidExpression(<VoidExpression>node);
        case SyntaxKind.AwaitExpression:
            return emitAwaitExpression(<AwaitExpression>node);
        case SyntaxKind.PrefixUnaryExpression:
            return emitPrefixUnaryExpression(<PrefixUnaryExpression>node);
        case SyntaxKind.PostfixUnaryExpression:
            return emitPostfixUnaryExpression(<PostfixUnaryExpression>node);
        case SyntaxKind.BinaryExpression:
            return emitBinaryExpression(<BinaryExpression>node);
        case SyntaxKind.ConditionalExpression:
            return emitConditionalExpression(<ConditionalExpression>node);
        case SyntaxKind.SpreadElementExpression:
            return emitSpreadElementExpression(<SpreadElementExpression>node);
        case SyntaxKind.YieldExpression:
            return emitYieldExpression(<YieldExpression>node);
        case SyntaxKind.OmittedExpression:
            return;
        case SyntaxKind.Block:
        case SyntaxKind.ModuleBlock:
            return emitBlock(<Block>node);
        case SyntaxKind.VariableStatement:
            return emitVariableStatement(<VariableStatement>node);
        case SyntaxKind.EmptyStatement:
            return write(";");
        case SyntaxKind.ExpressionStatement:
            return emitExpressionStatement(<ExpressionStatement>node);
        case SyntaxKind.IfStatement:
            return emitIfStatement(<IfStatement>node);
        case SyntaxKind.DoStatement:
            return emitDoStatement(<DoStatement>node);
        case SyntaxKind.WhileStatement:
            return emitWhileStatement(<WhileStatement>node);
        case SyntaxKind.ForStatement:
            return emitForStatement(<ForStatement>node);
        case SyntaxKind.ForOfStatement:
        case SyntaxKind.ForInStatement:
            return emitForInOrForOfStatement(<ForInStatement>node);
        case SyntaxKind.ContinueStatement:
        case SyntaxKind.BreakStatement:
            return emitBreakOrContinueStatement(<BreakOrContinueStatement>node);
        case SyntaxKind.ReturnStatement:
            return emitReturnStatement(<ReturnStatement>node);
        case SyntaxKind.WithStatement:
            return emitWithStatement(<WithStatement>node);
        case SyntaxKind.SwitchStatement:
            return emitSwitchStatement(<SwitchStatement>node);
        case SyntaxKind.CaseClause:
        case SyntaxKind.DefaultClause:
            return emitCaseOrDefaultClause(<CaseOrDefaultClause>node);
        case SyntaxKind.LabeledStatement:
            return emitLabelledStatement(<LabeledStatement>node);
        case SyntaxKind.ThrowStatement:
            return emitThrowStatement(<ThrowStatement>node);
        case SyntaxKind.TryStatement:
            return emitTryStatement(<TryStatement>node);
        case SyntaxKind.CatchClause:
            return emitCatchClause(<CatchClause>node);
        case SyntaxKind.DebuggerStatement:
            return emitDebuggerStatement(node);
        case SyntaxKind.VariableDeclaration:
            return emitVariableDeclaration(<VariableDeclaration>node);
        case SyntaxKind.ClassExpression:
            return emitClassExpression(<ClassExpression>node);
        case SyntaxKind.ClassDeclaration:
            return emitClassDeclaration(<ClassDeclaration>node);
        case SyntaxKind.InterfaceDeclaration:
            return emitInterfaceDeclaration(<InterfaceDeclaration>node);
        case SyntaxKind.EnumDeclaration:
            return emitEnumDeclaration(<EnumDeclaration>node);
        case SyntaxKind.EnumMember:
            return emitEnumMember(<EnumMember>node);
        case SyntaxKind.ModuleDeclaration:
            return emitModuleDeclaration(<ModuleDeclaration>node);
        case SyntaxKind.ImportDeclaration:
            return emitImportDeclaration(<ImportDeclaration>node);
        case SyntaxKind.ImportEqualsDeclaration:
            return emitImportEqualsDeclaration(<ImportEqualsDeclaration>node);
        case SyntaxKind.ExportDeclaration:
            return emitExportDeclaration(<ExportDeclaration>node);
        case SyntaxKind.ExportAssignment:
            return emitExportAssignment(<ExportAssignment>node);
        case SyntaxKind.SourceFile:
            return emitSourceFileNode(<SourceFile>node);
    }
}
```

递归通过简单的从这些函数调用`emitFoo`作为`emitFunctionDeclaration`需要的：
```ts
function emitFunctionDeclaration(node: FunctionLikeDeclaration) {
    if (nodeIsMissing(node.body)) {
        return emitOnlyPinnedOrTripleSlashComments(node);
    }

    if (node.kind !== SyntaxKind.MethodDeclaration && node.kind !== SyntaxKind.MethodSignature) {
        // Methods will emit the comments as part of emitting method declaration
        emitLeadingComments(node);
    }

    // For targeting below es6, emit functions-like declaration including arrow function using function keyword.
    // When targeting ES6, emit arrow function natively in ES6 by omitting function keyword and using fat arrow instead
    if (!shouldEmitAsArrowFunction(node)) {
        if (isES6ExportedDeclaration(node)) {
            write("export ");
            if (node.flags & NodeFlags.Default) {
                write("default ");
            }
        }

        write("function");
        if (languageVersion >= ScriptTarget.ES6 && node.asteriskToken) {
            write("*");
        }
        write(" ");
    }

    if (shouldEmitFunctionName(node)) {
        emitDeclarationName(node);
    }

    emitSignatureAndBody(node);
    if (languageVersion < ScriptTarget.ES6 && node.kind === SyntaxKind.FunctionDeclaration && node.parent === currentSourceFile && node.name) {
        emitExportMemberAssignments((<FunctionDeclaration>node).name);
    }
    if (node.kind !== SyntaxKind.MethodDeclaration && node.kind !== SyntaxKind.MethodSignature) {
        emitTrailingComments(node);
    }
}
```

