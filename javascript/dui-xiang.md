# 对象

## 属性描述符

通过 `Object.getOwnPropertyDescriptor()` 可以查看对象的某一属性的**属性描述符**

```javascript
var obj = {
    a: 0
}

Object.getOwnPropertyDescriptor(obj, 'a')
```

可以查看到 `obj.a` 的属性描述符：

```json
{
    value: 0,
    writable: true,
    enumerable: true,
    configurable: true
}
```

相应的，我们也可以使用 `Object.defineProperty()` 来设置对象的属性

### writable

决定是否可以修改属性的值。如果设置为 `false` ，在非严格模式下修改会被静默处理；而在严格模式下会抛出 `TypeError` 异常

```javascript
var obj = {}

Object.defineProperty(obj, 'a', {
    value: 0,
    writable: false
})

obj.a = 1 // 非严格模式下被静默处理

obj.a     // 0
```

### enumerable

决定属性能否出现在对象的属性枚举中，如 `for ... in` 循环。如果设置为 `false` 你仍然可以访问到该属性，只是它不会出现在枚举中。

### configurable

如果为 `true` ，那么就可以使用 `Object.defineProperty()` 来修改（重新覆盖）对象的属性，不管是否处于严格模式下。

> 也就是说，先用 `Object.defineProperty()` 将某一个属性的 `configurable` 设置为 `false` 后，再次对该属性使用 `Object.defineProperty()` 来尝试修改则会抛出 `TypeError` 异常。
>
> **也就是说，这是一次单向操作**

如果一个属性的 `configurable` 为 `false` ，那么你也不能用 `delete` 来删除该属性（不过修改他的值还是可以的）：

```javascript
var obj = {}

Object.defineProperty(obj, 'a', {
    value: 0,
    configurable: false
})

// 非严格模式下静默，严格模式下抛出 TypeError 异常
delete obj.a

obj.a   // 0
```

### 不可变性质

如果想要对象中的某个属性或者对象自己是不可变的，可以通过以下方式实现：

1. 将 `writable` 和 `configuable` 设为 `false`&#x20;
2. 使用 `Object.preventExtenstions()` 可以禁止一个对象扩展（不能添加新属性）
3. 使用 `Object.seal()` 可以使对象不可扩展，并且所有属性的 `configuable` 都设为 `false`（但是仍然可以修改值）
4. 使用 `Object.freeze()` 相当于在 `Object.seal()` 的基础上再把所有的属性的 `writable` 也设为 `false` ，这样对象不可扩展、不可配置、不可写。

> 所有的不可变性修改，都是浅不可变性；对于引用类型，并不能真正的冻结住他们。

## \[\[Gget]] & \[\[Put]]

对一个对象的属性访问时，其实会在该对象上实现了 `[[Get]]` 操作；相应的，设置一个对象时，也会对该对象实现 `[[Put]]` 操作

需要注意的是，和 RHS 不同的是，对一个对象中不存在的属性访问时，并不会抛出 `ReferrenceError` ，而是返回 `undefined`

```javascript
var obj = {}

a        // ReferenceError，RHS 错误

obj.a    // undefined
```

### getter && setter

可以使用字面量或者 `Object.defineProperty()` 来设置属性的 `getter`&#x20;

```javascript
var obj = {
    get a() {
        return 0
    }
}

Object.defineProperty(obj, 'b', {
    get: function() {
        return 1
    }
})

obj.a	// 0
obj.b	// 1
```

如果一个属性只有 `getter`，那么尝试修改该属性时会被忽略（严格模式下抛出 `TypeError` ）

```javascript
var obj = {
    get a() {
        return 0
    }
}

// 属性设置会被忽略
obj.a = 1

obj.a   // 0
```

进一步说，即使给属性 `a` 设置了正确的 `setter` 再次读取他的时候，仍然返回 0。

这是因为 `getter` 只会返回 0，因此正确的做法如下：

```javascript
var obj = {
    get a() {
        return this._a_
    },
    set a(val) {
        this._a_ = val
    }
}

obj.a = 0

obj.a   // 0
```

仔细思考上述代码 `this` 的指向，这里应用的是隐式绑定规则

`obj.a` 其实调用的是函数。在这个函数内部，`this` 绑定在对象 `obj` 上

```javascript
// 查看 obj 对象内，this 指向的就是 obj
obj    // { _a_: 0 }
```

### 存在性

前面提到过，不同于 RHS 错误。对于不存在的属性进行访问会返回 `undefined` ，但是如果该属性虽然存在，但是他的值是 `undefined` 也会返回相同的结果。

因此可以使用 `in` 关键字或者 `hasOwnProperty()` 来判断一个对象是否存在某一属性，然而他们的区别是：

* `in` 会向上寻找原型链
* `hasOwnProperty()` 只会检查属性是否在该对象上

`hasOwnProperty()` 是 `Object.prototype` 上的一个函数。通常情况下，对任意对象 `obj` ，都可以直接调用 `obj.hasOwnProperty()`&#x20;

然而，如果一个对象是通过 `Object.create(null)` 创建的，那这个对象就不能通过委托在原型链上寻找到 `hasOwnProperty()` 这一函数。一种解决方法是：

```javascript
var obj = Object.create(null)

obj.a = 0

Object.prototype.hasOwnProperty.call(obj, 'a') // true
```

> 注意 `in` 操作符并不会检查容器内是否存在某一个值；他只是检测该属性是否存在于对象或原型链上
>
> 比如 `3 in [1, 2, 3]` 并不会返回 `true` ，因为该数组只有 0, 1, 2 三个属性

## 遍历

### for ... in

使用 `for ... in` 可以遍历 `enumerable` 为 `true` 的属性：

```javascript
var obj = {
    a: 0,
    b: 1
}

// a 0
// b 1
for (var k in obj) {
    console.log(k, obj[k])
}
```
