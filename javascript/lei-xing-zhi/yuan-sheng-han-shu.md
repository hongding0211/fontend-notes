# 原生函数

JavaScript 提供了一些内置函数，他们是：

* `String()`
* `Number()`
* `Boolean()`
* `Array()`
* `Object()`
* `Function()`
* `RegExp()`
* `Date()`
* `Error()`
* `Symbol()`&#x20;

## \[\[class]]

所有 `typeof` 返回为 `'object'` 的对象（包括数组）都包含有一个内部的 `[[class]]` 属性。

这个属性不能直接查看，但我们可以通过 `Object.prototype.toString()` 来查看：

```javascript
Object.prototype.toString.call([])    // '[object Array]'
```

如果传入一个基本类型的话，他会先进行封装操作，将其转换为对应的对象。

```javascript
Object.prototype.toString.call(0)    // '[object Number]'
```

## 包装 & 拆封

### 包装

因为基本对象没有 `.length` 或 `.toString()` 等操作，因此需要将其包装成对应的对象才能操作。

引擎往往会帮你自动进行包装和拆封，因此你不需要过度关心。

如果想自行封装，可以使用 `Object()` （不要加 `new` 构造调用）

```javascript
Object(true)    // 返回一个包装的 Boolean 对象
```

### 拆封

而如果想要拆封的话，可以使用 `.valueOf()` 来获取包装对象内的原始值：

```javascript
Object(42).valueOf()    // 42
```

## 使用内置函数构造对象

一般我们都是用字面量的形式来创建 array, object 或 function。但我们也可以使用这些内置函数来创建。

> 仍然尽量选用字面量的形式来创建数组，对象或函数

### Array()

使用 `Array()` 内置函数构建数组时，`new` 可以不加（如果不加，自动补上）

```javascript
new Array(1, 2, 3)    // [1, 2, 3]
```

然而，如果值传入一个数值；这个数值将被当成数组的 “预设长度”，但是里面并没有数据。这是一种 “空单元数组” ，会引起许多奇怪的操作。

他和全是 `undefined` 元素的数组并不一样！虽然，对于一个空单元数组，你视图访问里面某一个元素，也能返回 `undefined`。

```javascript
// arr1 和 arr2 不是一种东西！
const arr1 = new Array(3)
const arr2 = [undefined, undefined, undefined]

// 虽然访问他们的第一个元素，都能返回 “相同” 的 undefined
arr1[0]    // undefined
arr2[0]    // undefined
```

### RegExp()

`RegExp()` 可以动态地创建一些正则表达式

```javascript
const re = new RegExp(`${str}`, 'g')
```

### Date()

`Date()` 和 `Error` 用的比较多，因为没有字面量的形式来创建。

在使用 `Date()` 时，如果不带 `new` 构造调用；他直接返回一个当前时间的字符串值。

> 仔细区分，这和 `Array()` 不带 `new` 不同；那个是可以省略，这个表达了不同的意义！

还可以使用静态函数 `Date.now()` 来返回一个当前时间的时间戳：

```javascript
Date.now()    // 1662902356823

// 上述操作等同于：
new Date().getTime()
```

`new Date()` 如果不传入参数，即以现在的时间创建一个 `Date` 对象

### Error()

与 `Array()` 一样，`new` 关键字可以省略。一般与 `throw` 配合使用

```javascript
throw new Error('some error')
```

### Symbol()

`Symbol()` 不能使用 `new` 构造调用，否则会出错。

### Object() Function()

除非万不得已，不然别用它们来创建对象 / 函数

## 内置函数的原型对象

有些内置对象的原型对象有些特殊，他们的 `.prototype` 属性不是一个普通对象

```javascript
typeof Function.prototype    // 'function' 一个函数

RegExp.prototype.toString()    // '/(?:)/' 一个空正则表达式

Array.isArray(Array.prototype)    // 一个空数组
```

函数、正则对象以及数组的 `.prototype` 属性都是对应的一个 “空” 的东西，因此用他们来做为 “默认值” 再好不过了：

```javascript
// 如果 someArr 不存在就赋一个空数组
const arr = someArr || Array.prototype
// 同理
const fn = someFn || Function.prototype
const re = someRe || RegExp.prototype
```
