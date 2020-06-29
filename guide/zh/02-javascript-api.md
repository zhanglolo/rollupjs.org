---
title: JavaScript API
---

Rollup 提供了可从 Node.js 使用的 JavaScript API。 您几乎不需要使用它，并且应该使用命令行 API，除非您要扩展Rollup本身或者使用它进行一些深奥的操作，比如通过编程生成 bundle。

### rollup.rollup

`rollup.rollup` 函数接收输入选项对象作为参数，并返回一个 Promise，该 Promise 解析为具有各种属性和方法的 `bundle` 对象，如下所示。 在此步骤中，Rollup 将构建模块图并执行 tree-shaking，但不会生成任何输出。

在 `bundle` 对象上，您可以使用不同的输出选项对象多次调用 `bundle.generate`，以在内存中生成不同的 bundle。 如果直接将它们写入磁盘，请改用 `bundle.write`。

```javascript
const rollup = require('rollup');

// 有关选项的详细信息，请参见下文
const inputOptions = {...};
const outputOptions = {...};

async function build() {
  // 创建一个 bundle
  const bundle = await rollup.rollup(inputOptions);

  console.log(bundle.watchFiles); // 该 bundle 依赖的文件名数组

  // 在内存中生成输出特定的代码
  // 您可以在同一个 bundle 对象上多次调用此函数
  const { output } = await bundle.generate(outputOptions);

  for (const chunkOrAsset of output) {
    if (chunkOrAsset.type === 'asset') {
      // 对于assets，包含
      // {
      //   fileName: string,              // asset 文件名
      //   source: string | Uint8Array    // asset 资源
      //   type: 'asset'                  // 表示这是一个 asset
      // }
      console.log('Asset', chunkOrAsset);
    } else {
      // 对于chunks, 包含
      // {
      //   code: string,                  // 生成的JS代码
      //   dynamicImports: string[],      // chunk 动态导入的外部模块
      //   exports: string[],             // 导出的变量名
      //   facadeModuleId: string | null, // 该chunk对应的模块的ID
      //   fileName: string,              // chunk的文件名
      //   imports: string[],             // chunk 静态导入的外部模块
      //   isDynamicEntry: boolean,       // 该 chunk 是否是动态入口点
      //   isEntry: boolean,              // 该 chunk 是否是静态入口点
      //   map: string | null,            // sourcemaps(如果存在)
      //   modules: {                     // 此 chunk 中模块的信息
      //     [id: string]: {
      //       renderedExports: string[]; // 导出的已包含变量名
      //       removedExports: string[];  // 导出的已删除变量名
      //       renderedLength: number;    // 模块中剩余代码的长度
      //       originalLength: number;    // 模块中代码的原始长度
      //     };
      //   },
      //   name: string                   // 命名模式中使用的 chunk 的名称
      //   type: 'chunk',                 // 表示这是一个chunk
      // }
      console.log('Chunk', chunkOrAsset.modules);
    }
  }

  // 或者将bundle写入磁盘
  await bundle.write(outputOptions);
}

build();
```

#### 输入选项对象(inputOptions object)

`inputOptions` 对象可以包含下列属性(查看[big list of options](guide/en/#big-list-of-options) 以获得这些参数更详细的信息)：

```js
const inputOptions = {
  // 核心输入选项
  external,
  input, // 必选
  plugins,

  // 高级输入选项
  cache,
  onwarn,
  preserveEntrySignatures,
  strictDeprecations,

  // 危险选项
  acorn,
  acornInjectPlugins,
  context,
  moduleContext,
  preserveSymlinks,
  shimMissingExports,
  treeshake,

  // 实验型选项
  experimentalCacheExpiry,
  perf
};
```

#### 输出选项对象(outputOptions object)

`outputOptions` 对象可以包含下列属性(查看[big list of options](guide/en/#big-list-of-options) 以获得这些参数更详细的信息)：

```js
const outputOptions = {
  // 核心输出选项
  dir,
  file,
  format, // 必选
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
  externalLiveBindings,
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

  // 危险选项
  amd,
  esModule,
  exports,
  freeze,
  indent,
  namespaceToStringTag,
  noConflict,
  preferConst,
  strict,
  systemNullSetters
};
```

### rollup.watch

Rollup 也提供了 `rollup.watch` 函数，当它检测到磁盘上单个模块已经改变，它会重新构建你的 bundle。 当你通过命令行运行 Rollup，并带上 "--watch" 标记时，它将在内部使用。

```js
const rollup = require('rollup');

const watchOptions = {...};
const watcher = rollup.watch(watchOptions);

watcher.on('event', event => {
  // event.code 会是下面其中一个：
  //   START        — 监听器正在启动（重启）
  //   BUNDLE_START — 构建单个 bundle
  //   BUNDLE_END   — 完成 bundle 构建
  //   END          — 完成所有bundle构建
  //   ERROR        — 构建时遇到错误
});

// 停止监听
watcher.close();
```

#### 监听选项(watchOptions)

`watchOptions` 参数是从配置文件导出的配置(或配置数组)。

```js
const watchOptions = {
  ...inputOptions,
  output: [outputOptions],
  watch: {
    buildDelay,
    chokidar,
    clearScreen,
    skipWrite,
    exclude,
    include
  }
};
```

查看以上文档了解更多 `inputOptions` 和 `outputOptions` 的细节, 或参考 [big list of options](guide/en/#big-list-of-options) 获取与 `chokidar`, `include` 和 `exclude` 有关的信息。

#### 以编程方式加载配置文件

为了帮助生成这样的配置，rollup 通过一个单独的入口点公开了用于在命令行界面中加载配置文件的帮助程序。 该帮助程序接收解析的 `fileName` 和可选的包含命令行参数的对象：

```js
const loadConfigFile = require('rollup/dist/loadConfigFile');
const path = require('path');
const rollup = require('rollup');

// 加载当前脚本旁边的配置文件；
// 提供的配置对象与在命令行上传递 "--format es" 具有相同的效果，并将覆盖所有输出的格式
loadConfigFile(path.resolve(__dirname, 'rollup.config.js'), { format: 'es' })
  .then(async ({options, warnings}) => {
    // “warnings”包装了CLI传递的默认`onwarn`处理程序。
    // 输出所有警告：
    // "warnings" wraps the default `onwarn` handler passed by the CLI.
    // This prints all warnings up to this point:
    console.log(`We currently have ${warnings.count} warnings`);

    // 输出所有延迟的警告：
    // This prints all deferred warnings
    warnings.flush();
    
    // options是一个带有其他“output”属性的“ inputOptions”对象，该属性包含一个“ outputOptions”数组。
    // 以下将生成所有输出，并将它们以与CLI相同的方式写入磁盘：
    // options is an "inputOptions" object with an additional "output"
    // property that contains an array of "outputOptions".
    // The following will generate all outputs and write them to disk the same
    // way the CLI does it:
    const bundle = await rollup.rollup(options);
    await Promise.all(options.output.map(bundle.write));
    
    // 您也可以将其直接传递给“ rollup.watch”
    // You can also pass this directly to "rollup.watch"
    rollup.watch(options);
  })
```
