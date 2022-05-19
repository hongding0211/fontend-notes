# HTML 相关概念

### script 标签

默认情况下立即执行，一般放在 `<body>` 最后

```html
<script src="script.js"></script>
```

如果加入 `async`，加载和渲染后续文档元素的过程将和 script.js 的\*\*加载与执行\*\*并行进行（异步）。但是多个js文件的加载顺序不会按照书写顺序进行

```html
<script async src="script.js"></script>
```

如果加入 `defer` ，加载后续文档元素的过程将和 script.js 的\*\*加载\*\*并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成,并且多个defer会按照顺序进行加载

```html
<script defer src="script.js"></script>
```

#### 解析顺序

如果放在 head 标签内部，那么会阻塞 html 的解析，必须要等到 js 代码全部下载执行后才能继续。所以一般放在 body 标签最后

#### DOM 操作加载 script 标签

// todo

> [(54条消息) javascript异步加载的三种方式以及如何动态创建script标签\_Choo01的博客-CSDN博客\_script标签异步加载](https://blog.csdn.net/Choo01/article/details/107312123?spm=1001.2101.3001.6661.1\&utm\_medium=distribute.pc\_relevant\_t0.none-task-blog-2\~default\~CTRLIST\~Rate-1.pc\_relevant\_aa\&depth\_1-utm\_source=distribute.pc\_relevant\_t0.none-task-blog-2\~default\~CTRLIST\~Rate-1.pc\_relevant\_aa\&utm\_relevant\_index=1)

### link 和 @import

link

```html
<link href="index.css" rel="stylesheet">
```

import

```html
<style type="text/css">
	@import url(index.css);
</style>
```

#### 区别

1. link 可以引入别的(图片等），import 只能引样式
2. link 与页面同时载入，import 需要页面完全载入后再加载
3. link 兼容性更高
4. link 支持使用 Javascript 控制 DOM 去改变样式；而 @import 不支持

### href 和 src

> link 使用 href；script、img使用 src

**src 用于替换当前元素，而href用于在当前文档和引用资源中建立联系**

* src
  * 指向的内容将会\*\*嵌入到文档中当前标签所在位置\*\*；在请求src资源时会将其指向的资源下载并应用到文档内，例如js脚本，img图片和frame等元素
* href
  * 指向网络资源所在位置，建立和当前元素（锚点）或当前文档（链接）之间的链接
  * 在文档中添加link标签，浏览器会识别该文档为css文件，就会并行下载资源并且**不会**停止对当前文档的处理。这也是为什么建议使用link方式来加载css，而不是使用@import方式

### iframe

将一个 HTML 页面嵌入到另一个页面中。

每个 iframe 对会有自己的 session history 和 DOM 树。

每个 都会增加内存资源，因为它们拥有完整的文档环境。
