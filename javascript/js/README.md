# JavaScript 相关概念

### 数据类型

基本：number, string, boolean, undefined, null, symbol, bigint

引用：object

#### 堆栈区别

* 栈
  * 空间小、效率高
  * 存储大小固定
  * 可以直接读取
  * 在定义时就已经分配好了空间
* 堆
  * 空间大，效率低
  * 存储大小不固定
  * 通过引用读取

### Map vs Object

|      | Map                                                                                                                     | Object                                                                                                                                                                                                   |
| ---- | ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 意外的键 | Map默认情况不包含任何键。只包含显式插入的键。                                                                                                | 一个Object有一个原型, 原型链上的键名有可能和你自己在对象上的设置的键名产生冲突。                                                                                                                                                             |
| 键的类型 | 一个Map的键可以是**任意值**，包括函数、对象或任意基本类型。                                                                                       | 一个Object的键必须是一个 [String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String) 或是[Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/Symbol)。 |
| 键的顺序 | Map中的 key 是有序的。因此，当迭代的时候，一个Map对象以插入的顺序返回键值。                                                                             | 一个Object的键是无序的注意：自ECMAScript 2015规范以来，对象_确实_保留了字符串和Symbol键的创建顺序； 因此，在只有字符串键的对象上进行迭代将按插入顺序产生键。                                                                                                            |
| Size | Map的键值对个数可以轻易地通过[size](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/Map/size) 属性获取 | Object的键值对个数只能手动计算                                                                                                                                                                                       |
| 迭代   | Map是 [iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/iterable) 的，所以可以直接被迭代。                    | 迭代一个Object需要以某种方式获取它的键然后才能迭代。                                                                                                                                                                            |
| 性能   | 在频繁增删键值对的场景下表现更好。                                                                                                       | 在频繁添加和删除键值对的场景下未作出优化。                                                                                                                                                                                    |

### Set, Map, WeakSet, WeakMap

**Set**

1. 成员不能重复；
2. 只有键值，没有键名，有点类似数组；
3. 可以遍历，方法有add、delete、has

**WeakSet**

1.  成员都是对象（引用）；

    ```js
    const ws = new WeakSet()
    ws.add(0)		// invalid
    ws.add({})	// valid
    ```
2. 成员都是弱引用，随时可以消失（不计入垃圾回收机制）。可以用来保存 DOM 节点，不容易造成内存泄露；
3. 不能遍历，方法有add、delete、has；

**Map**

1. 本质上是键值对的集合，类似集合；
2. 可以遍历，方法很多，可以跟各种数据格式转换；

**WeakMap**

1. 只接收对象为键名（null 除外），不接受其他类型的值作为键名；
2. 键名指向的对象，不计入垃圾回收机制；
3. 不能遍历，方法同get、set、has、delete；

#### 弱引用

不会被垃圾回收考虑，加入一个对象**只有被一个`弱引用`**，那么就会直接把它清除掉。

### Arguments

在js中，我们在调用有参数的函数时，当往这个调用的有参函数传参时，js会把所传的参数全部存到一个叫arguments的对象里面。它是一个**类数组数据**

```js
function foo() {
  console.log(arguments)	// [1, 2]
}
foo(1, 2)
```

### instanceof

**`instanceof`** **运算符**用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

```js
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}
const auto = new Car('Honda', 'Accord', 1998);

console.log(auto instanceof Car);
// expected output: true

console.log(auto instanceof Object);
// expected output: true
```

#### 手动实现

```js
function myInstanceOf(source, target) {
    let sourceProto = source.__proto__, targetProto = target.prototype
    while (true) {
        if (sourceProto === null)
            return false
        if (sourceProto === targetProto)
            return true
        sourceProto = sourceProto.__proto__
    }
}

console.log(myInstanceOf(auto, Car), myInstanceOf(auto, Object))	// true true
```

### 数组去重

使用 `Set`

```js
function unqiue (arr) {
	  return Array.from(new Set(arr))
}
console.log(unqiue([1, 2, 2, true, true, false]))		// [1, 2, true, false]

// 或者可以简化为
const newArr = [...new Set(arr)]
```

使用 `indexOf`

```js
function unqiue (arr) {
    const res = []
    for (const i of arr) {
        if (res.indexOf(i) === -1)
            res.push(i)
    }
    return res
}
console.log(unqiue([1, 2, 2, true, true, false]))		// [1, 2, true, false]
```

迭代处理

```js
function unqiue(arr) {
    for (let i = 0; i < arr.length; i++) {
        for (let j = i + 1; j < arr.length && arr[j] === arr[j - 1]; j++)
            arr.splice(j--, 1)
    }
    return arr
}
console.log(unqiue([1, 2, 2, true, true, false]))		// [1, 2, true, false]
```

### null & undefined

#### undefined

undefined 一般不会显式地赋值，表示最原始、自然地状态

**下列四种情况会出现 undefined**

1. 声明了一个变量，但没有赋值
2. 访问对象上不存在的属性
3. 函数定义了形参，但没有传递实参
4. 使用 void 对表达式求值

