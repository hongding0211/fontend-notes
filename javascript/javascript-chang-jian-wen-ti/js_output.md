# 看代码输出

### 连续赋值问题

```js
let a = {n : 1};
let b = a;
a.x = a = {n: 2};       
console.log(a.x)  // undefined
console.log(b.x)  // {n:2}
console.log(a === b.x)  // true
```

第三行，连续赋值时，访问 `.` 操作符的优先级大于 `=` 赋值操作符。所以 `a.x` 会先在 a 引用的对象上开辟一个新属性。

### toString()

```js
12.toString()	// error 12. 被当做 12.0 的省略，所以这个 . 表示小数点
// 可行方法
12 .toString()
12..toString()
```

### finally return

```js
function foo() {
    try {
        console.log('bar')
        return 0
    } catch (err) {
        console.log(err)
    } finally {
        console.log('finally')
    }
}

// bar
// finally
// 0
console.log(foo())
```

如果 finally return 了值，那就只会 return finally 里面的

```js
function foo() {
    try {
        console.log('bar')
        return 0
    } catch (err) {
        console.log(err)
    } finally {
        return 'finally'
    }
}

// bar
// finally
console.log(foo())
```
