---
title: 常见问题
---

#### 为什么 ES 模块比 CommonJS 模块更好?

ES 模块是一个官方标准，也是 JavaScript 代码结构的明确的发展方向，而 CommonJS 模块是一种特殊且老旧的格式，仅仅在提出 ES 模块标准之前作为暂时性的解决方案。ES 模块允许进行静态分析，从而有助于摇树(tree-shaking)和作用域提升(scope-hoisting)之类的优化工作，同时提供了一些高级特性，例如循环引用和实时绑定。

#### 什么是“tree-shaking”？

Tree-shaking，也被称为“live code inclusion”，是 Rollup 消除项目中并未实际使用到的代码的过程。它是一种[消除无效代码的方式](https://medium.com/@Rich_Harris/tree-shaking-versus-dead-code-elimination-d3765df85c80#.jnypozs9n)，但在优化输出内容大小方面可能比其他方法有效得多。这个名字源于模块内容(而非模块图)的[抽象语法树(abstract syntax tree, AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree)。Tree-shaking 算法首先标记所有相关语句，然后“摇动语法树”以删除所有无效代码。其思想类似于[标记-清除垃圾收集算法](https://en.wikipedia.org/wiki/Tracing_garbage_collection)。虽然该算法不仅限于 ES 模块，但由于 Rollup 将所有模块视为一颗具有共享绑定的大型抽象语法树，使它能够运行的更快。

#### 我如何在使用 CommonJS 模块的 Node.js 中使用 Rollup？

Rollup 致力于实现 ES 模块的规范，并不一定会考虑 Node.js，NPM，`require()`，和 CommonJS 的行为。所以，加载 CommonJS 模块的逻辑和使用 Node.js 的模块位置解析的逻辑都在可选的插件中实现，Rollup 的核心代码并未默认提供这些功能。你只需要通过 `npm install` 安装 [commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs) 和 [node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve) 这两个插件并在 `rollup.config.js` 中启用它们，就可以正常使用以上功能。如果模块导入了 JSON 文件，那么你需要额外安装 [json](https://github.com/rollup/plugins/tree/master/packages/json) 这个插件。

#### 为什么不将 node-resolve 作为内置功能？

有两个主要原因：

1. （从 Rollup 的设计）哲学上来讲，Rollup 本质上是一个 [polyfill](https://en.wikipedia.org/wiki/Polyfill_(programming))，作用是对工作在 Node 和浏览器中的原生模块加载器进行排序。在浏览器中，`import foo from 'foo'` 不会正常工作，因为浏览器并没有使用 Node 的解析算法。

2. 从现实层面来讲，如果可以通过设计良好的 API 巧妙地分解这些问题，Rollup 的开发将变得更加轻松。Rollup 的核心代码非常庞大，所以一切阻止它变得臃肿的事情都是好事。与此同时，也能更容易地修复 bug 和加入新特性。Rollup 的代码库越精简，欠下的技术债务就越少。

参阅 [这个 issue](https://github.com/rollup/rollup/issues/1555#issuecomment-322862209) 以获取更详细的解释。

#### 为什么进行代码拆分时我的入口 chunk 出现了其它的导入？

默认情况下，当创建一个多重 chunk 时，入口 chunk 的依赖项导入将被作为空导入加入到这些 chunk 本身。[例子](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMHZhbHVlJTIwZnJvbSUyMCcuJTJGb3RoZXItZW50cnkuanMnJTNCJTVDbmNvbnNvbGUubG9nKHZhbHVlKSUzQiUyMiUyQyUyMmlzRW50cnklMjIlM0F0cnVlJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMm90aGVyLWVudHJ5LmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMGV4dGVybmFsVmFsdWUlMjBmcm9tJTIwJ2V4dGVybmFsJyUzQiU1Q25leHBvcnQlMjBkZWZhdWx0JTIwMiUyMColMjBleHRlcm5hbFZhbHVlJTNCJTIyJTJDJTIyaXNFbnRyeSUyMiUzQXRydWUlN0QlNUQlMkMlMjJvcHRpb25zJTIyJTNBJTdCJTIyZm9ybWF0JTIyJTNBJTIyZXNtJTIyJTJDJTIybmFtZSUyMiUzQSUyMm15QnVuZGxlJTIyJTJDJTIyYW1kJTIyJTNBJTdCJTIyaWQlMjIlM0ElMjIlMjIlN0QlMkMlMjJnbG9iYWxzJTIyJTNBJTdCJTdEJTdEJTJDJTIyZXhhbXBsZSUyMiUzQW51bGwlN0Q=)：

```js
// 输入
// main.js
import value from './other-entry.js';
console.log(value);

// other-entry.js
import externalValue from 'external';
export default 2 * externalValue;

// 输出
// main.js
import 'external'; // 这个导入从 other-entry.js 提升到这里
import value from './other-entry.js';
console.log(value);

// other-entry.js
import externalValue from 'external';
var value = 2 * externalValue;
export default value;
```

这并不影响代码的执行顺序或行为，但可以加速代码的加载和解析。没有了这个优化，Javascript 引擎需要执行这些步骤来运行 `main.js`：
1. 加载并解析 `main.js`。最后发现对 `other-entry.js` 的导入。
2. 加载并解析 `other-entry.js`。最后发现对 `external` 的导入。
3. 加载并解析 `external`。
4. 执行 `main.js`。

有了这个优化，Javascript 引擎就会在解析入口模块后发现所有的依赖传递，从而避免瀑布式解析：
1. 加载并解析 `main.js`。最后发现对 `other-entry.js` 和对 `external` 的导入。
2. 加载并解析 `other-entry.js` 和 `external`。此时对 `other-entry.js` 的导入已经完成了加载和解析。
3. 执行 `main.js`。

在一些情况下你可能不希望进行这样的优化，此时你可以通过 [`output.hoistTransitiveImports`](guide/en/#outputhoisttransitiveimports) 选项将其关闭。该优化在使用 [`output.preserveModules`](guide/en/#outputpreservemodules) 选项时也不会开启。

#### 如何将一些 polyfill 添加到 Rollup bundle 中？

尽管 Rollup 通常会在打包时尝试确保模块的执行顺序，但在这两种情况下并不总是如此：代码拆分时和存在外部依赖时。存在外部依赖时该问题1更为明显，看下面的 [例子](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMCcuJTJGcG9seWZpbGwuanMnJTNCJTVDbmltcG9ydCUyMCdleHRlcm5hbCclM0IlNUNuY29uc29sZS5sb2coJ21haW4nKSUzQiUyMiUyQyUyMmlzRW50cnklMjIlM0F0cnVlJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMnBvbHlmaWxsLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmNvbnNvbGUubG9nKCdwb2x5ZmlsbCcpJTNCJTIyJTJDJTIyaXNFbnRyeSUyMiUzQWZhbHNlJTdEJTVEJTJDJTIyb3B0aW9ucyUyMiUzQSU3QiUyMmZvcm1hdCUyMiUzQSUyMmVzbSUyMiUyQyUyMm5hbWUlMjIlM0ElMjJteUJ1bmRsZSUyMiUyQyUyMmFtZCUyMiUzQSU3QiUyMmlkJTIyJTNBJTIyJTIyJTdEJTJDJTIyZ2xvYmFscyUyMiUzQSU3QiU3RCU3RCUyQyUyMmV4YW1wbGUlMjIlM0FudWxsJTdE):

```js
// main.js
import './polyfill.js';
import 'external';
console.log('main');

// polyfill.js
console.log('polyfill');
```

这里的执行顺序是 `polyfill.js` → `external` → `main.js`。现在当你打包代码，你将得到

```js
import 'external';
console.log('polyfill');
console.log('main');
```

也就是按 `external` → `polyfill.js` → `main.js` 的顺序执行。这并非 Rollup 将 `import` 放在了打包代码的顶部引起的问题——导入的内容始终是优先执行的，而与它们在文件中的什么位置无关。这个问题可以通过创建更多的 chunk 来解决：如果 `dep.js` 与 `main.js` 最终不在同一个 chunk 中，就会[保留正确的执行顺序](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMCcuJTJGcG9seWZpbGwuanMnJTNCJTVDbmltcG9ydCUyMCdleHRlcm5hbCclM0IlNUNuY29uc29sZS5sb2coJ21haW4nKSUzQiUyMiUyQyUyMmlzRW50cnklMjIlM0F0cnVlJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMnBvbHlmaWxsLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmNvbnNvbGUubG9nKCdwb2x5ZmlsbCcpJTNCJTIyJTJDJTIyaXNFbnRyeSUyMiUzQXRydWUlN0QlNUQlMkMlMjJvcHRpb25zJTIyJTNBJTdCJTIyZm9ybWF0JTIyJTNBJTIyZXNtJTIyJTJDJTIybmFtZSUyMiUzQSUyMm15QnVuZGxlJTIyJTJDJTIyYW1kJTIyJTNBJTdCJTIyaWQlMjIlM0ElMjIlMjIlN0QlMkMlMjJnbG9iYWxzJTIyJTNBJTdCJTdEJTdEJTJDJTIyZXhhbXBsZSUyMiUzQW51bGwlN0Q=)。然而目前 Rollup 还没有办法自动地完成这件事情。对于代码拆分的过程，Rollup 会尝试创建尽可能少的 chunk，同时避免执行不需要的代码。

对于大多数代码来说这不会引发问题，因为 Rollup 可以确保：

> 如果模块 A 导入了模块 B 并且不存在循环引入，那么模块 B 将会在模块 A 之前执行。

但这对于 polyfill 来说就有问题了，因为通常需要先执行这些 polyfill 却不希望将所有 polyfill 的导入放在每个模块中。幸运的是，我们不需要这么做：

1. 如果没有依赖 polyfill 的外部依赖，那么只需要将导入 polyfill 的语句放在入口文件中就足够了。
2. 如果有的话，就将 polyfill 作为单独的入口或者 [manual chunk](guide/en/#outputmanualchunks)，这将能够确保 polyfill 总是优先执行。

#### Rollup 可以用于构建库或者应用程序吗？

Rollup 已经被用在了很多流行的 Javascript 库中，也可以用于构建绝大多数应用程序。然而如果你希望在旧版本浏览器中使用代码拆分或动态导入，你将会需要一个额外的运行时来加载缺少的 chunk。我们建议使用 [生产版本的 SystemJS](https://github.com/systemjs/systemjs#browser-production)，因为它能够很好地与 Rollup 的系统格式化输出协同工作，并且能够正确处理所有 ES 模块的实时绑定与重复导出的边缘情况。另外，AMD loader 也是可以使用的。

#### Rollup 的 logo 是谁做的？这太可爱了。

是 [Julian Lloyd](https://twitter.com/jlmakes)!
