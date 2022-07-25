# Promise

## 时序调用链

多个 Promise 之间可以组成时序调用链，如有三个回调函数 `[func1, func2, func3]` ，它们依次在上一个 Promise resolve 后执行：

```javascript
p.then(func1).then(func2).then(func3)
```

上面这样的形式还可以使用 `reduce` 实现：

```javascript
function func1() { console.log('func1') }

function func2() { console.log('func2') }

function func3() { console.log('func3') }

[func1, func2, func3].reduce((p, f) => p.then(f), Promise.resolve('init'))

// output:
// func1
// func2
// func3
```

