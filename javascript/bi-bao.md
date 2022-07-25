# 闭包

## 什么是闭包

当一个函数可以记住所在的词法作用域，就产生了闭包。即使该函数在词法作用域以外被调用。&#x20;

```javascript
function foo() {
    var a = 0

    function bar() {
        console.log(a)
    }

    return bar()
}

var baz = foo

// bar() 在词法作用域之外被调用，但是可以访问到他所在的词法作用域内的变量
baz()   // 0
```

上述例子中，`bar()` **在其定义的词法作用域以外被执行**（虽然通过 baz 调用，但实际上就是 bar）。但是他可以访问到自己词法作用域内的变量 `a`

> bar 在全局变量调用的时候，他其实是**从自己的词法作用域内进行查找**，顺着 foo() 函数作用域向上
>
> 而不是在全局作用域内查找，若在全局作用域中查找，应该会抛出一个 ReferenceError （ RHS 失败）

函数 `foo()` 在执行后内部的作用域并没有被销毁，而是被 `bar()` 所持有。

`bar()` 依旧持有对该作用域的引用，**这个引用就叫闭包**。

也就是说：**一个函数在定义的词法作用域以外被调用，闭包使得这个函数在调用时仍然可以访问定义时的词法作用域。**

## 闭包解决循环问题

下列代码并不会按照顺序依次输出 0, 1, 2 ...，而是输出 5 次 5

```javascript
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i)    // 5 5 5 5 5
    }, 0)
}
```

造成上述问题的本质是：我们会**误以为 setTimeout 在每次循环时都会捕获时下变量 `i` 的值**，但实际上只存在一个在全局作用域内的 `i` 。

一种解决方法，就是利用闭包特性，**给每次 setTimeout 创建一个闭包，让回调函数可以捕获所对应的词法作用域**：

> 在不利用 `let` 关键词的情况下，可以利用 IIFE 来创建函数作用域，再使用闭包来捕获他

```javascript
for (var i = 0; i < 5; i++) {
    // 利用 IIFE 创建一个函数作用域
    (function() {
        // 用一个变量把时下的 i 保存在函数作用域内
        var j = i
        setTimeout(function() {
            console.log(j)
        }, 0)
    })()
}
```

### 利用 let 创建块级作用域

上述的方法是在不使用 `let` 关键词的情况下，通过创建一个函数作用域来将每一次的变量 `i` 捕获下来。

有了 `let` 关键词，可以免去使用 IIFE 创建函数作用域的过程：

```javascript
for (var i = 0; i < 5; i++) {
    // let 创建了一个块级作用域，用于回调函数捕获
    let j = i
    setTimeout(function() {
        console.log(j)
    }, 0)
}
```

`let` 关键词将变量 `j` 捕获在了一个块级作用域内部。

但是，上述的写法还可以继续简化成：

```javascript
for (let i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i)
    }, 0)
}
```

这种写法就能得到正确的输出结果，他的原理是：

`for` 循环头部的 `let` 声明有一个特殊行为：变量在循环过程中不止会被声明一次，而是**每次迭代都会声明一次**。每一次声明，都是用上次循环结束的值进行赋值。

## 用闭包封装模块

下面就是一种模块封装的基本形式：

```javascript
function FooModule() {
    var name = 'bar'

    function sayHello() {
        console.log(`Hello, ${name}`)
    }

    return {
        sayHello
    }
}

var foo = FooModule()

foo.sayHello()  // Hello, bar
```

这种模块封装的方式就利用到了闭包的特性。他**具备以下两个必要条件**：

* **必须有一个外部封装函数，并且需要被调用一次**（每调用一次就创建一个新的模块实例）
* 返回的内容中**必须持有一个内部函数**，这样才能保证对封装函数内部的作用域引用（形成闭包）

模块往往只需要被调用一次，因此可以采用 IIFE 的写法：

```javascript
var fooModule = (function() {
    var name = 'bar'

    function sayHello() {
        console.log(`Hello, ${name}`)
    }

    return {
        sayHello
    }
})()

fooModule.sayHello()  // Hello, bar
```

### 模块机制

现代的模块机制使用类似于下面的结构：

```javascript
var MyModules = (function () {
    var modules = {}

    function define(name, deps, impl) {
        for (var i = 0; i < deps.length; i++) {
            deps[i] = modules[deps[i]]
        }
        modules[name] = impl.apply(impl, deps)
    }

    function get(name) {
        return modules[name]
    }

    return {
        define,
        get
    }
})()

MyModules.define('foo', [], function() {
    function hello(name) {
        console.log(`Hello, ${name}`)
    }

    return {
        hello
    }
})

MyModules.define('bar', ['foo'], function(foo) {
    function helloFromBar() {
        foo.hello('bar')
    }

    return {
        helloFromBar
    }
})

var bar = MyModules.get('bar')

bar.helloFromBar()  // Hello, bar
```

这里最核心的就是：`modules[name] = impl.apply(impl, deps)`

这边 apply 传入的 deps 就是可以被内部函数（ return 出来的 helloFromBar）利用闭包来捕获。

也就是说，对于模块 bar，传入了一个依赖模块 foo。那么模块 foo 就存在 bar 模块的此法作用域中。当你在外面调用 bar 模块暴露的内部函数，内部函数仍然可以访问到 foo。即：**bar 成功依赖于 foo**，foo 存在于 bar 的闭包里。
