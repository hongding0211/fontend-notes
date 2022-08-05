# this

## 什么是 this

### 为什么要用 this

this 可以当做一个**隐式传递的上下文**，避免每次都需要显示地传递。

```javascript
function foo(context) {
    console.log(context.a)
}

var obj1 = { a: 1 }
var obj2 = { a: 2 }

foo(obj1)   // 1
foo(obj2)   // 2
```

上面的代码中，每次都显式地传入了一个 `context` 上下文。

但我们可以利用 `this` 来更加优雅的实现上列代码：

```javascript
function foo() {
    console.log(this.a)
}

var obj1 = { a: 1 }
var obj2 = { a: 2 }

foo.call(obj1)  // 1
foo.call(obj2)  // 2
```

### this 的误区

下面是两种常见的误区：

* `this` 指向函数本身
* `this` 指向函数的作用域

#### this 不指向函数自身的对象

虽然在 JavaScript 中，函数也是对象。但 `this` 并不是指向函数自身的对象。

例如，你想向函数对象内写入一个属性，来记录函数被调用的次数：

```javascript
function foo() {
    this.count++
}

foo.count = 0

for (let i = 0; i < 10; i++) {
    foo()
}

console.log(foo.count)  // 0
```

在函数内部使用 `this.count` 并不能读取到函数对象内的 `count` 属性。

如果要实现类似效果，将 `this.count` 改成 `foo.count`

```javascript
function foo() {
    foo.count++
}

foo.count = 0

for (let i = 0; i < 10; i++) {
    foo()
}

console.log(foo.count)  // 10
```

#### this **不指向函数所在的词法作用域**

不要把词法作用域和函数自身对象搞混在一起。虽然他们很相似，但作用域“对象”无法通过 JavaScript 的代码访问，他只能由引擎自身访问。

### this 是什么

`this` 是在**运行时绑定的**，而不是在编写的时候绑定的。

`this` 的绑定和函数声明的位置没有关系，**只取决于函数调用的方式（在哪里被调用）**。

> 当函数在调用时，会创建一个活动记录。包含调用栈，函数调用的方式，传入的参数等。`this` 就是其中的一个属性。

## 绑定规则

### 默认绑定

> 默认绑定是在无法应用其他规则时的 fallback option

当应用了默认绑定规则，`this` **指向全局对象**。

```javascript
function foo() {
    console.log(this.a)
}

var a = 0

foo()   // 0
```

上面的代码函数在调用时，应用了默认绑定规则，因此 `this` 指向全局对象

如果使用了严格模式，`this` 会被绑定为 `undefined`

```javascript
function foo() {
    "use strict"
    console.log(this.a)
}

var a = 0

foo()   // TypeError
```

> 只有在函数内部应用严格模式才会限制默认绑定规则
>
> 相反，在严格模式下调用函数并不会影响默认绑定规则

```javascript
function foo() {
    console.log(this.a)
};

var a = 0;

(function() {
    "use strict"
    foo();   // 0 在严格模式下调用函数不影响
})();
```

### 隐式绑定

当**函数的引用有上下文对象时**，函数中的 `this` 会绑定到这个上下文对象上：

```javascript
function foo() {
    console.log(this.a)
};

var obj = {
    foo: foo,
    a: 0
}

obj.foo()   // 0
```

`foo` 被对象 `obj` 引用，调用时 `this` 绑定在 `obj` 上

但是，如果存在引用链，那么以**最后一层调用**为主：

```javascript
function foo() {
    console.log(this.a)
};

var obj1 = {
    foo: foo,
    a: 0
}

var obj2 = {
    a: 1,
    obj1: obj1
}

obj2.obj1.foo() // 0
```

#### 绑定丢失

在某些情况下，隐式绑定可能会发生绑定丢失。取而代之应用了默认绑定规则

```javascript
function foo() {
    console.log(this.a)
};

var obj = {
    foo: foo,
    a: 0
}

// bar 只是对 foo 的一个引用
const bar = obj.foo

bar()   // undefined
```

除此之外，在回调函数中也很容易丢失绑定：

```javascript
function foo() {
    console.log(this.a)
};

function doFoo(fn) {
    // 这里的 fn 其实是对 foo 的引用
    fn()
}

var obj = {
    foo: foo,
    a: 0
}

// 虽然传入给回调函数看起来有隐式绑定，但其实实质上传的只是 foo 的引用
doFoo(obj.foo)  // undefined
```

即便是传入给内置的回调函数，如 `setTimeout` 也会发生绑定丢失：

```javascript
function foo() {
    console.log(this.a)
};

var obj = {
    foo: foo,
    a: 0
}

setTimeout(obj.foo, 0) // undefined
```

### 显式绑定

我们可以在一个对象内引用一个函数，并用这个对象的属性来调用该函数，从而使得 `this` 隐式绑定到该对象上。但是我们也可以使用显示绑定，将 `this` 绑定到某一特定对象上。

```javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 0
}

foo.call(obj)   // 0
```

如果传入的是一个值类型，那么 `call()` 方法会把它包装成一个对象。如：new String(), new Boolean()...&#x20;

> `call` 接受一个个单独的参数，`apply` 接受一个参数数组

#### 硬绑定

```javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 0
}

var bar = function() {
    foo.call(obj)
}

// 无论如何调用 bar，this 始终绑定为 obj
bar()               // 0
setTimeout(bar, 0)  // 0
bar.call({})        // 0
```

每次调用 `bar` 的时候，在他的内部都会调用一次显示绑定，因此 `obj` 都会被绑定在 `this` 上面。

### new 绑定
