---
title: Tutorial
---

### 创建第一个 Bundle

*开始之前,需要安装 [Node.js](https://nodejs.org)， 这样才可以使用 [NPM](https://npmjs.com)；还需要了解如何使用 [command line](https://www.codecademy.com/learn/learn-the-command-line)。*

使用 Rollup 最简单的方法是通过 Command Line Interface （或 CLI）。先全局安装 Rollup （之后会介绍如何在项目中进行安装，更便于打包，但现在不用考虑这个问题）。在命令行中输入以下内容：

```
npm install rollup --global
# or `npm i rollup -g` for short
```

现在可以运行 `rollup` 命令了. 试试看!

```
rollup
```

由于没有传参, Rollup 打印出了使用说明。这和运行命令 `rollup --help`, or `rollup -h` 效果一样。

创建一个简单的项目:

```
mkdir -p my-rollup-project/src
cd my-rollup-project
```

首先，我们需要一个 *入口*。将以下内容粘贴到新建文件夹 `src/main.js` 中:

```js
// src/main.js
import foo from './foo.js';
export default function () {
  console.log(foo);
}
```

然后，创建 `foo.js` 在入口文件中进行导入:

```js
// src/foo.js
export default 'hello world!';
```

现在准备创建一个 bundle:

```
rollup src/main.js -f cjs
```

`-f` 可选项 (为 `--format`缩写) 明确我们正创建的是哪类 bundle — 在本项目中，CommonJS (将在 Node.js中运行)。由于我们没有指定一个输出文件，因此将打印到 `stdout` 中:

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

(也可以使用 `rollup src/main.js -f cjs > bundle.js`, 但是稍后将会明白，如果你使用生成映射的灵活度较低。)

试着运行代码：

```
node
> var myBundle = require('./bundle.js');
> myBundle();
'hello world!'
```

恭喜！你已经创建了你的第一个 Rollup 的 bundle 了。

### 使用配置文件

到目前为止，如此的顺利，但是当我们增加更多可选项到输出命令中时，它就会变得有些麻烦。

为了解决类似问题，我们可以创建一个包含有所有所需可选项的配置文件。使用 JavaScript 书写的配置文件比未经处理的 CLI 更灵活。

在项目根目录创建一个名为 `rollup.config.js` 的文件，并增加如下代码：

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

(注：你可以使用 CJS 模式 `module.exports = {/* config */}`)

为了使用这个配置文件, 我们可以用 `--config` 或者 `-c` 标记:

```
rm bundle.js # so we can check the command works!
rollup -c
```

你可以在配置文件的相同可选项行重写任意可选项:

```
rollup -c -o bundle-2.js # `-o` is equivalent to `--file` (formerly "output")
```

_注: Rollup 本身执行了配置文件, 因此我们能够使用 `export default` 语法 – 这些代码不能被 Babel 或者其他类似工具转换编译，因此你必须使用你正在使用的 Node.js 版本支持的 ES2015 特性语法。_

根据你的喜好，也可以由默认的 `rollup.config.js` 派生其他的配置文件：

```
rollup --config rollup.config.dev.js
rollup --config rollup.config.prod.js
```

### 安装本地的 Rollup

当处于团队合作或分布式环境时，添加 Rollup 作为本地依赖是非常合适的。安装本地 Rollup 能够使多个开发者不再需要单独安装 Rollup, 保证了所有开发者使用相同版本的 Rollup。

使用 NPM 安装本地 Rollup:

```
npm install rollup --save-dev
```

或者使用 Yarn:

```
yarn -D add rollup
```

在安装完成之后， 在项目根目录下可运行 Rollup:

```
npx rollup --config
```

或者使用 Yarn：

```
yarn rollup --config
```

一旦安装完成，在 `package.json` 中会增加一个打包脚本，提供给开发者一个简洁的命令。例如：

```json
{
  "scripts": {
    "build": "rollup --config"
  }
}
```

_注: 一旦本地安装完成，当调用 package 脚本中的命令时，NPM 和 Yarn 都会解析依赖的 bin 文件并且执行 Rollup。_

### 使用插件

到目前为止，我们已经创建了一个简单的 bundle，从一个入口文件到导入一个相对路径的模块。当创建更复杂的 bundles 时，需要更多的灵活性 - 使用 NPM　安装导入模块，使用　Babel 编译代码，运行 JSON 文件等。

为此, 使用 *插件*, 将改变 Rollup 在绑定进程中的重要点的行为. 在 [the Rollup Awesome List](https://github.com/rollup/awesome) 中维护着一些优秀的插件。

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

安装 @rollup/plugin-json 为开发依赖:

```
npm install --save-dev @rollup/plugin-json
```

(使用 `--save-dev` 而不是 `--save` 因为当它运行时，我们的代码并没有真正依赖于插件 - 仅仅在我们在绑定 bundle 时。)

更新 `src/main.js` 文件，使其导入来源为 package.json 替换原来的 `src/foo.js`:

```js
// src/main.js
import { version } from '../package.json';

export default function () {
  console.log('version ' + version);
}
```

编辑 `rollup.config.js` 文件 ，添加 JSON 插件:

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

使用 `npm run build`命令运行 Rollup。 运行结果类似如下内容:

```js
'use strict';

var version = "1.0.0";

function main () {
  console.log('version ' + version);
}

module.exports = main;
```

_注: 只有我们真正需要的数据会被导入 – `name` 和 `devDependencies` 以及 `package.json` 中的其他部分是会被忽略的。这 **tree-shaking** 在起作用._

### 使用输出插件

一些插件专门应用于输出中。查阅 [plugin hooks](guide/en/#build-hooks) 了解关于输出-特定插件能做的更多技术细节。这些插件作用范围较小，仅能在已经编译的 Rollup 主分析之后能更改代码。当输出-特定插件中使用了一个不兼容的插件时，Rollup 会给出警告。出现的可能场景为，在压缩要在浏览器中使用的 bundle 包。

扩展之前的示例，提供一个压缩打包和一个不压缩打包。紧接着， 安装 `rollup-plugin-terser`:

```
npm install --save-dev rollup-plugin-terser
```

编辑 `rollup.config.js` 文件，在输出中增加第二项。 格式上, 选择 `iife`。 该种格式下，覆盖的代码能够在浏览器中打上一个自定义 `脚本` 版本的标记，避免被其他代码的污染。因为已经有了一个导出，我们需要提供一个通过 bundle 创建的全局变量名称以便导出的代码能被其他代码使用。

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

除了 `bundle.js` 之外，现在创建 Rollup 的第二个文件 `bundle.min.js`:

```js
var version=function(){"use strict";var n="1.0.0";return function(){console.log("version "+n)}}();
```


### 代码分割

为了代码分割, Rollup 有很多场景使用代码分割自动成块，例如动态加载或者多入口, 使用在 [`output.manualChunks`](guide/en/#outputmanualchunks) 选项中配置的方式告诉了 Rollup 将哪些模块分割成块。

为了使用代码分割特性来完成动态懒加载 (仅在一个函数执行之后加载其导入的模块), 回到最初的示例并修改 `src/main.js`，替换静态加载为 动态加载 `src/foo.js`:

```js
// src/main.js
export default function () {
  import('./foo.js').then(({ default: foo }) => console.log(foo));
}
```

Rollup 将使用动态导入来创建一个所需加载的分片块。为了能让 Rollup 知道哪里是第二个分块，替换 `--dir` 使用`--file` 选项设置一个文件夹作为输出:

```
rollup src/main.js -f cjs -d dist
```

它将会创建一个 `dist` 文件，其中包含两个文件: `main.js` 和 `chunk-[hash].js`，这里的 `[hash]` 是 hash 字符串的基础内容。你可以通过指定 [`output.chunkFileNames`](guide/en/#outputchunkfilenames) 和 [`output.entryFileNames`](guide/en/#outputentryfilenames) 可选项提供自己的命名模式。

You can still run your code as before with the same output, albeit a little slower as loading and parsing of `./foo.js` will only commence once we call the exported function for the first time.

```
node -e "require('./dist/main.js')()"
```

如果我们没有使用 `--dir` 可选项，Rollup 将重复输出块到 `stdout` 中，添加注释来凸显块的边界:

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

如果你想加载和编译他们使用到的一次性特有属性，这是非常有用的。

代码分割的另一个用途是能够区分多个共享相同依赖的入口。再次扩展我们的示例，增加第二个入口 `src/main2.js` 并像之前最初示例类似做法静态导入 `src/foo.js` :

```js
// src/main2.js
import foo from './foo.js';
export default function () {
  console.log(foo);
}
```

如果我们同时添加两个入口文件到 rollup 中，会创建三个块:

```
rollup src/main.js src/main2.js -f cjs
```

将输出

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

注意多个入口如何导入相同的共享块。Rollup 将不会重复代码而会创建格外新增块来仅满足的最小需求。再次，通过 `--dir` 可选项写文件到桌面上。

你可以由不同浏览器本地 ES 模块，AMD 加载或者 SystemJS 方式打包出相同的代码。

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

或者可选的，用 `-f system`  来使用SystemJS:

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

参见 [rollup-starter-code-splitting](https://github.com/rollup/rollup-starter-code-splitting) 示例，如何设置一个 web app，该 web app 在浏览器上使用本地 ES 模块，如果需要它支持回滚到 SystemJs。
