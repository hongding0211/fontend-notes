# 优化代码运行性能

## Code Split

把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间。

常用的代码分离方法有三种：

* **入口起点**： 在 `entry` 中手动分离（见 output.md）
* **防止重复**：使用 `Entry dependencies` 或 `SplitChunksPppplugin` 去重分离
* **动态导入**：通过模块的内联函数来分离代码

### 入口起点

见 [output.md](output.md)，这种方法存在一些隐患：

* 如果 entry 中多个 chunk 包含同一个模块，那这个模块会被重复引入
* 不够灵活，不能动态地将核心应用逻辑中的代码拆分出来

### 防止重复

#### 入口依赖

> 略，详见：[代码分离 | webpack 中文文档 (docschina.org)](https://webpack.docschina.org/guides/code-splitting/)

#### SplitChunksPlugin

将公共的依赖模块提取出来

**webpack.config.js**

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
}
```

假设 index.js 和 print.js 都引入了 lodash，打包后，第三方库 lodash 已经被单独提取出来

```
asset index.0c57e77abda314a84ed1.js 17.8 KiB
asset print.16746bd586ccb72489d3.js 17.4 KiB
vendors-node_modules_lodash_lodash_js.196895f8828735c59bdd.js 1.37 MiB 
```

对比不提取时候的体积

```
asset index.2374a1266dfc8e3e369f.js 1.38 MiB
asset print.fd5a45164084432521fd.js 1.38 MiB 
```

### 动态导入

#### 使用 import()

**webpack.config.js**

```js
module.exports = {
  entry: './src/index.js'
}
```

**index.js**

```js
async function component() {
  const element = document.createElement('div')
  const { default: _ } = await import('lodash')   // 使用 Promise 动态的 import
  element.innerHTML = _.join(['Hello', 'Webpack'], ' ')
  const btn = document.createElement('button')
  return element
}
component().then(component => document.body.appendChild(component))
```

#### 预获取/预加载模块

> 略，详见：[代码分离 | webpack 中文文档 (docschina.org)](https://webpack.docschina.org/guides/code-splitting/)

## Cache

若打包的 bundle 文件名不改变，浏览器就会命中缓存

### 修改输出的文件名

使用文件内容的 hash 值来命名 bundle，使得在 bundle 不变时，hash 值也不改变

**webpack.config.js**

```js
output: {
  filename: '[name].[contenthash].js'
},
```

输出的文件：

```bash
asset main.f4fb0c69a973e7ecc535.js 1.38 MiB [emitted] [immutable] (name: main)
```

> 💡 然而这样的方法并不能完全确保**在文件内容不改变时保持 hash 值不变**，因为入口 chunk 中可能包含了某些引导模板

### 提取引导模板 (extracting boilerplate)

使用 `SplitChunksPlugin` 可以将模块分离到单独的 bundle 中。利用 `optimization.runtimeChunk` 将 runtime 代码拆分为一个单独的 chunk

**webpack.config.js**

```js
module.exports = {
  entry: './src/index.js',
  optimization: {
    runtimeChunk: 'single',
  },
};
```

构建后，runtime 代码已经被提取出来

```bash
asset runtime.bf1d01aa56a316de9c1b.js 15.5 KiB [emitted] [immutable] (name: runtime)
```

接着，我们还可以将第三方库（如 react、lodash）的代码提取到单独的 `vendor` chunk 中

**webpack.config.js**

```js
optimization: {
  runtimeChunk: 'single',
  splitChunks: {
    chunks: 'all'
  }
}
```

再次构建，不仅 runtime 代码被提取出来，第三方库也被提取了出来

```
asset runtime.bf1d01aa56a316de9c1b.js 15.5 KiB
asset index.217c743aea23034fef65.js 1.98 KiB 
asset print.fce4bfb7ed7994ea07c8.js 1.67 KiB 
vendors-node_modules_lodash_lodash_js.196895f8828735c59bdd.js 1.37 MiB
```

### 模块标识符\*

`vendor` bundle 会随着自身的 `module.id` 的变化，而发生变化，但这不符合需求

```js
optimization: {
  moduleIds: 'deterministic'
}
```

## Tree Shaking

> Tree Shaking 是一种通过消除最终文件中未使用的代码来优化体积的方法。
>
> 你可以将应用程序想象成一棵树。绿色表示实际用到的 source code(源码) 和 library(库)，是树上活的树叶。灰色表示未引用代码，是秋天树上枯萎的树叶。为了除去死去的树叶，你必须摇动这棵树，使它们落下。

**webpack.config.js**

```js
optimization: {
  usedExports: true
}
```

**math.js**

```js
export function square(x) {			// 不会被调用的函数
  return x * x;
}

export function cube(x) {
  return x * x * x;
}
```

**bundle.js**

```js
// 没有被引用到的函数会被打上注释标签（development环境下）
/* unused harmony export square */
function square(x) {
  return x * x;
}
```

### side-effect-free

`side-effect-free` 即 `没有副作用`，是指 tree shaking 不会错误地把代码给剔除掉。但是真实中的项目不是所有的代码都是这么的纯粹，是 `side-effect-free` 的。

因此，要在 **package.json** 中，指定那些`不纯粹`即不是`side-effect-free`的文件

```json
"sideEffects": ["xxx.js", "*.css"]	// css 文件就是 none-side-effect-free 的
```

当然，如果所有的文件都是 `side-effect-free`，那么在 **package.json** 中直接指定

```json
"sideEffects": false
```

最后，要将 mode 切换为 production，才会真正的将不需要的代码给剔除

#### 总结

* 必须依赖 ES6 的 `import` `export` 特性
* 确保编译器没有将 ES6 转换为 CommonJS
* 要添加 `sideEffects` 属性
* 使用 `production` mode

## lazy load

```js
button.onclick = e => import(/* webpackChunkName: "print" */ './print').then(module => {
	const print = module.default;
  print();
}
```
