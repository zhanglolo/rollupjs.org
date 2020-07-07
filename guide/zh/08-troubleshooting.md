---
title: 疑难解答
---

如果你遇到疑难问题，可以尝试在 [the Rollup Gitter](https://gitter.im/rollup/rollup) 创建一个 issue 讨论或者在 https://stackoverflow.com/questions/tagged/rollupjs 中发布你的问题。如果你找到了一个 bug 或者你认为 rollup 不能满足你的需求，请尝试[创建 1 个 issue](https://github.com/rollup/rollup/issues)。最后，你还可以在 Twitter 和 [@RollupJS](https://twitter.com/RollupJS) 取得联系。

### 避免使用 `eval`

你很有可能已经知道 “`eval` 是魔鬼”，至少听别人这么说过。 但对 Rollup 来说它确实危害极大，原因是 rollup 的工作机制——不像其它模块打包器那样会把每个模块包裹在独立的一个函数里面，相反，Rollup 把你所有的代码放在相同的作用域中。

这种处理方式效率更高，但也意味着一旦你使用了 `eval`，这个共享的作用域就被“污染”了， 然而，对于其他的打包器来说，如果模块 *没有* 使用过 eval 的话，就不会被“污染”。代码压缩工具无法在被“污染”的代码中去混淆变量名，因为它无法保证使用 eval 执行的代码不会再引用这些变量名。


此外， **它带来了一个安全风险**，也就是一个恶意的模块可以通过 `eval('SUPER_SEKRIT')` 访问另一个模块的私有变量。

幸运的是，如果你 *确实* 想要被 eval 执行的代码可以访问到局部变量的话（这种情况你很有可能在做一些错误的事情！），你可以在这两种方式中选择任意一种达到相同的效果：


#### eval2 = eval

通过简单地“复制” `eval` 提供了一个几乎完全相同的函数，区别是它在全局作用域中执行而不是在局部作用域中：
Simply 'copying' `eval` provides you with a function that does exactly the same thing，but which runs in the global scope rather than the local one:

```js
var eval2 = eval;

(function () {
  var foo = 42;
  eval('console.log("with eval:",foo)');  // logs 'with eval: 42'
  eval2('console.log("with eval2:",foo)'); // throws ReferenceError
})();
```

#### `new Function`

使用[函数构造器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) 可以把提供的字符串参数创建为一个函数。同样，它也是在全局作用域运行的。如果你想要重复地调用这个函数，这种方式会比使用 `eval` 快 *很* 多。


### Tree-shaking Doesn't Seem to be Working

有时候，你最后会在你的包中发现一些看起来不应该出现在这个地方的代码。例如，如果你从 `lodash-es` 中 import 一个 utility，你也许期望的是你将会获取最少数量的但足以让这个 utility 正常运行的代码。

但是 rollup 必须保守地对待它要移除的代码以确保最终的代码能够正确无误地运行。如果一个导入的模块是有 *副作用* 的，不管是在你使用的这个模块内还是在全局环境中，Rollup 以安全的方式处理它同时也会把副作用包含进来。


因为静态分析在一门动态语言如 JavaScript 中是很困难的，偶尔会有误报。lodash 就是一个很好的示例，由于它 *看起来* 好像是一个有很多副作用的模块，尽管有些地方它并非如此。你通常可以通过引入子模块来消除这样的误报（ e.g. 使用 `import map from 'lodash-es/map'` 替代 `import { map } from 'lodash-es'` ）。

rollup的静态分析会逐渐优化提升，但是它不会在任何情况下都很完美——仅仅在 JavaScript 中如此。


### Error: "[name] is not exported by [module]"

你偶尔会看到像这样的一条错误信息：

> 'foo' is not exported by bar.js (imported by baz.js)

引入声明必须在引入的模块中导出对应的声明。例如，如果你在一个模块中有 `import a from './a.js'`， 与此同时 a.js 文件没有 `export default` 声明，或者当你 `import {foo} from './b.js'` 与此同时 b.js 文件中没有导出 `foo`，Rollup 就不会打包这些代码。


这个错误经常出现在被 [@rollup/plugin-commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs) 转换的 CommonJS 模块中，这会导致原本合理的给 CommonJS 创建命名的导出操作会失败，这是因为 CommonJS 无拘无束的特性和 JavaScript 模块中使我们受益的严格模式相违背。这个问题可以通过使用 [namedExports](https://github.com/rollup/plugins/tree/master/packages/commonjs#custom-named-exports) 选项解决，它允许你手动地填补信息差。




### Error: "this is undefined"

在一个 JavaScript 模块中，在顶级作用域（即：函数外部） `this` 是 `undefined` 的。因此，Rollup 会重写所有的 `this` 引用为 `undefined`，使其最终的行为跟原生支持模块时的行为一致。

在一些偶然情况下，让 `this` 表示成别的含义是有效的。如果你的包出现了错误，你可以使用  `options.context` 和 `options.moduleContext` 去改变这个行为。



### Warning: "Sourcemap is likely to be incorrect"

如果你给包文件创建了 sourcemap（`sourceMap: true` or `sourceMap: 'inline'`），你会看到这个警告，此时你使用了一个或者多个插件来转换代码，但是没有给转换也生成一个 sourcemap。

通常情况下，如果一个插件配置了 `sourceMap: false` ——你所要做仅仅是修改它，只有在这种情况下会省略 sourcemap。如果在正常情况下这个插件没有创建 sourcemap，你可以尝试创建一个 issue 给这个插件的作者。


### Warning: "Treating [module] as external dependency"

Rollup 只会默认处理  *相关联的* 模块id，这意味着如果你像这样引入一个模块......

```js
import moment from 'moment';
```

结果是 `moment` 并不会被包含在你的包中——相反，它会是一个在运行时才需要被加载的外部模块。如果这不是你想要的，你可以通过 `external` 选项来消除这些警告，来使你的意图更加清晰：


```js
// rollup.config.js
export default {
  entry: 'src/index.js',
  dest: 'bundle.js',
  format: 'cjs',
  external: [ 'moment' ] // <-- suppresses the warning
};
```

如果你 *确实* 想要把这个模块包含在你的包中，你需要告诉 Rollup 怎么去找到它。在多数情况下，这是一个关于如何使用 [@rollup/plugin-node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve) 的问题。

有些模块，比如 `events` 或者 `util`这些 Node.js 的内建模块。如果你想要包含这些模块（例如，这么做可以使你的包就在浏览器环境也可以运行），或许需要使用 [rollup-plugin-node-polyfills](https://github.com/ionic-team/rollup-plugin-node-polyfills)。
