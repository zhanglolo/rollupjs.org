---
title: Rollup 集成第三方工具
---

### 使用 NPM 包

有时候，您的项目可能需要依赖于一些 NPM 包。与 Webpack 和 Browserify 等其他打包器不同，Rollup 并不能“开箱即用”地处理 NPM 包的依赖关系——我们需要添加一些配置。

让我们添加一个简单的依赖， [the-answer](https://www.npmjs.com/package/the-answer)。该依赖可以输出关于生命、宇宙和一切的问题的答案：

```
npm install the-answer
# 或者 `npm i the-answer`
```

假设我们更改了 `src/main.js` 文件的内容：

```js
// src/main.js
import answer from 'the-answer';

export default function () {
  console.log('the answer is ' + answer);
}
```

然后运行 Rollup

```
npm run build
```

我们将会看到这样的警告：

```
(!) Unresolved dependencies
https://github.com/rollup/rollup/wiki/Troubleshooting#treating-module-as-external-dependency
the-answer (imported by main.js)
```

由此输出的 `bundle.js` 仍然可以在 Node.js 中运行，因为 `import` 声明被转换成为 CommonJS 规范的 `require` 语句，但是 `the-answer` **并未** 被打包在 bundle 中。为此，我们需要一个插件。


#### @rollup/plugin-node-resolve

[@rollup/plugin-node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve) 插件可以让 Rollup 查找到外部模块。首先来安装它……

```
npm install --save-dev @rollup/plugin-node-resolve
```

并将其添加到你的配置文件中：

```js
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [ resolve() ]
};
```

这一次，当你运行 `npm run build` 时，不会再有警告——因为 bundle 中已经包含引入的模块。


#### @rollup/plugin-commonjs

一些库暴露了可以按照原样导入的 ES 模块——`the-answer` 就是这样的一种模块。但目前，大多数的 NPM 包暴露的都是 CommonJS 模块。在此更改之前，我们需要将 CommonJS 转换为 ES2015，这样 Rollup 才能处理它们。

[@rollup/plugin-commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs) 可以做到这一点。

请注意，`@rollup/plugin-commonjs` 应该在其他插件 *之前* 使用——这是为了防止其他插件进行的更改破坏了 CommonJS 检测。


### 前置依赖（Peer dependencies）

假设你正在构建一个类似 React 和 Lodash 这样具有前置依赖的库，如果你按照上述的方式设置外部引入（external），rollup 将会打包 *所有* 的引入项:

```js
import answer from 'the-answer';
import _ from 'lodash';
```

你可以微调哪些引入需要被打包，哪些引入需要设置为外部引入。在这个例子中，我们可以将 `lodash` 视为外部引入，而不是 `the-answer`。

下面是配置文件:

```js
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [resolve({
    // 将自定义选项传递给解析插件
    customResolveOptions: {
      moduleDirectory: 'node_modules'
    }
  })],
  // 指出哪些模块需要被视为外部引入
  external: ['lodash']
};
```

瞧，`lodash` 现在会被视为外部引入，而不会和你的库打包在一起。

属性 `external` 接收一个模块名称组成的数组，或者接收一个参数为模块名字的函数，如果需要把某模块设置为外部引入，只需要让该函数返回 true。例如：

```js
export default {
  // ...
  external: id => /lodash/.test(id)
}
```

如果你正在使用 [babel-plugin-lodash](https://github.com/lodash/babel-plugin-lodash) 来按需引入 `lodash` 的子模块，则可以使用这种形式。在这种情况下，Babel 会将您的引入语句转换为如下形式：

```js
import _merge from 'lodash/merge';
```

`external` 的数组形式不处理通配符，因此在函数形式中，此引入仅被视为外部引入。


### Babel

为了能够使用浏览器和 Node.js 尚不支持的最新 JavaScript 特性，许多开发人员会在项目中使用 [Babel](https://babeljs.io/)。

Babel 和 Rollup 一起使用的最简单的方式是通过插件 [@rollup/plugin-babel](https://github.com/rollup/plugins/tree/master/packages/babel)。首先，安装该插件：

```
npm i -D @rollup/plugin-babel @rollup/plugin-node-resolve
```

添加以下代码到 `rollup.config.js`:

```js
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import babel from '@rollup/plugin-babel';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    resolve(),
    babel({ babelHelpers: 'bundled' })
  ]
};
```

在 Babel 实际编译你的代码之前，需要对其进行配置。 创建一个新文件，`src/.babelrc.json`：

```json
{
  "presets": [
    ["@babel/env", {"modules": false}]
  ]
}
```

该设置有一些不寻常的元素。首先，我们将 `modules` 设置为 `false`，否则 Babel 将在 Rollup 有机会执行其操作之前将我们的模块转换为 CommonJS ，从而导致其失败。

其次，我们将 `.babelrc.json` 文件放置在 src 下，而不是项目根目录中。 如果以后需要的话，可以让我们在做测试之类的事时使用不同的 `.babelrc.json`——请参阅 [Babel documentation](https://babeljs.io/docs/en/config-files#project-wide-configuration)，以获取有关项目范围和文件相关配置的更多信息。

现在，在运行 rollup 之前，我们需要安装 [`babel-core`](https://babeljs.io/docs/en/babel-core) 和 [`env`](https://babeljs.io/docs/en/babel-preset-env)：

```
npm i -D @babel/core @babel/preset-env
```

现在运行 Rollup 将创建一个 bundle ——除非我们并未添加任何 ES2015 的功能。 让我们通过编辑 `src/main.js` 来改变它：

```js
// src/main.js
import answer from 'the-answer';

export default () => {
  console.log(`the answer is ${answer}`);
}
```

运行 Rollup：`npm run build`，并检查输出的 bundle：

```js
'use strict';

var index = 42;

var main = (function () {
  console.log('the answer is ' + index);
});

module.exports = main;
```

### Gulp

Gulp 可以理解 Rollup 返回的 Promise，所以二者集成起来相对容易。

语法和配置文件非常相似，但是有两种不同的操作方式，分别对应两种 [JavaScript API](guide/zh/#javascript-api)：

```js
const gulp = require('gulp');
const rollup = require('rollup');
const rollupTypescript = require('@rollup/plugin-typescript');

gulp.task('build', () => {
  return rollup.rollup({
    input: './src/main.ts',
    plugins: [
      rollupTypescript()
    ]
  }).then(bundle => {
    return bundle.write({
      file: './dist/library.js',
      format: 'umd',
      name: 'library',
      sourcemap: true
    });
  });
});
```

或者使用 `async/await` 语法:

```js
const gulp = require('gulp');
const rollup = require('rollup');
const rollupTypescript = require('@rollup/plugin-typescript');

gulp.task('build', async function () {
  const bundle = await rollup.rollup({
    input: './src/main.ts',
    plugins: [
      rollupTypescript()
    ]
  });

  await bundle.write({
    file: './dist/library.js',
    format: 'umd',
    name: 'library',
    sourcemap: true
  });
});
```
