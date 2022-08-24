# Object.create()

## Polyfill / 手写实现

```javascript
Object.create = function(o) {
    function fn() {}
    fn.prototype = o
    // 返回一个新对象，这个对象的 [[prototype]] 链接到了 o
    return new fn()
}
```
