---
title: Tutorial
---

### 创建第一个 Bundle

*在我们开始之前,你需要已经安装了 [Node.js](https://nodejs.org)， 才能使用 [NPM](https://npmjs.com)；同时，你也需要了解如何在你的机器上使用 [命令行](https://www.codecademy.com/learn/learn-the-command-line)。*

使用 Rollup 最简单的方式是通过 Command Line Interface （或 CLI）。现在，我们全局安装 Rollup （之后我们会学习如何本地化安装它到你的项目中，更便于打包，但是现在先不用考虑这个问题）。在命令行中输入以下内容：

```
npm install rollup --global
# or `npm i rollup -g` for short
```

现在可以运行 `rollup` 命令了. 试试看!

```
rollup
```

由于没有传参, Rollup 打印出了使用说明。这和运行命令 `rollup --help`，或者 `rollup -h` 效果一样。

让我们一起构建一个简单的项目:

```
mkdir -p my-rollup-project/src
cd my-rollup-project
```

首先，我们需要一个 *入口*。将以下内容粘贴到新建文件 `src/main.js` 中:

```js
// src/main.js
import foo from './foo.js';
export default function () {
  console.log(foo);
}
```

然后，我们新建 `foo.js` 模块用于导入到我们的入口文件中:

```js
// src/foo.js
export default 'hello world!';
```

现在，我们准备构建一个 bundle:

```
rollup src/main.js -f cjs
```

`-f` 可选项 (是 `--format`的缩写) ，它明确了我们正在构建的是哪类 bundle — 在本示例中，是使用的 CommonJS (将在 Node.js中运行)。由于我们没有指定一个输出文件，它将直接打印到 `stdout` 中:

```js
'use strict';

const foo = 'hello world!';

const main = function () {
  console.log(foo);
};

module.exports = main;
```

你可以像如下示例一样作为一个文件保存这个 bundle:

```
rollup src/main.js -o bundle.js -f cjs
```

(也可以使用 `rollup src/main.js -f cjs > bundle.js`, 但是稍后将会明白，如果你生成 sourcemaps, 这种方式的灵活度较低。)

试着运行代码：

```
node
> var myBundle = require('./bundle.js');
> myBundle();
'hello world!'
```

恭喜！你已经使用 Rollup 构建完成你的第一个 bundle 了。

### 使用配置文件

到目前为止，如此的顺利，但是当我们开始添加更多可选项时，键入命令变的有些麻烦。

为了解救自我，我们可以创建一个配置文件，它包含了所有我们想要的可选项。这份配置文件是用 JavaScript 写的，比单纯的 CLI 更灵活。

在项目根目录创建一个名为 `rollup.config.js` 的文件，并添加如下代码：

```js
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

(注：你可以使用 CJS 模式如 `module.exports = {/* config */}`)

为了使用这个配置文件, 我们可以用 `--config` 或者 `-c` 选项:

```
rm bundle.js # so we can check the command works!
rollup -c
```

你可以使用命令行中同等的可选项重写配置文件中可选项:

```
rollup -c -o bundle-2.js # `-o` is equivalent to `--file` (formerly "output")
```

_注: Rollup 本身执行了配置文件, 因此我们能够使用 `export default` 语法 – 这些代码没有被 Babel 或者其他类似工具编译，因此你只能使用 Node.js 版本支持的 ES2015 语法特性。_

根据你的喜好，你可以指定除 `rollup.config.js` 外的其他的配置文件：

```
rollup --config rollup.config.dev.js
rollup --config rollup.config.prod.js
```

### 安装本地的 Rollup

当处于团队合作或分布式环境时，添加 Rollup 作为本地依赖是非常适合的。安装本地 Rollup ，多个贡献者不需要再单独安装 Rollup, 保证所有贡献者使用相同的 Rollup 版本。

使用 NPM 安装本地 Rollup:

```
npm install rollup --save-dev
```

或者使用 Yarn:

```
yarn -D add rollup
```

在安装完成之后， 在项目根目录下可以运行 Rollup:

```
npx rollup --config
```

或者使用 Yarn：

```
yarn rollup --config
```

一旦安装完成，在 `package.json` 中会新增一个打包脚本，提供给所有贡献者一个便捷的命令。例如：

```json
{
  "scripts": {
    "build": "rollup --config"
  }
}
```

_注: 一旦本地安装完成，当调用 package 脚本中的命令时，NPM 和 Yarn 都会解析依赖的 bin 文件并且执行 Rollup。_

### 使用插件

到目前为止，我们已经基于一个入口文件和通过相对路径导入的模块创建了一个简单的 bundle。当你构建更复杂的 bundles 时，通常会需要更丰富的的灵活度 - 使用 NPM 安装导入模块，使用　Babel 编译代码，运行 JSON 文件等。

为此, 我们使用 *插件*, 插件将改变 Rollup 在打包过程中关键点的行为. 在 [the Rollup Awesome List](https://github.com/rollup/awesome) 中维护着一些优秀的插件。

在本篇教程中, 我们将使用 [@rollup/plugin-json](https://github.com/rollup/plugins/tree/master/packages/json), 其允许 Rollup 从一个 JSON 文件中导入数据。

在项目根目录下创建一个名叫 `package.json` 的文件, 并添加以下内容：

```json
{
  "name": "rollup-tutorial",
  "version": "1.0.0",
  "scripts": {
    "build": "rollup -c"
  }
}
```

安装 @rollup/plugin-json 作为开发态依赖:

```
npm install --save-dev @rollup/plugin-json
```

(使用 `--save-dev` 而不是 `--save` 是因为我们的代码在运行时并不依赖于插件，其只在我们打包 bundle 时用到。)

更新你的 `src/main.js` 文件，替换 `src/foo.js`，将 package.json 导入:

```js
// src/main.js
import { version } from '../package.json';

export default function () {
  console.log('version ' + version);
}
```

修改你的 `rollup.config.js` 文件 ，使其包含这个 JSON 插件:

```js
// rollup.config.js
import json from '@rollup/plugin-json';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [ json() ]
};
```

使用 `npm run build` 命令运行 Rollup。 运行结果应该类似为这样:

```js
'use strict';