#### null

null 有自己的类型，但是 type of null 会返回 Object

```js
> typeof null
'object'
> typeof undefined
'undefined'
```

### 类数组 vs 数组

#### 转换方法

使用 `Array.from()`

```js
let al1 = {
    length: 4,
    0: 0,
    1: 1,
    3: 3,
    4: 4,
    5: 5,
};
console.log(Array.from(al1)) // [0, 1, undefined, 3]
```

### 内存泄漏情况

1. 意外的全局变量；
2. 闭包；
3. 未被清空的定时器；
4. 未被销毁的事件监听；
5. DOM 引用；

### Object

#### Object.create()

\*\*`Object.create()`\*\*方法创建一个新对象，使用现有的对象来提供新创建的对象的\_\_proto\_\_。

可以通过 `Object.create()` 来实现继承。

```js
const A = {foo: 0}
const B = Object.create(A)
console.log(B.foo)		// 0
```

`Object.create(null)` 创建出来的对象不是 Object 的子类

#### Object.assign()

可以将一个对象复制到另一个对象，会覆盖重名属性。

#### Object.is()

用来比较两个值是否严格相等，和 === 逻辑基本相似

* 都是 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/undefined)
* 都是 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/null)
* 都是 `true` 或 `false`
* 都是相同长度的字符串且相同字符按相同顺序排列
* 都是相同对象（意味着每个对象有同一个引用）
* 都是数字且
  * 都是 `+0`
  * 都是 `-0`
  * 都是 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/NaN)
  * 或都是非零而且非 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/NaN) 且为同一个值

#### Object.setPrototypeOf() Object.getPrototypeOf()

设置 & 获取原型

#### Object.entries()

返回一个对象的键值对

```js
[
  [key1, value1],
  [key2, value2],
  ...
]
```

#### Object.keys() Object.values()

返回一个数组，包含所有 keys / values

### Symbol

为解决对象属性重名的问题，在 ES5 之前，对象属性只能使用 string

Symbol 用来标识独一无二的值

```js
const s = Symbol()	// 注意不通过 new 来创建
```

> Symbol 不能 new ，因为他不是个对象

对象中以 symbol 为 key 的属性不能被 for...in, for...of 遍历，但是可以使用 Object.getOwnPropertySymbols() 来获得所有 symbol 为 key 的属性。

### delete 操作符

**`delete` 操作符**用于删除对象的某个属性；如果没有指向这个属性的引用，那它最终会被释放。

```js
const foo = {a: 0, b: 1}
delete foo.a
console.log(foo.a)  // undefined
```

* delete 删除数组，长度不变，元素变为 undefined
*   如果一个属性是用 var 创建的（其实他挂载在 window 下面），那么不能被delete

    ```js
    var foo = 'bar'
    console.log(window.foo)	// 'bar'
    delete window.foo				// false 不能删除
    console.log(window.foo)	// 'bar'
    ```

### ES6 新增扩展

#### 箭头函数 vs 普通函数

1. 箭头函数的 this 指向在定义时就确定下来的
2. 箭头函数不能通过 call，bind，apply 改变 this 指向
3. 不能作为构造函数使用
4. 没有原型 prototype
5. 不能用作 Generator 函数
6. 没有 arguments

#### String 扩展

1. 加强对 Unicode 支持
2. 字符串遍历
3. repeat() 等方法
4.  模板字符串

    ```js
    `Hello, ${msg}`
    ```

#### RegExp

构造函数第一个参数是正则表达式，指定第二个参数不再报错、u修饰符、y修饰符、s修饰符。

#### Number

1. 二进制和八进制的表示：0bxxxx, 0oxxx
2. 新方法 parseInt(), parseFloat()
3. Number.EPSILON(常数e)，Number.MAX\_SAFE\_INTEGER, Number.MIN\_SAFE\_INTERGER
4. Math 的一些新方法

#### Function

1. 函数默认值
2.  rest 参数，rest 参数必须放在最后

    ```js
    function foo (...args) {
      console.log(args)	// [arg1, arg2, ...]
    }
    ```
3. 函数内部严格模式
4.  name 属性

    ```js
    function foo() {}
    console.log(foo.name)	// 'foo'
    ```

#### Array

扩展运算符，可以理解成 rest 参数的\*\*逆运算\*\*

```js
console.log(...arr)						// 将 arr 转换为用 ，分割的序列

function foo (...arr) { ... }	// 将多个传入参数转换为一个数组
```

### Proxy & Reflect

#### Proxy

**const proxy = new Proxy(target, handler)**，target 是被代理的对象，handler 是配置参数

```js
const obj = {
    foo: 'bar'
}

const proxy = new Proxy(obj, {
    get: (target, property) => 0
})

console.log(obj.foo)				// 正常访问，'bar'
console.log(proxy.foo)			// 通过 proxy 访问，0
```

this 指向问题，被代理对象中的 this 会指向代理对象中的 this

