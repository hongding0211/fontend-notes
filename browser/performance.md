# 性能优化

* 加载优化
* 执行优化
* 渲染优化
* 样式优化
* 脚本优化

### 加载优化

1. 减少 HTTP 请求：合并 css，js文件；使用 css 精灵图
2. 缓存资源
3. 压缩代码，启用 gzip
4. 压缩图像
5. 减少 cookie，静态资源域名不试用 cookie
6. 内联样式会阻塞加载，样式在 head 中用 link 引入，script 放在 body最后
7. 首屏加载，优化快速显示
8. 按需加载：懒加载，滚屏加载
9. 预加载
10. 异步加载

### 执行优化

1. css 写头部，js 放尾部并使用异步
2. 避免 img iframe 的 src 为空，因为空的 src 会导致重载

### 渲染优化

1. 设置 viewpoint
2. 减少 DOM 节点
3. 优化动画：元素多时使用 canvas
4. 优化高频事件：scroll、touch move：节流，防抖

### 样式优化

1. 避免在 HTML 中写 style
2. 避免 CSS 表达式
3. 不滥用 float
4. 不滥用 Web 字体，字体需要下载，解析

### 脚本优化

1. 减少 DOM 操作
2. 尽量修改 class 而不是 style

### 尾调用优化

如果一层层函数调用下去，会占用调用栈，如果内层函数不需要外层函数的数据，那么可以不 push 一个新的调用栈

```js
function foo() {
  function bar() {
    ...
  }
  return bar()
}
```

### 懒加载

#### 图片
