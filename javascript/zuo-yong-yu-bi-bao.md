# 作用域

## 作用域

负责收集维护所有变量的一系列查询，并实施一套严格的规则，确保代码对这些变量的访问权限

对于 `var a = 0` 这一操作，可以分解成如下：

* 如果变量 a 在作用域中不存在，**编译器**就在作用域中声明一个（若存在就忽略不会再声明一个，**注意只是不再在作用域中声明一个，不要和赋值搞混**）
* **在运行时**引擎在作用域查找该变量，并且为它赋值

> 也就是说，var a = 0 是拆分成编译和运行时两个部分的
>
> 编译是在运行时之前先运行的（虽然时间很短）

## LHS RHS 查询

### LHS

可以理解为找到变量容器的本身，**从而可以对其赋值**

### RHS

可以理解为 Retrieve his source value （取到他的源值），**来获取变量的值**

```javascript
function foo(n) {	// 这里其实有个隐式赋值 n = 2，对 n 做了一次 LHS
    console.log(n)	// 对 n 做了一次 RHS，并且对 console 也做了一次 RHS
}

foo(2)	// 对 foo 做了一次 RHS，来找到函数
```

### 查询错误

对于还未声明的变量，进行 LHS 和 RHS 查询抛出的异常不同：

*   RHS 在查询失败时，会抛出 ReferenceError

    > RHS 查询成功后，但如果对变量的值进行不合理的操作的时候
    >
    > 如访问 null 的属性，会抛出 TypeError

    ```javascript
    var a = b	// RHS 查询失败，ReferenceError: b is not defined
    ```
*   LHS 在查询失败时，**在非严格模式下**，会创建一个具有该名称的变量

    **具体来说，如果在顶层都无法找到变量，就会在全局作用域内创建一个变量**

    > 在严格模式下，LHS 查询失败时也会抛出 ReferenceError

    ```javascript
    "use strict"

    a = 0	// 在严格模式下 LHS 抛出异常：Uncaught ReferenceError ReferenceError: a is not defined
    ```

> ReferenceError 表示作用域判别失败；TypeError 表示作用域判别成功，但是对结果的操作是不合理的。

## 词法作用域

**JavaScript 和大部分语言一样，使用的都是词法作用域。**尽管 this 可以模拟类似动态作用域的行为（后文展开）。

词法作用域即：定义在词法阶段的作用域。换言之，**他是由你在写代码时将变量放置的位置所决定的**。（大部分如此，例外在后面展开）

### 查找

作用域查找会在**第一个匹配**时停止。

此外，如果访问 `foo.bar.baz` 只会对第一个标识符 `foo` 进行查找，查询成功后，`bar`, `baz` 由对象属性访问规则接管

#### 遮蔽

如果多个层级存在同名变量，内部标识符会遮蔽外部标识符。例如：

```javascript
var a = 0

function foo() {
    var a = 1
    console.log(a)        // 1
    console.log(window.a) // 0
}

foo()
```

可以使用 `window` 来“越级”访问变量 a。避免了 a 被内部作用域遮蔽。

### 改变词法作用域

可以通过一些技巧来在运行时改变词法作用域

#### eval

eval 执行了传入的 str，覆盖了全局变量 a：

```javascript
var a = 0

function foo(str) {
    eval(str)
    console.log(a)  // 1
}

foo('var a = 1')
```

在严格模式下，eval 会有自己的作用域：

```javascript
var a = 0

function foo(str) {
    "use strict"
    eval(str)
    console.log(a)  // 0
}

foo('var a = 1')    
```

#### with

在 with 中可以产生一个 LHS，就可以在全局变量中创建一个变量：

```javascript
function foo(obj) {
    with (obj) {
        a = 0
        b = 1 //  LHS 自动在全局创建了一个变量 b，并为他赋值 1
    }
}

var obj = {
    a: undefined
}

foo(obj)

console.log(obj.a) // 0
console.log(b)     // 1
```

## 函数作用域

### 函数声明 vs 函数表达式

一种最简单的区分函数声明和函数表达式的方式，就是看 `function` 关键字是否出现在声明中的第一个词。如果是，那就是函数声明。

他们之间的区别就是：**他们的名称标识符会被绑定在哪里**：

