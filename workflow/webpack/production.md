# 生产环境配置

## 独立的配置文件

development 和 production 环境应使用独立的配置文件，但是两个配置文件中存在着相同的部分。因此可以借助 `webpack-merge` 工具，来引用一个 common 的配置，以此避免编写重复的配置项。

```bash
yarn add -D webpack-merge
```

**webpack.common.js**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const path = require('path')

module.exports = {
  entry: {
    app: './src/index.js'
  },
  output: {
      filename: '[name].[hash:10].js',
      path: path.resolve(__dirname, 'dist'),
      clean: true
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack'
    })
  ]
}
```

**webpack.dev.js**

```js
const { merge } = require('webpack-merge')
const common = require('./webpack.common')

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval-source-map',
  devServer: {
    static: './dist'
  }
})
```

**webpack.prod.js**

```js
const { merge } = require('webpack-merge')
const common = require('./webpack.common')

module.exports = merge(common, {
  mode: 'production'
})
```

在 **package.json** 添加 scripts

```json
"scripts": {
  "build": "webpack --config webpack.prod.js",
  "start": "webpack serve --open --config webpack.dev.js"
 },
```
