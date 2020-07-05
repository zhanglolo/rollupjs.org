---
title: 命令行接口
---

我们一般在命令行中使用 Rollup。你也可以通过一份可选的 Rollup 配置文件来简化命令行操作，同时还可以通过配置文件启用 Rollup 的高级特性。

### 配置文件

Rollup 的配置文件并不是必须的，但是配置文件非常强大而且很方便，所以我们**推荐你能够使用**。配置文件是一个 ES 模块，它对外导出一个对象，这个对象配置了我们想要的一些选项：

```javascript
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

通常，这个配置文件位于你项目的根目录，并且命名为 `rollup.config.js`。Rollup 将会在配置文件被依赖之前，在后台将配置文件和 CommonJS 的相关依赖项进行转译和打包，这样做的优点就是你可以与 ES 模块代码库共享代码，同时与 Node 生态完全互通。

如果你想要使用 `require` 和 `module.exports` 将配置文件写成一个 CommonJS 模块，则应该将文件扩展名更改为 `.cjs`，这会阻止 Rollup 尝试转译文件。此外，如果你使用的是 Node 13+，则将文件扩展名更改为 `.mjs` 也可以阻止 Rollup 进行编译，并将文件导入为 ES 模块。有关更多的详细信息以及为什么会存在这样的场景，请参见 [使用未转译的配置文件](guide/en/#using-untranspiled-config-files)。

配置文件支持的选项如下所示。关于每个选项的详情请查看 [配置项的完整列表](guide/en/#big-list-of-options)：

```javascript
// rollup.config.js

export default { // 可以是一个数组（用于多个输入的情况）
  // 核心的输入选项
  external,
  input, // required
  plugins,

  // 高级输入选项
  cache,
  onwarn,
  preserveEntrySignatures,
  strictDeprecations,

  // 危险区
  acorn,
  acornInjectPlugins,
  context,
  moduleContext,
  preserveSymlinks,
  shimMissingExports,
  treeshake,

  // 实验性
  experimentalCacheExpiry,
  perf,

  output: { // required (可能是一个数组，用于多输出的情况)
    // 核心的输出选项
    dir,
    file,
    format, // required
    globals,
    name,
    plugins,

    // 高级输出选项
    assetFileNames,
    banner,
    chunkFileNames,
    compact,
    entryFileNames,
    extend,
    footer,
    hoistTransitiveImports,
    inlineDynamicImports,
    interop,
    intro,
    manualChunks,
    minifyInternalExports,
    outro,
    paths,
    preserveModules,
    sourcemap,
    sourcemapExcludeSources,
    sourcemapFile,
    sourcemapPathTransform,

    // 危险区
    amd,
    esModule,
    exports,
    externalLiveBindings,
    freeze,
    indent,
    namespaceToStringTag,
    noConflict,
    preferConst,
    strict,
    systemNullSetters
  },

  watch: {
    buildDelay,
    chokidar,
    clearScreen,
    skipWrite,
    exclude,
    include
  } | false
};
```

甚至在监听模式下，如果你想一次从几个无关的输入项构建 bundle，就可以在配置文件中导出一个**数组**。而为了使用相同的输入项构建不同的 bundle，你可以为每个输入项提供一个输出项的数组。

```javascript
// rollup.config.js (构建多个 bundle)

export default [{
  input: 'main-a.js',
  output: {
    file: 'dist/bundle-a.js',
    format: 'cjs'
  }
}, {
  input: 'main-b.js',
  output: [
    {
      file: 'dist/bundle-b1.js',
      format: 'cjs'
    },
    {
      file: 'dist/bundle-b2.js',
      format: 'es'
    }
  ]
}];
```

如果你想异步创建配置，Rollup 也能处理结果是一个对象或者数组的 `Promise`。

```javascript
// rollup.config.js
import fetch from 'node-fetch';
export default fetch('/some-remote-service-or-file-which-returns-actual-config');
```

Similarly, you can do this as well:

```javascript
// rollup.config.js (Promise resolving an array)
export default Promise.all([
  fetch('get-config-1'),
  fetch('get-config-2')
])
```

如果你想使用 Rollup 的配置文件，可以在命令行加上 `--config` 或者 `-c` 的选项。

```
# 将自定义配置文件的路径传给 Rollup
rollup --config my.config.js

