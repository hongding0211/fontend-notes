# Reactivity

## 如何工作

JavaScript 不能跟踪值类型数据，但是可以追踪对象属性。

在 Vue3 中，`getter / setter` 用于 `ref` ，`Proxies` 用于 `reactive`。