```js
const obj = {
    foo: function() { console.log(this === proxy) }
}
const proxy = new Proxy(obj, {})

obj.foo()		// false
proxy.foo()	// true，被代理的 obj 里的 this 实际上指向了他的代理 proxy
```

#### Reflect

**Reflect** 是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与[proxy handlers (en-US)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global\_Objects/Proxy/Proxy)的方法相同。`Reflect`不是一个函数对象，因此它是不可构造的。

### Class

```js
class Foo {
  	count = 0							// 实例属性
    constructor(bar) {		// 构造函数
        this.bar = bar
    }
    getBar() {						// 实例方法
        return this.bar
    }
}
```

Class 必须用 new 来实例化，而函数是可以不需要的

#### 静态方法 & 静态属性

```js
class Foo {
    static bar() {
        return 'bar'
    }
  	static bar = 'bar'
}

Foo.bar()							// ok
new Foo().bar() 			// not ok
console.log(Foo.bar)	// 'bar'
```

#### 继承

```js
class A {}
class B extends A {...}
```

子类的构造函数必须调用 super 方法，来获得 this 对象，因为子类没有 this，需要改造父类的this

#### ES5 原型继承 vs ES6 Class 继承

* 原型继承不会继承父类的属性；而 Class 继承可以

### Module

*   CommonJS：用于 Node.js

    ```js
    const module = require('module')
    ```
* AMD：用于浏览器

ES6 将一个文件即视为一个模块，通过 export 暴露，再使用 import 导入

#### export

支持导出：变量，函数，类

```js
export const foo = 'foo'
export function bar () {}

// 也可以使用
export {foo, bar}
export {foo as foo2} // 可以重命名
```

export 不能处于块级作用域内

```js
function foo () {
  export default 'bar'	// not ok
}
```

#### import

```js
import {foo, bar} from 'xxx.js'
```

#### export default

```js
// a.js
export default function() { ... }

// b.js
import whateverName from './a.js'	// 导入的时候不需要知道 export 的名字，而且可以自行命名
```

### Common.js

Common.js 加载时同步的，只有加载完成后，才能继续执行

```js
exports.name = function() { return 'foo' }		// 导出

cosnt getName = require('name')								// 导入
```

#### Common.js vs ES6 Module

* ES6 是编译时引入的，而 Common.js 是运行时引入的
* ES6 输出的模块是引用，而 Common.js 是拷贝
* ES6 可以引用 Common.js，但反之不可

### Iterator

结构类似于一个带头结点的 `LinkedList`

```js
// inteface
{
  value: any,
  done: boolean
}
```

Iterator 接口部署在 `Symbol.iterator` 属性上

```js
const obj = {
  [Symbol.iterator]: function() {
    return {
      next: function() {
        value: 1,
        done: true
      }
    }
  }
}
```

上述 obj 就实现了 Iterator 接口，他是可迭代的

* for...in 遍历属性
* for...of 消费 iterator

### ... 扩展运算符

* 函数调用，将数组转换为一个个参数
* 深拷贝数组，合并数组
* Math.max(), \[].push()

### 单线程

**Q: JavaScript 为什么使用单线程？**

A: JS主要实现用户与浏览器的交互，以及操作DOM，使用单线程会避免许多问题。如，一个线程修改了 DOM 元素，另一个又要删除这个 DOM 元素，这时候会产生冲突。

> 线程：CPU 调度的最小单位
>
> 进程：资源分配的最小单位

### TDZ

在代码块内，使用 let, const 声明之前，该变量都不可用

```js
if (true) {
    // TDZ start
    foo = 'bar'
    console.log(foo)
    // TDZ end
    let 
}
```

TDZ 的本质是：只要进入某一作用域，在声明变量前，该变量都不可用

### 闭包

**函数和函数能访问到的变量就叫闭包**

* 可以让内层函数访问到外层函数内容
* 避免使用全局变量

### Generators / yield

可以控制函数的执行，yield 暂停函数，next() 启动执行

```js
function* foo() {
    yield 'hello'
    yield 'world'
    return 'end'
}

const f = foo()
console.log(f.next())	// {value: 'hello', done: false}
console.log(f.next()) // {value: 'world', done: false}
console.log(f.next())	// {value: 'end', done: true}
```

### XHR 的 readyState

1. 初始化
2. 开始发送请求
3. 请求发送完成
4. 开始读取服务器相应
5. 读取完成

### == ===

\==：比较数值是否相等

\===：比较数值是否相等之外，还比较类型

* NaN 和谁都不等，甚至包括自己，只能用 isNaN判断
* 一个字符串 == 数字时候，将字符串转换成数字
* 对象的比较不同于值比较，比较他们的引用

#### 隐式转换

1. any + 字符串 => 字符串
2. bool + 字符串/数字 => 1 + 字符串/数字
3. undefined + 数字 => NaN + 数字 = NaN
4. null + 数字 => 0 + 数字

#### Examples

```js
[] == []		// false
[] == 0			// true, [] => 0
```
