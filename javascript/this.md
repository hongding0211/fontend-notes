# this

### call apply bind

```js
function Base(foo) {
    this.foo = foo
}
function Child(foo) {
    Base.call(this, foo)	// 可以理解成继承，不过是把 this 替换成了 Child 的了
}
const child = new Child('foo')
```

#### 相同

改变函数内部的 this 指向

#### 区别

call，apply 会调用函数；而 bind 不会调用

call 接受若干参数，apply 接受一整个数组

#### 应用

* call 多用作继承
*   apply 多用于跟数组有关的操作，如求 max, min

    ```js
    const arr = [1, 22, -10, 0, 9]
    const max = Math.max.apply(Math, arr)
    console.log(max)		// 22
    // 利用 ... 运算符也可以
    const max = Math.max(...arr)
    ```
*   bind 不调用函数，但会改变内部 this

    ```js
    x = 0       // global
    const foo = {
        x: 10,  // foo
        getX: function() { return this.x }
    }

    console.log(foo.getX())     // 10

    const retrieveX = foo.getX
    console.log(retrieveX())    // 0，函数里的 this 实际指向的是 global

    // 注意 bind 返回的才是绑定过的对象
    const boundGetX = retrieveX.bind(foo)
    console.log(boundGetX())    // 10, 函数里的 this 重新指向了 foo
    ```

### this 指向

#### 普通函数中的 this

谁调用了这个函数，this就指向谁

```js
function foo() {
    console.log(this)
}
const obj = {
    bar: function() {
        console.log(this)
    }
}
foo()       // window，因为 foo 是在全局作用于调用的
obj.bar()   // obj
```

#### 匿名函数的 this

匿名函数的 this 具有\*\*全局性\*\*，指向 window

```js
const obj = {
    bar: function () {
        return function () { console.log(this) }
    }
}
obj.bar()()   // window
```

#### 箭头函数的 this

* 箭头函数的 this 是在定义时候确定下来的，而不是在调用时决定的
*   this 指向父级作用域的上下文

    > 如何寻找箭头函数的 this 指向？
    >
    > **找到离箭头函数最近的function，与该function平级的执行上下文中的this即是箭头函数中的this**
* 箭头函数无法使用 apply，call，bind 改变 this 指向

```js
const obj = {
    bar: function () {
        return () => console.log(this)
    }
}
obj.bar()()   // obj
```
