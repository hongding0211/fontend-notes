# 类型转换

## 内置类型

JavaScript 一共有 7 中内置类型：`null`, `undefined`, `boolean`, `number`, `string`, `object` 和 `Symbol`&#x20;

使用 `typeof` 可以检查它们的类型；但是 `typeof null` 返回 `'object'` (这是一个 bug）

因此，如果要想检测一个值为 `null` 需要使用：

```javascript
var a = null
!a && typeof a === 'object'    // true，(所有对象都是真值，所以 !a 返回 true)
```

## 强制类型转换

我们在讨论强制类型转换时，转换的结果都是**基本类型**。

我们可以把强制类型转换分为隐式和显式两种，例如下面：

```javascript
var a = 42

var b = a + ''     // 隐式
var c = String(a)  // 显式
```

> 当然，隐式和显式只是相对的；并没有绝对的划分

## 抽象值操作

### ToString

* `null` 返回 `'null'`，`undefined` 返回 `'undefined'`，`true` 返回 `'true'`
* 数字返回对应的字符串，极小或极大的使用指数形式
*   普通对象，除非自定义；否则返回内部 `[[Class]]` 属性的值

    > 数组的 `toString()` 方法被重定义了，它将所有元素以 “ , ” 分割再拼成字符串

`toString()` 可以被显式调用，也可以在需要字符串化时自动调用。

#### JSON.stringify()

我们还可以对对象使用 `JSON.stringify()` 来将它序列化为字符串。

在这个过程中，也是用到了 `ToString` 的上述规则。不过他们还是不一样的！

值得一提的是，在 `JSON.stringify()` 的过程中，会忽略一些 “不安全的 JSON 值” ，即：`undefined`, `function`, `symbol` 以及循环引用（对象之间互相引用）。

这些 “不安全的 JSON 值” 在序列化时，会被自动忽略；如果他们存在某一个数组中，那么则会在对应位置返回 `null`&#x20;

```javascript
JSON.stringify({
    a: function foo() { },
    b: [undefined, function bar(){}]
})
// '{"b":[null,null]}'
```

可以通过定义 `toJSON()` 方法来返回一个安全的 JSON 值

```javascript
var obj = {
    a: 'foo',
    b: function bar() { }
}

// 返回一个指定的安全的对象
obj.toJSON = function() {
    return {
        a: this.a
    }
}
```

注意，这里介绍 `JSON.stringify()` 并不是因为它也是一个强制类型转换。而是在它其中，**也运用到了 `ToString` 的一些规则**。

例如当遇到对象中某一个属性是 `null` ，它就会应用规则将其转换为 `'null'`

### ToNumber

* `true` 转换为 1，`false` 转换为 0
* `undefined` 转换为 `NaN`
* `null` 转换为 0
* 字符串先尝试转换，若失败则变为 `NaN`&#x20;
* 对象先转换为相应的基本类型；若非数字，再应用上面的规则。将对象转换为基本类型会使用 `ToPrimitive` 操作，具体的过程为：
  * 检查是否有 `valueOf` 方法；若有，返回基本类型值
  * 如果没有则使用 `toString()` 的返回值，再执行上述规则
  * 如果 `valueOf` 和 `toString()` 均不会返回基本类型，则产生 `TypeError`&#x20;

### ToBoolean

在 JavaScript 当中，数字 1, 0 不等同于 `true` 和 `false` 。我们虽然可以使用强制类型转换将 1 变为 `true` ，（反之亦然）。但是他们不能画等号！

转换为 `boolean` 时只需要检查这个值是否为 **“假值”** ，以下都是假值

* `undefined`
* `null`
* `false`&#x20;
* `''`&#x20;
* \+0, -0, `NaN`

以上 “假值” 将会转换为 `false`；除此之外，所有值都转换为 `true`&#x20;

对象，数组都是真值，即便是对假值进行封装：

```javascript
!!new Boolean(false)    // true，!! 应用了隐式强制类型转换
!![]                    // true
!!function() {}         // true
```

## 显式强制类型转换

### 字符串和数字之间转换

使用内置的 `String()` 和 `Number()` 函数可以实现字符串和数字之间的相互转换。

注意不需要使用 `new` 构造调用它们，因此并不创建封装对象。

```javascript
String(0)    // '0'
Number('0')  // 0
```

`String()` 和 `Number()` 分别遵循上面介绍的 `ToString` 和 `ToNumber` 抽象值操作的规则。

此外，还可以通过以下的方法来进行 “显式转换”

```javascript
0..toString()    // '0' 注意要两个 .
+'0'             // 0
```

除了 `Number` 可以将字符串转换为数字，还可以使用 `parseInt()` 方法；但他们之间有点区别：

```javascript
Number('0px')    // NaN
parseInt('0px')  // 0
```

`Number` **不支持解析字符串中的一部分**，而 `parseInt()` **在遇到非数字字符时可以截止**，如果失败再返回 `NaN`

`parseInt` 只支持传入字符串，他不能解析其他类型的值。**它会先将传入的值转换为字符串**，再继续下去。

> 当然还有 `parseFloat` 将值转换为浮点数

### 转换为 boolean

可以使用 `Boolean()` 内置函数来显式地将值转换为 boolean；不过一种更常用的方式是使用 `!!`&#x20;

```javascript
!!0    // false
!!''   // false
!![]   // true
```

## 隐式强制类型转换

### 字符串和数字之间的转换

在使用 `+` 运算的时候，会发生隐式转换；因为 `+` 既可以执行加法运算，也能进行字符串拼接。他的具体规则如下：

* 如果其中的一个操作符是字符串，或者可以通过 `ToPrimitive` 抽象操作转换为字符串；那么就执行拼接
* 否则，执行数字加法运算

`ToPrimitive` 操作的具体过程如下：

* 先调用其 `valueOf()` 获得基本类型值
* 如果没有，再调用 `toString()` 方法

> 这个操作和 `ToNumber` 类似

利用上述特性，我们通常会使用 `0 + ''` 这种操作来将数字转换为字符串。

此外，还可以使用 `a - 0` , `a * 1` , `a / 1` 来隐式转换为数字。

### boolean 转换为数字

`true` 会转换为 1，`false` 会转换为 0

```javascript
Number(!![])    // 1, 先使用 !! 保证传入的是 boolean；否则可能会返回 NaN
```

### 转换为 boolean

在以下几种情况下，都会发生隐式的转换，将值转换为 boolean：

* if 语句
* for 语句的第二个表达式
* while, do ... while 语句
* ? : 三目运算符
* || && 左边的操作数

`||` `&&` 操作符的实际行为可能和你想象中的不太一样，因为**他们的返回值并不一定是 boolean**&#x20;

相反，**他们会返回左右两个操作数中的某一个**；具体操作如下：

* 首先对左边的操作数进行判断，此时可能会发生隐式转换将值变为 boolean
* 对于 `||` 如果判断为 `true` 则**返回左边**的操作数；否则返回右边
* 对于 `&&` 如果判断为 `true` 则**返回右边**的操作数；否则返回左边

## == 和 ===

两者的区别在于：`==` **允许在比较时进行强制类型转换**，而 `===` 不允许。

也就是说，如果两个值的类型相同；那么他们使用 `==` 或者 `===` 效果是相同的。

### == 规则

如果两个值的类型相同，则直接比较他们是否相等；不过以下两种情况注意：

* `NaN` 不等于 `NaN`&#x20;
* \+0 等于 -0

如果是两个对象，那么之比较他们的引用是否相同。

但是，如果两个值的类型不同；**那么就会发生隐式强制类型转换，将其中一个或者两个转换为相同类型再进行比较。**下面将分别介绍不同的情况：

### 字符串和数字比较

将两者之间的字符串转换为数字，然后再执行比较。

```javascript
'0' == 0    // true
```

> 转换使用 `ToNumber` 抽象操作

### 其他类型和 boolean 比较

不要和 `Boolean()` 强制转换时判断假值混淆，以下结果可能出乎你的预料：

```javascript
'0' == true     // false，'0' 虽然不是假值，但是结果并不是 true
```

