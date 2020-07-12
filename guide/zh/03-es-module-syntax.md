---
title: ES 模块语法
---

下面内容旨在对 [ES2015 规范](https://www.ecma-international.org/ecma-262/6.0/) 中定义的模块行为做简要介绍，正确理解导入和导出语句对于使用好 Rollup 至关重要。

### 导入（Importing）

导入的值不能重新赋值，尽管导入的对象或者数组*允许*被修改（这样导出的模块，以及其他的导入，都将受到该修改的影响）。在这种情况下，他们的行为和 `const` 声明类似。


#### 命名导入（Named Imports）

从源模块中导入其原始命名的特定项。

```js
import { something } from './module.js';
```

从源模块中导入特定项，并在导入时重命名。

```js
import { something as somethingElse } from './module.js';
```

#### 命名空间导入（Namespace Imports）

将源模块中的所有项作为一个对象导入，源模块中所有的命名导出将作为属性和方法暴露在这个对象上。

```js
import * as module from './module.js'
```

上面的例子中的 `something` 将被作为属性赋值到导入的对象上，像这样 `module.something`。如果存在默认导出项，可以通过`module.default`得到默认导出项。

#### 默认导入（Default Import）

导入源模块中的**默认导出项**

```js
import something from './module.js';
```

#### 空的导入（Empty Import）

加载模块代码，但不创建任何新的变量

```js
import './module.js';
```

这个对于 `polyfill` 很有用，或者导入的代码主要的目的是为了处理原型相关的。

#### 动态导入（Dynamic Import）

使用 [动态导入 API](https://github.com/tc39/proposal-dynamic-import#import) 导入模块。

```js
import('./modules.js').then(({ default: DefaultExport, NamedExport })=> {
  // 用模块做一些处理
})
```

这对于需要使用代码分割的应用和使用动态模块会很有用。

### 导出（Exporting）

#### 命名导出（Named exports）

导出一个先前声明过的值：

```js
const something = true;
export { something };
```

在导出时重命名：

```js
export { something as somethingElse };
```

声明后立即导出：

```js
// 可以与 var, let, const, class, 和 function 配合使用
export const something = true;
```


#### 默认导出（Default Export）

导出单个值作为源模块的默认导出：

```js
export default something;
```

仅当源模块只有一个导出项时，才建议使用此写法.

将默认和命名导出组合在同一模块中是不好的做法，尽管它是规范允许的。


### 绑定是如何工作的（How bindings work）

ES 模块导出的是 *动态绑定*，而不是值，因此在最初导入值后，可以对其进行更改，来看这个[示例](https://rollupjs.org/repl/?shareable=JTdCJTIybW9kdWxlcyUyMiUzQSU1QiU3QiUyMm5hbWUlMjIlM0ElMjJtYWluLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmltcG9ydCUyMCU3QiUyMGNvdW50JTJDJTIwaW5jcmVtZW50JTIwJTdEJTIwZnJvbSUyMCcuJTJGaW5jcmVtZW50ZXIuanMnJTNCJTVDbiU1Q25jb25zb2xlLmxvZyhjb3VudCklM0IlNUNuaW5jcmVtZW50KCklM0IlNUNuY29uc29sZS5sb2coY291bnQpJTNCJTIyJTdEJTJDJTdCJTIybmFtZSUyMiUzQSUyMmluY3JlbWVudGVyLmpzJTIyJTJDJTIyY29kZSUyMiUzQSUyMmV4cG9ydCUyMGxldCUyMGNvdW50JTIwJTNEJTIwMCUzQiU1Q24lNUNuZXhwb3J0JTIwZnVuY3Rpb24lMjBpbmNyZW1lbnQoKSUyMCU3QiU1Q24lNUN0Y291bnQlMjAlMkIlM0QlMjAxJTNCJTVDbiU3RCUyMiU3RCU1RCUyQyUyMm9wdGlvbnMlMjIlM0ElN0IlMjJmb3JtYXQlMjIlM0ElMjJjanMlMjIlMkMlMjJnbG9iYWxzJTIyJTNBJTdCJTdEJTJDJTIybW9kdWxlSWQlMjIlM0ElMjIlMjIlMkMlMjJuYW1lJTIyJTNBJTIybXlCdW5kbGUlMjIlN0QlMkMlMjJleGFtcGxlJTIyJTNBbnVsbCU3RA==)：

```js
// incrementer.js
export let count = 0;

export function increment() {
  count += 1;
}
```

```js
// main.js
import { count, increment } from './incrementer.js';

console.log(count); // 0
increment();
console.log(count); // 1

count += 1; // 错误 — 只有 incrementer.js 才能更改它
```
