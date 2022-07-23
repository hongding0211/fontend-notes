# 作用域 & 闭包

## 作用域

负责收集维护所有变量的一系列查询，并实施一套严格的规则，确保代码对这些变量的访问权限

对于 `var a = 0` 这一操作，可以分解成如下：

- 如果变量 a 在作用域中不存在，**编译器**就在作用域中声明一个
- **在运行时**引擎在作用域查找该变量，并且为它赋值

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

- RHS 在查询失败时，会抛出 ReferenceError

  > RHS 查询成功后，但如果对变量的值进行不合理的操作的时候
  >
  > 如访问 null 的属性，会抛出 TypeError

  ```javascript
  var a = b	// RHS 查询失败，ReferenceError: b is not defined
  ```

  

- LHS 在查询失败时，**在非严格模式下**，会创建一个具有该名称的变量

  **具体来说，如果在顶层都无法找到变量，就会在全局作用域内创建一个变量**

  > 在严格模式下，LHS 查询失败时也会抛出 ReferenceError

  ```javascript
  "use strict"
  
  a = 0	// 在严格模式下 LHS 抛出异常：Uncaught ReferenceError ReferenceError: a is not defined
  ```

> ReferenceError 表示作用域判别失败；TypeError 表示作用域判别成功，但是对结果的操作是不合理的。

## 词法作用域

