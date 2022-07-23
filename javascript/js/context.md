# Context 执行上下文

### 概述

执行环境分为三种

1. 全局，即 window / global
2. 函数执行环境
3. Eval 函数

每个执行环境都有一个与之关联的`变量对象（variable object）`，环境中定义的所有变量和函数都保存在这个对象中。

### 执行环境

用一个栈里维护执行环境，默认栈底就是 global

每当一个函数执行时，就压入这个函数的执行环境，函数执行完毕后，再弹出，然后把控制权交给之前的环境。

```js
// 示例代码
var name = "Jack";
function func() {
  console.log(name); // 访问全局作用域
}

function func2() {
  var name = "啊哈哈";
  console.log(name); // 访问函数内部作用域
}

func(); // Jack
func2(); // 啊哈哈
```

### 词法作用域 / 静态作用域

静态作用域：**函数被定义的时候，它的作用域就已经确定了**，和拿到哪里执行没有关系。

JavaScript 使用的就是静态作用域，参考下面例子

```js
var value = 1;

function foo() {
  console.log(value);
}

function bar() {
  var value = 2;
  foo();
}

bar();	// 结果是: 1
```

上述代码三个作用域：全局、foo()、bar（）

因为 foo() 中没有 value，那他就要向上层寻找

### 块级作用域

JS 默认只用两种作用域：全局作用域 / 函数作用域。使用 `{}` 包裹起来的是块级作用域。

```js
if (false) {
    var foo = 'bar'
}
console.log(foo)	// undefined foo 被变量提升了
```

使用 var 创建的变量是没有块级作用域的。然而，使用 let const 是可以创建的

```js
if (false) {
    const foo = 'bar'
}
console.log(foo)	// ReferenceError
```

### 创建作用域的方法

#### 函数式作用域

```js
function foo() { }
```

#### 使用 let / const 创建块级作用域

```js
for (let i = 0; i < 10; i++) {
    console.log(i)
}
console.log(i)		// ReferenceError
```
