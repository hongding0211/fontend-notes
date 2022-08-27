# 原型

## 什么是原型

### \[\[Prototype]]

JavaScript 中几乎所有对象都有**一个特殊的 `[[Prototype]]` 属性**，**它其实就是对另一个对象的引用。**

> 仔细体会上面这句话：每个对象都有个 `[[Prototype]]` 属性，它指向的其实就是一个**对象**。
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

### 函数 prototype 对象

函数有个特性：所有的**函数默认都拥有一个名为 `prototype` 的公有不可枚举属性**，他会指向另一个对象。而一个普通的对象，并不会自带这个 `prototype` 属性。

```javascript
function foo() {}

foo.prototype  // {constructor: ƒ}
```

> 也就是说，一个函数，他默认就“附赠”一个对象；即 `.prototype` 所指向的那个

这个对象通常叫做 “Foo 的原型” ；**这又造成了极大的误导！他只是一个普通对象**，和原型链其实没有实际的关联。

只是我们通常会把某一个对象的 `[[prototype]]` 链接到这个对象上而已；又或者是在 `new` 调用时会将它所返回的对象的 `[[prototype]]` 链接到函数的 `prototype` 对象上：

```javascript
function foo() {}

// 使用 new 调用会返回一个新对象
// 这个对象的 [[prototype]] 会链接到 foo.prototype 上
const bar = new foo()

Object.getPrototypeOf(bar) === foo.prototype    // true 
```

> &#x20;其实将 `[[Prototype]]` 链接到 `prototype` 对象上不过是 `new` 的一个副作用；可以使用 `Object.create()` 来更加直接显式的将两个对象“关联”起来（后文详细展开）

### constructor 属性

函数的 `prototype` 对象中默认还有一个 `constructor` 属性，他在函数声明时自动创建。

而这个 `constructor` 又指向所关联的函数：

```javascript
function foo() {}

foo.prototype.constructor === foo   // true
```

紧接着，观察下面代码：

```javascript
function foo() {}

// new 调用自动将 bar 的 [[Prototype]] 链到 foo.prototype
var bar = new foo()

// 所以，bar 本身并没有 constructor
// 他是顺着原型链找到了上一级 foo.protype
bar.constructor === foo.prototype.constructor   // true
```

上面的这种写法，配合着 `new` ，`constructor` 让我们很容易和面向对象中类的相似概念混淆在一起

其实，`bar.constructor` 不过是顺着原型链查找到的；而且原型链的创建也是因为 `new` 的一个副作用而已

> 长久以来社区为了尽可能的实现“模拟类”的行为，甚至有一种约定： `new` 调用的函数名必须首字母大写
>
> 然而，无论他是否是大写，它本质上就是一个函数。`new` 也不过只是一种特殊的调用方式而已！

不要把 `constructor` 理解为 “构造”，这个属性完全不能正确地 ”标识一个对象是否由另一个构造”；参照以下代码：

```javascript
function foo() {}

foo.prototype = {}

var bar = new foo()

bar.constructor    // f Object()
```

虽然 `bar` 是通过 `foo()` ”new 出来的“ ，但是 `bar` 的 `constructor` 属性并不指向 `foo()` ，反而是指向 `Object()` （通过原型委托）

千万**不要有这样错误地认识！**

### 原型和类的区别

在面向对象的语言中，类本质上就是进行“复制”。

然而，在 JavaScript 当中，尽管可以使用原型来模拟“类”的行为，但是一个最大的区别就是：原型并没有复制操作，仅仅是用委托关联到了另一个对象之上。

### 典型的“原型风格”

下面的代码是一种典型的使用原型来模拟类的行为：

