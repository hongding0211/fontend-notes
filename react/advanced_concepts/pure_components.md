# Pure 组件

### 什么是纯函数

* 不改变函数作用域以外的内容
* 同样的输入永远返回同样的结果

### Pure Components

对于 Pure Components 来说，相同的输入（props, state, context）永远得到相同的 JSX 内容，同时也不要在操作组件外部的”全局变量“。

### 为什么要保持 Purity ？

* 纯组件可以让 React 优化渲染，如果输入不变时即可跳过
* 在渲染途中数据发生了变化可以重启渲染
* 可以在服务端完成渲染
* 更多新的 React 特性都会利用到 Purity

### Side Effects

Pure 意味着相同的输入应该得到相同的结果。

但是一些额外的”副作用“也是必要的，一般分为以下两种 Side Effects：

* 事件回调
* useEffect

这些副作用不会在 render 阶段发生。**而是发生在 render 之后**。因此，这些副作用可以不需要保证 Purity。

在处理 Side Effect 时，优先使用回调的方式，继而再考虑使用 useEffect()。

### StrictMode 下的 Purity 检查

在开发环境下，默认会启动 StrictMode。

在 StrictMode 下，React 都会默认调用两次渲染函数。通过两次调用，来找出哪些组件违反了 Purity。