# 如果你不传文件名, Rollup 将会尝试
# 按照以下顺序加载配置文件：
# rollup.config.mjs -> rollup.config.cjs -> rollup.config.js
rollup --config
```

你也可以导出返回上述任何配置格式的函数。这个函数会接收当前命令行的参数，因此你可以动态调整配置以遵守 [`--silent`](guide/en/#--silent)。甚至你还可以在命令行加上 `config` 前缀来定义自己的命令行选项：

```javascript
// rollup.config.js
import defaultConfig from './rollup.default.config.js';
import debugConfig from './rollup.debug.config.js';

export default commandLineArgs => {
  if (commandLineArgs.configDebug === true) {
    return debugConfig;
  }
  return defaultConfig;
}
```

如果你现在运行 `rollup --config --configDebug`，就会使用 debug 配置。

默认情况下，命令行参数始终会覆盖从配置文件中导出的各个值。如果你想改变这样的方式，可以在 `commandLineArgs` 对象中删除命令行参数，这样使得 Rollup 会忽略命令行参数：

```javascript
// rollup.config.js
export default commandLineArgs => {
  const inputBase = commandLineArgs.input || 'main.js';

  // 这会使 Rollup 忽略命令行参数
  delete commandLineArgs.input;
  return {
    input: 'src/entries/' + inputBase,
    output: {...}
  }
}
```

### 与 JavaScript API 的区别

尽管配置文件提供了配置 Rollup 的简单方式，但是它也限制了 Rollup 如何能被调用以及在何处进行的配置。尤其是如果你想要在另一个构建工具中重新打包 Rollup，或者想要把它集成到高级的构建进程中，那么最好是在脚本中用编程的方式调用 Rollup。

如果你想从配置文件切换为使用 [JavaScript API](guide/en/#javascript-api)，这里有一些需要注意的重要区别：

- 使用 JavaScript API 时，传给 `rollup.rollup` 的配置必须是一个对象，并且不能包装在 Promise 或者函数中。
- 你不能再使用数组作为配置。取而代之的是，你应该为每组 `inputOptions` 都运行一次 `rollup.rollup`。
- `output` 配置将会会被忽略。取而代之的是，你应该为每组 `outputOptions` 都运行一次 `bundle.generate(outputOptions)` 或者 `bundle.write(outputOptions)`。

### 从 Node 包中加载配置

为了实现互通性，Rollup 也支持从安装在 `node_modules` 目录下的包中加载配置文件。

```
# Rollup 首先会尝试加载 "rollup-config-my-special-config";
# 如果失败，Rollup 则会尝试加载 "my-special-config"
rollup --config node:my-special-config
```

### 使用未转译的配置文件

默认情况下，Rollup 会希望配置文件是 ES 模块，并在依赖它们之前，把它们和它们 CommonJS 的相关导入打包和转译。这是一个很快的过程，其优点就是易于在你的配置和 ES 模块代码库之间共享代码。如果你想写 CommonJS 的配置来替代，可以通过使用 `.cjs` 的拓展名来跳过这个过程。

```javascript
// rollup.config.cjs
module.exports = {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

如果不仅要在命令行中使用配置文件，也要在自定义脚本中以编程的方式使用，那么这可能是合适的。

另一方面，如果你使用的是 Node 13 或者更高版本，并且在你的 `package.json` 中有 `"type": "module"`，Rollup 的转译过程将阻止你的配置文件导入本身就是 ES 模块的 package。在这种情况下，将文件拓展名改为 `.mjs` 将指示 Rollup 将配置直接导入为 ES 模块。但是请注意，这只在 Node 13+ 有效；在旧版本的 Node 环境下，对待 `.mjs` 就像对待 `.js`。

在 Node 13+ 环境下，使用 `.mjs` 会有一些潜在的陷阱：

- 你只能从 CommonJS 插件中获取默认的导出。
- 你可能无法导入 JSON 文件，例如：`package.json file`。有两种方式可以解决这个问题
  - 通过以下方式运行 Rollup

    ```
    node --experimental-json-modules ./node_modules/.bin/rollup --config
    ```
    
  - 创建一个依赖 JSON 文件的 CommonJS 的包装器:
  
    ```js
    // load-package.cjs
    module.exports = require('./package.json');
    
    // rollup.config.mjs
    import pkg from './load-package.cjs';
    ...
    ```

### 命令行标志

很多选项都有等价的命令行参数。如果你使用的话，此处传递的所有参数都将覆盖配置文件。这是所有支持的选项列表：

```text
-c, --config <filename>     使用配置文件（如果使用参数但是值没有
                              指定, 默认就是 rollup.config.js）
-d, --dir <dirname>         构建块的目录（如果不存在，就打印到标准输出）
-e, --external <ids>        逗号分隔列出排除的模块 ID
-f, --format <format>       输出类型 (amd, cjs, es, iife, umd, system)
-g, --globals <pairs>       逗号分隔列出 `moduleID:Global` 对
-h, --help                  显示帮助信息
-i, --input <filename>      输入 (替代 <entry file>)
-m, --sourcemap             生成 sourcemap (`-m inline` 生成行内 map)
-n, --name <name>           UMD 导出的名字
-o, --file <output>         单个的输出文件（如果不存在，就打印到标准输出）
-p, --plugin <plugin>       使用指定的插件（可能重复）
-v, --version               显示版本号
-w, --watch                 监听 bundle 中的文件并在文件改变时重新构建
--amd.id <id>               AMD 模块 ID（默认是匿名的）
--amd.define <name>         代替 `define` 使用的功能
--assetFileNames <pattern>  构建的资源命名模式
--banner <text>             插入 bundle 顶部（包装器之外）的代码
--chunkFileNames <pattern>  次要构建块的命名模式
--compact                   压缩包装器代码
--context <variable>        指定顶层的 `this` 值
--entryFileNames <pattern>  入口构建块的命名模式
--environment <values>      设置传递到配置文件 (看示例)
--no-esModule               不增加 __esModule 属性
--exports <mode>            指定导出的模式 (auto, default, named, none)
--extend                    通过 --name 定义，拓展全局变量
--no-externalLiveBindings   不生成实施绑定的代码
--footer <text>             插入到 bundle 末尾的代码（包装器外部）
--no-freeze                 不冻结命名空间对象
--no-hoistTransitiveImports 不提升传递性的导入到入口构建块
--no-indent                 结果中不进行缩进
--no-interop                不包含互操作块
--inlineDynamicImports      使用动态导入时创建单个 bundle
--intro <text>              在 bundle 顶部插入代码（包装器内部）
--minifyInternalExports     强制或者禁用内部导出的压缩
--namespaceToStringTag      为命名空间创建正确的 `.toString` 方法
--noConflict                为 UMD 全局变量生成 noConflict 方法
--outro <text>              在 bundle 的末尾插入代码（包装器内部）
--preferConst               使用 `const` 代替 `var` 进行导出
--no-preserveEntrySignatures 避免表面的构建块作为入口
--preserveModules           保留模块结构
--preserveSymlinks          解析文件时不要遵循符号链接
--shimMissingExports        给丢失的导出创建填充变量
--silent                    不打印警告
--sourcemapExcludeSources   source map 中不包含源码
--sourcemapFile <file>      source map 中指定 bundle 的路径
--no-stdin                  不从标准输入中读取 "-"
--no-strict                 在生成的模块中不使用 `"use strict";`
--strictDeprecations        不推荐使用的特性抛出错误
--systemNullSetters         用 `null` 替换空的 SystemJS setter
--no-treeshake              禁用 tree-shaking 优化
--no-treeshake.annotations  忽略纯的调用注释
--no-treeshake.moduleSideEffects 假设模块没有副作用
--no-treeshake.propertyReadSideEffects 忽略属性访问的副作用
--no-treeshake.tryCatchDeoptimization 不关闭 try-catch-tree-shaking
--no-treeshake.unknownGlobalSideEffects 假设未知的全局变量不抛出
--waitForBundleInput        等待 bundle 的输入文件
--watch.buildDelay <number> 监听重新构建的延时
--no-watch.clearScreen      重新构建时不进行清屏
--watch.skipWrite           监听时不写入文件到磁盘
--watch.exclude <files>     监听时排除的文件
--watch.include <files>     限制监听指定的文件
```

下面列出来的选项只能在命令行接口使用。其他所有的选项都一一对应并会覆盖配置文件的等价配置项，具体细节可以参考 [配置项的完整列表](guide/en/#big-list-of-options)。

#### `-h`/`--help`

打印帮助文档。

#### `-p <plugin>`, `--plugin <plugin>`

使用指定的插件。这里有几种方法可以指定插件：

- 通过相对路径：

  ```
  rollup -i input.js -f es -p ./my-plugin.js
  ```

  这个文件应该导出一个 plugin 对象或者返回 plugin 对象的函数。
- 通过安装在本地或者全局的 `node_modules` 文件夹的插件名字：

  ```
  rollup -i input.js -f es -p @rollup/plugin-node-resolve
  ```

  如果插件的名字不是以 `rollup-plugin-` 或者 `@rollup/plugin-` 开头，Rollup 会自动尝试添加这些前缀：

  ```
  rollup -i input.js -f es -p node-resolve
  ```

- 通过内联实现：

  ```
  rollup -i input.js -f es -p '{transform: (c, i) => `/* ${JSON.stringify(i)} */\n${c}`}'
  ```

如果想要加载多个插件，则可以重复该选项，或者提供按逗号分隔的名称列表：

```
rollup -i input.js -f es -p node-resolve -p commonjs,json
```

默认情况下，导出方法的插件在创建时被调用应该是不接收参数的。然而你也可以传递自定义的参数：

```
rollup -i input.js -f es -p 'terser={output: {beautify: true, indent_level: 2}}'
```

#### `-v`/`--version`

打印安装的版本号。

#### `-w`/`--watch`

当源文件在磁盘上发生更改时，重新构建 bundle。

_注意: 在监听模式下，环境变量 `ROLLUP_WATCH` 将会被 Rollup 的命令行接口设置成 `"true"`，并且可以被其他进程所检查到。插件应该改为检查 [`this.meta.watchMode`](guide/en/#thismeta-rollupversion-string-watchmode-boolean)，因为它独立于命令行接口。_

#### `--silent`

不在控制台打印警告。如果配置文件包含一个 `onwarn` 的处理方法，那么这个方法就会被调用。要手动阻止这种情况，你可以按照 [配置文件](guide/en/#configuration-files) 末尾的说明访问配置文件中的命令行选项。

#### `--environment <values>`

通过 `process.ENV` 传递额外的设置给配置文件。

```sh
rollup -c --environment INCLUDE_DEPS,BUILD:production
```

将设置 `process.env.INCLUDE_DEPS === 'true'` 和 `process.env.BUILD === 'production'`。你可以多次使用这个选项。在这种情况下，后面设置的变量将覆盖前面的定义。这保证了你可以覆盖 `package.json` 脚本中的环境变量：

```json
// in package.json
{
  "scripts": {
    "build": "rollup -c --environment INCLUDE_DEPS,BUILD:production"
  }
}
```

如果通过以下的方式调用脚本：

```
npm run build -- --environment BUILD:development
```

那么配置文件就会接收 `process.env.INCLUDE_DEPS === 'true'` 和 `process.env.BUILD === 'development'`。

#### `--waitForBundleInput`

如果其中一个入口文件不可访问，将不会抛出错误。相反，它会等到所有文件都存在后再开始构建。当 Rollup 正在消费另一个进程的输出时，这个功能很有用，尤其是在监听模式下。

#### `--no-stdin`

不从 `stdin` 读取文件。设置这个标志可以阻止传递内容到 Rollup，也保证了 Rollup 将 `-` 解释为常规文件名，而不是将其解释为 `stdin`。另见 [从标准输入读取文件](guide/en/#reading-a-file-from-stdin)。

### 从标准输入中读取文件

使用命令行界面时，Rollup 也能从标准输入读取内容：

```
echo "export const foo = 42;" | rollup --format cjs --file out.js
```

当文件包含导入时，Rollup 将尝试相对于当前工作路径来解析它们。当使用配置文件时，如果入口的文件名是 `-`，Rollup 将仅用 `stdin` 作为入口。要从标准输入读取一个非入口的文件，只需要给它取名为 `-`，它是内部引用标准输入的文件名。即：

```js
import foo from "-";
```

在任何文件中，都将促使 Rollup 尝试从 `stdin` 读取导入文件，并且将其默认导出赋值给 `foo`。你可以通过 [`--no-stdin`](guide/en/#--no-stdin) 命令行选项来使得 Rollup 像对待常规文件名一样对待 `-`。

JavaScript API 始终将 `-` 视为普通文件名。
