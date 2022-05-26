# 计算属性 & 侦听器

## 计算属性

计算属性根据 data 计算出新的内容：

```javascript
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```javascript
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

### 缓存

计算属性和调用一个函数最大区别是拥有缓存。虽然上面的例子也可以这样实现：

```markup
<p>Reversed message: "{{ reversedMessage() }}"</p>
```

```javascript
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

但是，计算属性**只会在相关响应式依赖发生改变时才重新求值**。

而通过一个函数来求值时，每次重渲染时都会调用一次。

