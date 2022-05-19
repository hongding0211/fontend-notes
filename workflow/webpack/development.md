# 开发环境配置

### Webpack Dev Server

```bash
yarn add -D webpack-dev-server
```

**webpack.config.js**

```js
module.exports = {
  ...,
  devServer: {
  	static: './dist',  // 将 dist 目录下的文件 serve 到服务器上
  	port: xxxx,
  	open: true				// 自动打开浏览器
  }
}
```

> Webpack Dev Server 只是将编译后的文件保存在内存中，并不真实输出至磁盘中

不妨在 **package.json** 中增加一个脚本

```json
"start": "webpack serve --open"
```

### HMR 模块热加载

> ⚠️ HMR 只适用于开发环境

```js
module.exports = {
  ...,
  devServer: {
  	...,
    hot: true,
  }
}
```

从 `webpack-dev-server` v4.0.0 开始，热模块替换是默认开启的。

`style-loader` 默认支持 HMR

#### js 支持 HMR

在 `index.js` 中：

```js
import xxxFoo() from './xxx.js'

if (module.hot) {
   module.hot.accept('./xxx.js', function() {
     xxxFoo()	// xxx.js 中的某一方法
   })
}
```

这样，`xxx.js`就会被监听，如果文件内容发生改变，就会重新调用 `xxxFoo()`

> ⚠️ 入口文件 index.js 不支持热加载，但是在 index.js 引入的其他 js 模块是可以支持热加载的

### source map

将源代码和构建后的代码进行映射，使得在出错是可以定位到错误在源代码中的位置

在 `webpack.config.js` 中：

```js
module.exports = {
  ...,
  devtool: 'source-map'
}
```

#### 不同的 source map

> 💡 不同的 source map 会明显影响到 build 和重新构建 rebuild 的速度
>
> ​ https://webpack.docschina.org/configuration/devtool/

| xxx-source -map | 性能 (build/rebuild) | 是否适用于 production |      品质     | 备注                 |
| :-------------: | :----------------: | :--------------: | :---------: | ------------------ |
|        默认       |        最慢/最慢       |         ✅        |   original  |                    |
|       eval      |        最慢/尚可       |         ❌        |   original  | 适合开发环境             |
|      cheap      |        尚可/慢        |         ❌        | transformed |                    |
|      inline     |        最慢/最慢       |         ❌        |   original  | 适合发布单一文件           |
|    nosources    |        最慢/最慢       |         ✅        |   original  | 不包含源代码             |
|      hidden     |        最慢/最慢       |         ✅        |   original  | 没有引用，适用于仅需要错误报告的场景 |

|      品质     | 说明                            |
| :---------: | ----------------------------- |
|   original  | 能看到源代码                        |
|  generated  | 能看到生成的代码，但是在开发者工具中每个模块都是独立的文件 |
| transformed | 只能定位到行，列信息会被忽略                |

#### development 下的选择

*   **eval-source-map**

    第一次构建较慢，但重构时速度还行。行数可以正确的映射
*   **eval-cheap-source-map**

    只有行，没有列的信息，可以提高性能
*   **eval-cheap-module-source-map**

    比上面的效果好一些，是一个折中

> **eval-cheap-module-source-map** 是多数情况下的一个最佳选择

#### production 下的选择

*   **不使用 devtool**

    不生成 source map，是一个不错的选择
*   **hidden-source-map**

    hidden-source-map - 与\`source-map 相同，但不会为 bundle 添加引用注释。如果你只想 source map 映射那些源自错误报告的错误堆栈跟踪信息，但不想为浏览器开发工具暴露你的 source map，这个选项会很有用
*   **nosources-source-map**

    创建的 source map 不包含 `sourcesContent(源代码内容)`。它可以用来映射客户端上的堆栈跟踪，而无须暴露所有的源代码。你可以将 source map 文件部署到 web 服务器。（仍然会暴露反编译后的文件名和结构，但是不暴露原始代码）