var version = "1.0.0";

function main () {
  console.log('version ' + version);
}

module.exports = main;
```

_注: 只有我们确实需要的数据会被导入 – `name` 和 `devDependencies` 以及 `package.json` 中的其他部分是会被忽略的。这 **tree-shaking** 在起作用._

### 使用输出插件

一些插件专门应用于输出中。查阅 [plugin hooks](guide/en/#build-hooks) 了解关于输出-特定插件能做的更多技术细节。总的来说，这些插件仅仅只可以在 Rollup 的主要代码分析完成之后，才可以修改代码。如果一个不适用的插件被当作输出特定插件使用的时候，Rollup 会给出警告。一个可能出现的场景是，在浏览器中会使用压缩后的 bundles。

我们来扩展下之前的例子，提供一个压缩打包和一个不压缩打包的示例。紧接着， 安装 `rollup-plugin-terser`:

```
npm install --save-dev rollup-plugin-terser
```

修改你的 `rollup.config.js` 文件，增加第二项压缩输出。 压缩格式, 选择 `iife`。这种格式下包裹代码在浏览器中能够通过 `script` 标签自执行，避免其他代码的植入。当我们导出时，需要提供一个将通过 bundle 创建的全局变量名称，使导出的代码能被其他代码所使用。

```js
// rollup.config.js
import json from '@rollup/plugin-json';
import {terser} from 'rollup-plugin-terser';