```javascript
// 绑定在全局作用域
function foo() {}     // 函数声明

// 绑定在自身的函数作用域中
(function foo() {})   // 函数表达式
```

### 具名函数 vs 匿名函数

> 函数声明不允许使用匿名函数

相比具名函数，匿名函数的缺陷是：

* 在调用栈上不能显示名称，不利于调试
* 不能通过函数名递归地调用自己
* 可读性较差

下面的这种写法也是允许的，因此你也可以使用具名函数：

```javascript
setTimeout(function foo() {
    console.log('bar')
}, 0)
```

### IIFE 立即执行函数

> IIFE 也可以接受具名函数

```javascript
var a = 0;

(function() {
    var a = 1;
    console.log(a);  // 1
})();

console.log(a);      // 0
```

当然，还可以传入参数：

```javascript
var a = 0;

(function(global) {
    console.log(global.a)  // 0
})(window)
```

#### 避免 undefined 被覆盖问题

有时候 undefined 关键词可能会被污染，因此可以使用 IIFE 来解决。IIFE 接受一个 undefined 参数，但在对应的位置不要传入他：

```javascript
// JavaScript 允许下面的写法
var undefined = true;

(function(undefined) {
    console.log(undefined);    // undefined
})();                          // 不要传入参数

console.log(undefined);        // true
```

## 块作用域

JavaScript 虽然会创建函数作用域，但有些情况下并不会创建块级作用域 `{ }`&#x20;

```javascript
if (false) {
    var foo = 'bar'  // if 语句中的 foo 其实是在全局作用域中
}

console.log(foo)     // undefined
```

或者一个更常见的例子，在 `for` 语句中：

```javascript
// 变量 i 其实是创建在全局作用域中的
for (var i = 0; i < 10; i++) {}

console.log(i)      // 10
```

### 创建块级作用域的方式

在上面的例子中，并不会创建块级作用域。但是在另一些情况下，可以创建：

* with
* catch 块

#### catch 块

`catch` 语句可以创建一个块级作用域，因此可以利用这个技巧来创建一个块级作用域：

```javascript
try {
    throw 0
} catch (foo) {
    console.log(foo)  // 0
}

console.log(foo)      // ReferrenceError
```

## let & const

在 ES6 中，使用 `let` 关键词创建的变量会**将该变量劫持在所在的块作用域中**：

```javascript
if (false) {
    let foo = 'bar'  // foo 在块级作用域中
}

console.log(foo)     // ReferenceError
```

### 显式的创建块级作用域

上面的例子中，其实是隐式的创建了一个块级作用域。

你可以用 `{ }` 来显示的创建一个块级作用域。那么 `let` 就会把变量劫持在内部：

```javascript
{
    let foo = 'bar'
}
```

### for 循环

使用 `let` 来代替 `var` 创建循环变量即可解决作用域泄露问题：

```javascript
for (let i = 0; i < 10; i++) {}
```

`let` 关键字不仅将变量 `i` **绑定在了循环块内部**，并且在**每次迭代时进行重新绑定**

> 每次都是用上一次循环结束时的值进行重新赋值

```javascript
{
    let i
    for (i = 0; i < 5; i++) {
        let j = i
        console.log(j)  // 0 1 2 3 4
    }
}
```

## 变量提升

* 定义发生在编译阶段
* 赋值发生在运行时

**每个作用域都会做变量提升，但不会提升到外部**，比如函数作用域的变量只会提升到自己作用域的前部

函数声明会被提升，但是函数表达式并不会。参考下面例子：

```javascript
foo()   // TypeError

var foo = function() {
    console.log('bar')
}
```

上面的代码变量 `foo` 被提升到了头部，但是声明并没有。因此在第一行对一个 undefined 进行函数调用时，会抛出 TypeError。

而一个函数声明是可以被提升的：

```javascript
foo()   // bar

function foo() {
    console.log('bar')
}
```

### 函数优先

变量和函数都会被提升，**但函数的优先级更高**

```javascript
foo()   // 0

var foo 

// 函数声明会被提升到最前
function foo() {
    console.log(0)
}

foo = function() {
    console.log(1)
}
```

上面的代码其实等价于：

```javascript
function foo() {
    console.log(0)
}

var foo // 重复的声明会被忽略

foo()

foo = function() {
    console.log(1)
}
```
