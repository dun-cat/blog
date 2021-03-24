## Babel Ast Custom Plugin 
### 简介

Babel 是一款 javascript 的编译器，其主要工作是把 ECMAScript 2015+ 标准以上的代码向下兼容到当前的浏览器或环境。这直接带来的好处是可以采用更高版本的标准语法去编写代码，而无需考虑过多的环境兼容因素。

Babel 提供了[插件系统](https://babeljs.io/docs/en/plugins)，任何人都可以基于 babel 编写插件来实现自定义语法转换，这对于开发者来说是个福音。

而这一切的基础需要了解的一个概念：`语法树（Abstract Syntax Tree）`，简称：`AST`。

AST 表示的你的代码，对于 AST 的编辑等同于对代码的编辑，传统的编译器也有做同样工作的结构被叫做`具体语法解析树（CST）`，而 AST 是 CST 的简化版本。

### 如何使用 Label 转换代码

下面是个简单的转换例子：

``` javascript
import { parse } from '@babel/parser';
import traverse from '@babel/traverse';
import generate from '@babel/generator';

const code = 'const n = 1';

// parse the code -> ast
const ast = parse(code);

// transform the ast
traverse(ast, {
  enter(path) {
    // in this example change all the variable `n` to `x`
    if (path.isIdentifier({ name: 'n' })) {
      path.node.name = 'x';
    }
  },
});

// generate code <- ast
const output = generate(ast, code);
console.log(output.code); // 'const x = 1;'
```

`解析（parse）`-> `转换（transform）`-> `生成(generate)`，三个明确的步骤完成代码转换操作。


> 你可以直接安装[@babel/core](https://www.npmjs.com/package/@babel/core)完成以上操作，*@babel/parser*、*@babel/traverse*、*@babel/generator* 都是 @babel/core 的依赖，所以直接安装 @babel/core 即可。

### 通过插件来实现转换

除了上面的方式，更为通用的做法是通过插件来实现：

``` javascript
import babel from '@babel/core';

const code = 'const n = 1';

const output = babel.transformSync(code, {
  plugins: [
    // your first babel plugin 😎😎
    function myCustomPlugin() {
      return {
        visitor: {
          Identifier(path) {
            // in this example change all the variable `n` to `x`
            if (path.isIdentifier({ name: 'n' })) {
              path.node.name = 'x';
            }
          },
        },
      };
    },
  ],
});

console.log(output.code); // 'const x = 1;'
```


扩展阅读：

\> [https://lihautan.com/step-by-step-guide-for-writing-a-babel-transformation/](https://lihautan.com/step-by-step-guide-for-writing-a-babel-transformation/)