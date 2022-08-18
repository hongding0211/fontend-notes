# 原型

## 什么是原型

### \[\[Prototype]]

JavaScript 中几乎所有对象都有**一个特殊的 `[[Prototype]]` 属性**，**它其实就是对另一个对象的引用。**

> 好好领悟上面这句话：每个对象都有个 `[[Prototype]]` 属性，它指向的其实就是一个**对象**。
>
> 这个对象**和其他普通对象一样**，并没有什么神奇的地方！
>
> 因此，不要被所谓“原型继承”的概念误导，更不要把原型和面向对象中的“类”混为一谈！（他们完全不是一个东西，也不要借用面向对象的想法来理解原型）

而 `[[Prototype]]` 的作用就是当触发 `[[Get]]` 操作时，如果在对象本身找不到对应的属性，就会顺着原型链一路寻找上去，这种方式称为“**委托**”，在后文会详细展开。

```javascript
var obj = Object.create({
    a: 0
})

// a 并不在 obj 上，而是在 obj 的原型上
// 但是他能“正确”地找到 0
obj.a   // 0
```

> 当遍历完整条原型链都没有找到对应属性时，返回 `undefined`

在使用 `for ... in` 时也会遍历整条原型链（当然确保属性的 `enmerable` 为 `true`）

```javascript
var obj = Object.create({
    b: 0
})

obj.a = 1

for (var k in obj) {
    console.log(k)  // a b
}
```

### 原型链的尽头

所有普通的原型链的尽头就是 `Object.prototype` ，这个对象内有许多通用的方法，如 `toString()` `valueOf()` 等...

> 当然，`Object.prototype` 的 `[[Prototype]]` 是 `null`

### 属性屏蔽

当给一个对象设置属性时，并没有想象中的简单。而是与该对象所关联的原型链有关。

对于一条的简单赋值语句 `obj.a = 0` 来说，可以分为三种情况：

* 如果对象中已经存在该属性，则修改该属性
* 如果对象中不存在该属性，并且整条原型链上也不存在；那么属性就会被添加到对象上面
* 如果对象中不存在，但原型链上存在同名属性；那么情况就会**变得非常复杂**，下面展开细说

对于对象中不存在某一属性，但原型链上却存在同名属性的情况；尝试设置某一属性也许会得到意想不到的结果：

1. 如果原型链上的同名属性的 `writable` 不为 `false` ，则就在该对象上添加这一属性并对原型链上的同名属性形成**遮蔽**
2. 如果同名属性的 `writable` 为 `false` ，那么即不能修改这个同名属性，也不能在对象本上创建一个屏蔽属性（默认静默，严格模式抛出 TypeError）
3. 如果原型链上存在一个同名 `setter` ，那么就一定会调用这个 `setter` ，而不会将这个属性添加到对象自身上

可见，对于上述的三种情况，只有在属性不是只读的情况下，并且不存在同名 `setter` 时，才能正确的进行属性屏蔽。

```javascript
var foo = {}

// foo.bar 是一个只读属性
Object.defineProperty(foo, 'bar', {
    value: 0,
    writable: false
})

var obj = Object.create(foo)

// 静默处理
obj.bar = 1
```

其实对于第 2 条，如果改用 `defineProperty()` 而不是使用 `=` 进行属性设置，则不会有这条限制。

## 原型 vs 类？

千万不要把原型和类混为一谈，JavaScript 完全没有所谓”类“的概念；也没有所谓的实例化、类构造函数等概念。

> JavaScript 中一些关键词如 `new` ， `prototype.constructor` 非常有误导性！不要用其他面向对象的语言的概念来对照这理解这些关键词

然而，多年以来社区一直尝试用各种手段来“模拟”类的行为，具体如下：

### 函数 prototype

函数有个特性：所有的**函数默认都拥有一个名为 `prototype` 的公有不可枚举属性**，他会指向另一个对象。而一个普通的对象，并不会自带这个 `prototype` 属性。

```javascript
function foo() {}

foo.prototype  // {constructor: ƒ}
```

> 也就是说，一个函数，他默认就“附赠”一个对象；即 `.prototype` 所指向的那个

这个对象通常叫做 “Foo 的原型” ；**这又造成了极大的误导！他只是一个普通对象**，和原型链其实没有实际的关联。

只是我们通常会把某一个对象的 `[[prototype]]` 链接到这个对象上而已；又或者是在 `new` 调用时发生的自动绑定：

```javascript
function foo() {}

// 使用 new 调用会返回一个新对象
// 这个对象的 [[prototype]] 会链接到 foo.prototype 上
const bar = new foo()

Object.getPrototypeOf(bar) === foo.prototype    // true 
```

>
