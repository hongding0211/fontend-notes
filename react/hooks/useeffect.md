# useEffect

### Concepts

获取数据、设置订阅、修改组件都属于副作用。

可以把 `useEffect()` 看做是 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。

* `useEffect()` 传入的函数中执行的内容，类似于于 `componentDidMount` 和 `componentDidUpdate`
* `useEffect()` 传入的函数返回的另一个函数，类似于 `componentWillUnmount`

#### 为什么要融合这三个函数？

* 很多时候 componentDidMount 和 componentDidUpdate 内部执行的逻辑是相同的，组件在首次和后续更新中都需要执行某一 Effect，而代码却没有被复用。
* componentDidMount 和 componentWillUnmount 其实也是对应的，在组件挂载时执行了某一副作用（如消息订阅），在卸载之前也应该负责清除。

### 执行 Effect

通过 `useEffect()` 可以让 React **在渲染后执行某些操作**。

> React 保证每次运行 Effect 时，DOM 已经更新完毕

默认情况下（不传入第二个参数），首次渲染和所有重渲都会执行这个 Effect。

### 额外需要清除 Effect

有些情况，如设置了消息订阅，这些 Effect 需要被及时清除。

比如在 Class 组件中，一般在 componentDidMount 中订阅，并且在 componentWillUnmount 中清除这个订阅。

#### Usage

通过 return 一个函数，来在组件卸载的时候执行清除操作。

> ⚠️ 需要注意的是：清除函数虽然类似于 `componentWillUnmount`，但有很大的区别。
>
> **清除函数在每次重渲染时都会执行**，而不同于 `componentWillUnmount`，只会在最后卸载时执行。

**`useEffect()`在调用一个新的 Effect 之前会对前一个 Effect 进行清理：**

```jsx
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // 运行第一个 effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // 运行下一个 effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // 清除上一个 effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // 运行下一个 effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // 清除最后一个 effect
```

#### 为什么要在每次重渲时执行一次 Effect ？

* 因为很多时候的逻辑是需要在首次挂载和更新后都需要执行的。
* 在重渲染时，还能对上一次的 Effect 做一次正确清除。

### 跳过不执行 Effect

`useEffect()` 第二个参数可接受一个数组，来控制是否执行 Effect。

* 如果传入一个空数组，只会在首次挂载后运行。
* 如果传入非空数组，那么只有在数组中的某一元素发生变化时，才会再次执行 Effect。

```jsx
// 只会在首次渲染后执行
useEffect(()=> ..., [])
```

### 使用 async

在 `useEffect()` 内执行 aysnc 函数会导致问题。

因为 async 会隐式的返回一个 Promise 对象，而 `useEffect()` 接受的这个函数要么不返回内容，要么只能返回一个清除函数。

> 🔹 async 函数会将他的返回值包装成一个 resolve 的 Promise 对象（没有返回值则包装一个 undefined）。如果捕获到了异常，则包装成一个 reject Promise。

#### 解决方法

不要直接传入一个 async 函数，而是传入一个同步方法，在内部执行 async 函数。

```jsx
useEffect(() => {
    async function fetchData() {
        // 异步操作...
    }
    
    fetchData()
})
```
