# Webpack 概念

### entry

`entry` 是 webpack 构建的起点，默认是 `./src/index.js`

### output

`output` 指定输出 bundle 的位置，以及如何命名这些文件。主文件默认输值是 `./dist/main.js` ，其他生成文件默认放在 `./dist` 文件夹中

### loader

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。**loader** 让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块，以供应用程序使用，以及被添加到依赖图中

### plugin

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量

### mode

通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`
