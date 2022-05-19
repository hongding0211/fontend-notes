# handle\_events

### 添加事件回调

```jsx
<button onClick={handleClick}>Click Me</button>
```

### 将回调作为 prop 传入

可以将回调函数作为 prop 传入给子组件。

```jsx
function Button({onClick, children}) {
    return (
        <button onClick={onClick}>{children}</button>
    )
}

function App() {
    return (
        <div className="container">
            <Button onClick={() => console.log('Clicked')}>Click Me</Button>
        </div >
    )
}
```

#### 自定义回调函数命名

可以给组件使用自定义的回调函数命名。一般情况下，都以 `on` 开头。

### 事件冒泡

当子组件触发了 click 事件后，父组件也能捕获到该事件。

```jsx
function App() {
    return (
        <div className='container'>
            <div className='block' onClick={() => console.log('Click block')}>
                <button onClick={() => console.log('Click button')}>Button</button>
            </div>
        </div >
    )
}
```

![iShot2022-05-19\_00.07.53](../../.gitbook/assets/iShot2022-05-19\_00.07.53.gif)

#### stopPropagation()

可以在子组件中调用 `stopPropagation()` 来阻止事件继续向上冒泡：

```jsx
function App() {
    function handleButtonClick(e) {
        e.stopPropagation()
        console.log('Click button')
    }

    return (
        <div className='container'>
            <div className='block' onClick={() => console.log('Click block')}>
                <button onClick={handleButtonClick}>Button</button>
            </div>
        </div >
    )
}
```

#### 事件捕获

可以在父组件上设置 `onClickCapture` 属性来使用事件捕获的方式，无论子组件有没有设置 `stopPropagation()`，父组件总能率先捕捉到事件。

> 🔹 事件捕获与事件冒泡正好相反，因此父组件可以率先被捕获到

```jsx
function App() {
    function handleBlockClick(e) {
        console.log('Click Block')
    }

    function handleButtonClick(e) {
        e.stopPropagation()
        console.log('Click button')
    }

    return (
        <div className='container'>
            { /* onClickCapture 的函数会被率先执行 */ }
            <div className='block' onClickCapture={handleBlockClick}>
                <button onClick={handleButtonClick}>Button</button>
            </div>
        </div >
    )
}
```

![iShot2022-05-19\_00.14.08](../../.gitbook/assets/iShot2022-05-19\_00.14.08.gif)

#### 手动传入回调来代替冒泡

比起默认实现的事件冒泡，可以通过手动传入回调的方法来实现：

```jsx
function Child({ onClick, children }) {
    function handleClick(e) {
        // 先阻止继续冒泡
        e.stopPropagation()
        // 手动调用传入的回调
        onClick()
    }
    
    <button onClick={handleClickl}>{children}</button>
}
```

这样做的好处就是不仅子组件可以先做一些处理，同时父组件可以继续执行额外的一些工作。

比起自动冒泡，这种做法能够更好地做到追踪。

### 阻止默认行为

点击在表单中的按钮会默认提交并刷新页面，使用 `preventDefault()` 来阻止默认提交行为

```jsx
 <form onSubmit={e => {
        e.preventDefault();
        alert('Submitting!');
    }}>
    <input />
    <button>Send</button>
</form>
```
