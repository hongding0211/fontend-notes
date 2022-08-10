# bind()

## API

`bind()` **创建一个新的函数**，新函数中的 `this` 绑定为 `bind()` 的第一个参数，其余参数作为新函数的参数；供之后调用使用。

```javascript
function foo(b) {
    console.log(this.a + b)
}

var obj = {
    a: 0
}

// 创建了一个新函数
var bar = foo.bind(obj, 1)

// 再进行调用
bar()   // 1
```

如果使用 `new` 构造调用 `bar` ，那么 `bind` 绑定是无效的：

```javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 0
}

var a = 'oops, global'

// 无效绑定
var bar = foo.bind(obj)

// 使用 new 调用
new bar()   // undefined, 注意不是绑定到了全局，而是绑定到一个新的空对象上了
```

> 虽然使用 `new` 调用会忽略 `this` ，但是后续传入的参数，在 `new` 调用时会被正确传入

## 实现细节

`bind()` 函数会创建一个新的绑定函数，它包装了原始函数；绑定函数内部有如下几个属性：

* \[\[BoundTargetFunction]] - 被包装的原始函数
* \[\[BoundThis]] - `this` 绑定的对象
* \[\[BoundArgumengts]] - 传入的剩余参数

调用绑定函数时，会调用 \[\[BoundTargetFunction]] 上的内部方法 `[[call]]` ，就像：`Call(boundThis, args)`

## 应用

### 柯里化

利用 `bind()` 可以创建一些预设好参数的函数：

```javascript
function add(a, b) {
    return a + b
}

// 传给 bind() 的第二个开始的参数会被插入到目标函数的参数的起始位置
const add10 = add.bind(null, 10)

// 传给绑定函数的参数会继续附加到后面
add10(5)   // 15
```

> 正如上文所提，这里传给 `bind()` 的 thisArg 最好是一个 DMZ 对象，这样更加安全

### 配合 setTimeout

前文提到过，在没有 ES6 的箭头函数时，回调函数中的 `this` 容易泄漏到全局对象上。我们可以用 `bind()` 包装一个回调函数，从而指定这个回调函数中的 `this`：

```javascript
var obj = { a: 0 }

function foo() {
    function callbackFn() {
        console.log(this.a)
    }

    // 将 callbackFn 中的 this 显式绑定到 obj 上
    setTimeout(callbackFn.bind(obj), 0)
}

foo()   // 0
```

## 手写实现 bind()

根据 `bind()` 实现的功能：

```javascript
Function.prototype.myBind = function(thisArg, ...args) {
    // 根据隐式绑定规则，this 就是被包装函数本身
    const fn = this
    // 记得 return 的 function 也可以接受剩余的参数，参见前面柯里化例子
    return function(...rest) {
        // 调用的时候，先传入 this，然后再将剩余的参数拼接
        fn.apply(thisArg, [...args, ...rest])
    }
}
```

按照前面的例子，可以测试下：

```javascript
function foo(b) {
    console.log(this.a + b)
}

const obj = { a: 1 }

// 这里 myBind() 是由 foo 调用的（确切来说是 foo 委托 Function 的原型）
// 所以，这里应用的是 “隐式绑定” 规则，this 指向 foo 函数本身
const bar = foo.myBind(obj, 2)

bar()   // 3
```

```javascript
function add(a, b) {
    console.log(a + b)
}

const add10 = add.myBind(null, 10)

add10(5)    // 15
```