export default {
  input: 'src/main.js',
  output: [
    {
      file: 'bundle.js',
      format: 'cjs'
    },
    {
      file: 'bundle.min.js',
      format: 'iife',
      name: 'version',
      plugins: [terser()]
    }
  ],
  plugins: [ json() ]
};
```

除了 `bundle.js` 之外，现在来创建 Rollup 的第二个文件 `bundle.min.js`:

```js
var version=function(){"use strict";var n="1.0.0";return function(){console.log("version "+n)}}();
```


### 代码分割

就代码分割来说, Rollup 有很多场景使用了代码分割自动成块，例如动态加载或者多入口, [`output.manualChunks`](guide/en/#outputmanualchunks) 配置项告诉了 Rollup 哪些模块需要分割成块。

为了使用代码分割特性来完成动态懒加载 (仅在一个函数执行之后加载其导入的模块), 回到最初的例子并修改 `src/main.js`，替换静态加载为 动态加载 `src/foo.js`:

```js
// src/main.js
export default function () {
  import('./foo.js').then(({ default: foo }) => console.log(foo));
}
```

Rollup 将采用动态导入的方式去构建一个仅需加载的分片块。为了能让 Rollup 知道哪里是第二个分块，替换 `--file` 可选项，我们使用 `--dir` 可选项设置一个文件夹用来输出:

```
rollup src/main.js -f cjs -d dist
```

它将会创建一个 `dist` 文件夹，其中包含两个文件: `main.js` 和 `chunk-[hash].js`，这里的 `[hash]` 是基于 hash 字符串内容。你可以通过指定 [`output.chunkFileNames`](guide/en/#outputchunkfilenames) 和 [`output.entryFileNames`](guide/en/#outputentryfilenames) 可选项提供自己的命名模式。

在相同输出之前你仍然可以运行你的代码。 `./foo.js` 的加载和编译尽管有一点慢，但是它只会在我们首次调取导出函数时执行一次。

```
node -e "require('./dist/main.js')()"
```

如果我们没有使用 `--dir` 可选项，Rollup 将会再次打印块到 `stdout` 中，添加注释来凸显这个块的边界:

```js
//→ main.js:
'use strict';

function main () {
  Promise.resolve(require('./chunk-b8774ea3.js')).then(({ default: foo }) => console.log(foo));
}

module.exports = main;

//→ chunk-b8774ea3.js:
'use strict';

var foo = 'hello world!';

exports.default = foo;
```

如果你想加载和编译仅一次使用的少见特性，这是有用的。

代码分割的另一个用途是能够区分多个入口，这些入口共享相同依赖。再次扩展我们的例子，增加第二个入口 `src/main2.js` 并像之前最原始的例子中的做法一样静态导入 `src/foo.js` :

```js
// src/main2.js
import foo from './foo.js';
export default function () {
  console.log(foo);
}
```

如果我们添加两个入口文件到 rollup 中，会构建出三个块:

```
rollup src/main.js src/main2.js -f cjs
```

将会输出

```js
//→ main.js:
'use strict';

function main () {
  Promise.resolve(require('./chunk-b8774ea3.js')).then(({ default: foo }) => console.log(foo));
}

module.exports = main;

//→ main2.js:
'use strict';

var foo_js = require('./chunk-b8774ea3.js');

function main2 () {
  console.log(foo_js.default);
}

module.exports = main2;

//→ chunk-b8774ea3.js:
'use strict';

var foo = 'hello world!';

exports.default = foo;
```

注意多个入口如何导入相同的共享块。Rollup 不会复制代码而是会插入额外的满足需求最小新增块。再次，通过 `--dir` 可选项写文件到桌面上。

你可以由不同浏览器规范：本地 ES 模块，AMD 加载或者 SystemJS 打包出相同的代码。

例如，使用 `-f es` 作为本地模块:

```
rollup src/main.js src/main2.js -f es -d dist
```

```html
<!doctype html>
<script type="module">
  import main2 from './dist/main2.js';
  main2();
</script>
```

或者可选的，用 `-f system` 可选项使用SystemJS:

```
rollup src/main.js src/main2.js -f system -d dist
```

安装 SystemJS

```
npm install --save-dev systemjs
```

然后，在一个所需的 HTML 页加载一个或者两个入口：

```html
<!doctype html>
<script src="node_modules/systemjs/dist/s.min.js"></script>
<script>
  System.import('./dist/main2.js')
  .then(({ default: main }) => main());
</script>
```

参见 [rollup-starter-code-splitting](https://github.com/rollup/rollup-starter-code-splitting) 示例，在有需要的情况下，如何设置一个在浏览器上使用本地 ES 模块的 Web app 回滚到 SystemJs。
