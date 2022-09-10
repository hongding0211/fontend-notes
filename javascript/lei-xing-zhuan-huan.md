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

### toString

* `null` 返回 `'null'`，`undefined` 返回 `'undefined'`，`true` 返回 `'true'`
* 数字返回对应的字符串，极小或极大的使用指数形式
*   普通对象，除非自定义；否则返回内部 `[[Class]]` 属性的值

    > 数组的 `toString()` 方法被重定义了，它将所有元素以 “ , ” 分割再拼成字符串

`toString()` 可以被显式调用，也可以在需要字符串化时自动调用。

#### JSON.stringify()

我们还可以对对象使用 `JSON.stringify()` 来将它序列化为字符串。

在这个过程中，也是用到了 `toString` 的上述规则。不过他们还是不一样的！

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

注意，这里介绍 `JSON.stringify()` 并不是因为它也是一个强制类型转换。而是在它其中，**也运用到了 `toString` 的一些规则**。

例如当遇到对象中某一个属性是 `null` ，它就会应用规则将其转换为 `'null'`

### toNumber

* `true` 转换为 1，`false` 转换为 0
* `undefined` 转换为 `NaN`
* `null` 转换为 0
* 字符串先尝试转换，若失败则变为 `NaN`&#x20;
* 对象先转换为相应的基本类型；若非数字，再应用上面的规则。将对象转换为基本类型会使用 `toPrimitive` 操作，具体的过程为：
  * 检查是否有 `valueOf` 方法；若有，返回基本类型值
  * 如果没有则使用 `toString()` 的返回值，再执行上述规则
  * 如果 `valueOf` 和 `toString()` 均不会返回基本类型，则产生 `TypeError`&#x20;

### toBoolean

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

## 隐式强制类型转换