```javascript
function Foo(name) {
    this.name = name
}

Foo.prototype.myName = function() {
    return this.name
}

function Bar(name, label) {
    Foo.call(this, name)    // 盗用构造
    this.label = label
}

// Object.create() 创建一个新对象
// 并且把这个对象的 [[prototype]] 链接到指定对象
Bar.prototype = Object.create(Foo.prototype)

// 修复丢失的 constructor 属性（如果你需要的话）
// 不然他会“错误地”指向 Foo
Bar.prototype.constructor = Bar

Bar.prototype.myLabel = function() {
    return this.label
}

var a = new Bar('a', 'obj a')

a.myName()  // a
a.myLabel() // obj a

```

注意，下面的两种写法都是错误的：

```javascript
// 只是把 Bar 的 prototype 指向了 Foo 的 prototype
// 而并非创建一个新对象并且链接 [[prototype]]
Bar.prototype = Foo.prototype

// 基本上正确，的确创建了新对象并且链接了 [[prototype]]
// 但是 new 本质上是一次 “函数调用”；因此在函数内部可能会执行一些不想要的 “副作用”
Bar.prototype = new Foo()

```

## 如何链接 \[\[prototype]]

可以通过以下几种方式来将一个对象的 `[[prototype]]` 链接到另一个对象：

* `Object.create()`（创建一个新对象，链接到目标对象）
* 使用 `.__proto__ 来设置`
* `Object.setPrototypeOf()`

例如对于上面 “模拟类” 的实现代码中，可以使用 `setPrototypeOf()` 来替换原先的 `Object.create()` 写法：

```javascript
// 可以将这种写法替换为下面这种：
Bar.prototype = Object.create(Foo.prototype)

Object.setPrototypeOf(Bar.prototype, Foo.prototype)
```

第二种做法的 “好处” 就是：不会因为 `Object.create` 创建了一个 ”新对象“ 从而覆盖掉了默认的 `prototype` 对象，还需要重新 “修复” `constructor` 属性；并且被覆盖的对象还需要被 GC。

**不过被上面这种写法过于限制思想，为什么一定要模拟类的行为？**

**为什么一定要将一个对象的 `prototype` 对象的 `[[prototype]]` 链接到另一个对象的 `prototype` 对象上面？为什么不能直接将一个对象 “关联” 到另一个对象上面？**

**在后文中，会展开介绍一种新的 “面向原型” 的设计方式**

## 检查原型链

### instanceof

`instanceof` 操作符左边是 “对象”，右边是 “函数”；它检查的是：

**在这个 “对象” 的整条原型链上，是否有指向该 “函数” 的 `prototype` 对象？**&#x20;

```javascript
function foo() {}

const bar = new foo()

bar instanceof foo  // true
```

> `instanceof` 只能检查一个对象和一个函数的 `prototype` 对象之间的关联，**实属有点丑陋！**而且又会给人一种 “类” 的错觉

使用 `bind()` 方法返回的包装函数，不会有 `prototype` 属性；如果对该包装函数使用 `instanceof` ，实际上会使用 `bind()` 的目标函数的 `prototype` 属性。

### Object.prototype.isPrototypeOf()

你不能通过 `instanceof` 关键字来**直接判断两个对象之间的关系；**如果硬要实现，可以通过下面这种丑陋的方式：

```javascript
var obj1 = {}
var obj2 = {}

// obj1 的 [[prototype]] 链接到了 obj2 
Object.setPrototypeOf(obj1, obj2)

// 创建一个临时函数
// 并设置它的 prototype 为需要检测的对象
function foo() {}
foo.prototype = obj2

// 借用这个临时函数就可以判断 obj1 的 [[prototype]] 是否链接到了 obj2
obj1 instanceof foo    // true
```

借助 `Object.prototype.isPrototypeOf()` 就可以实现：

```javascript
obj2.isPrototypeOf(obj1)    // true
```

它检查的是：目标对象 (obj1 )的原型链上，是否存在自己 (obj2)

### Object.getPrototypeOf()

我们可以使用标准的 `Object.getPrototypeOf()` 来检查一个对象的 `[[prototype]]` 属性，从而避免使用 `.__proto__` 这样的写法：

```javascript
Object.getPrototypeOf(obj1) === obj2    // true
```
