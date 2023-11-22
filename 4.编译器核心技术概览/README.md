1. 在计算机科学中，抽象语法树（Abstract Syntax Tree，AST），或简称语法树（Syntax tree），是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。比如，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现；而类似于 if-condition-then 这样的条件跳转语句，可以使用带有三个分支的节点来表示。和抽象语法树相对的是具体语法树（通常称作分析树）。一般的，在源代码的翻译和编译过程中，语法分析器创建出分析树，然后从分析树生成AST。一旦AST被创建出来，在后续的处理过程中，比如语义分析阶段，会添加一些信息。

### 模板编译分为那几个步骤？

_解析模板生成模板 AST_

```js
let str = "<p>Vue</p>";
```

如以上模板利用 _有限状态机_ 先解析语法生成 `tokens` 集合。

```js
let tokens = [
  { type: "tag", name: "p" },
  { type: "text", content: "Vue" },
  { type: "tagEnd", name: "p" },
];
```

根据 `tokens` 维护一个栈用于匹配标签的开头和结尾生成模板的 `AST`。

```json
{
  "type": "Root",
  "children": [
    {
      "type": "Element",
      "tag": "p",
      "children": [{ "type": "Text", "content": "Vue" }]
    }
  ]
}
```

_转换模板 AST 为 JSAST_

这一步是为了转化为目标语言的 `AST` 节点便于后续的优化和生成。

```json
{
  "type": "Root",
  "children": [
    {
      "type": "FunctionDecl",
      "id": { "type": "Identifier", "name": "render" },
      "params": [],
      "body": [
        {
          "type": "ReturnStatement",
          "return": {
            "type": "CallExpression",
            "callee": { "type": "Identifier", "name": "h" },
            "arguments": [
              { "type": "StringLiteral", "value": "p" },
              { "type": "StringLiteral", "value": "Vue" }
            ]
          }
        }
      ]
    }
  ]
}
```

_根据 JSAST 生成渲染函数_

最后根据 `JSAST` 拼接字符串生成目标语言。

```js
function render() {
  return h("p", "Vue");
}
```
