# React.lazy()

可以使 React 动态地加载组件

```jsx
// instead of using:
import Button from './components/Button'

// use:
const Button = React.lazy(() => import('./components/Button'))


export default function App() {
    return (
        <Button type="primary">Hello</Button>
    )
}
```

组件在首次渲染时，动态导入所需的其他组件。

`React.lazy()` 传入一个函数，这个函数需要：

* 动态地调用 import()
* 返回一个 Promise，这个 Promise 必须要能够 resolve 一个 React 组件（只能是 export default 暴露的)

## export default

React.lazy() 只接受使用 export default 暴露的组件，即在正常模式下：

```jsx
// ok 这样的组件可以被 React.lazy() 懒加载
import Button from './components/Button'

// not ok
import { Button } from './components/Button'
```

> 例如 Antd 组件库使用的就不是 export default，可以在中间额外地添加一层来将其转换成 export default

```js
// MyButton.js
export { Button as default } from 'antd'
```

## Suspense

利用 Suspense 可以做到优雅地 fallback。设置 Suspense 的 fallback 属性，可以在组件没有完成加载前，预先展示其他组件。

```jsx
import { Spin } from "antd"
import React, { Suspense } from "react"

// 模拟一个组件耗时加载
const Button = React.lazy(async () => {
    await new Promise(resolve => setTimeout(resolve, 1000))
    return import('./components/Button')
})

export default function App() {
    return (
        <Suspense fallback={<Spin />}>
            <Button type="primary">Hello</Button>
        </Suspense>
    )
}
```

![iShot2022-04-30\_11.58.19](../img/iShot2022-04-30\_11.58.19.gif)

> Suspense 可以放在需要懒加载的组件的任意父节点上，Suspense 中也可以包裹任意多个懒加载组件。

## Transition

> 使用 Suspense 并不是万能的，有时候不必要为所有懒加载组件展示一个 fallback。
>
> 比如，当用户从一个组件切换到另一个新组件时，在新组建加载完成之前，可以\*\*仍然展示旧组件\*\*。
>
> 这避免了使用 Suspense 后造成的频繁闪屏问题，即每当用户切换组件时，**旧的组件立马消失，并展示 fallback 组件，最后才展示新的组件。**

参考下面的例子：如果利用了 Suspense 属性，在切换到新组件时，会有一段 fallback 组件展示时间。

```jsx
// Button, Rate 都是两个懒加载组件，模拟了耗时加载
export default function App() {
    const [toggle, setToggle] = useState(true)
    return (
        <div className="container">
            <Suspense fallback={<Spin />}>
                {/* 在两个懒加载组件中切换 */}
                {toggle ? <Button type="primary">Hello</Button> : <Rate defaultValue={5} />}
            </Suspense>
            <button onClick={() => setToggle(!toggle)}>Switch</button>
        </div>
    )
}
```

![iShot2022-04-30\_12.12.59](../img/iShot2022-04-30\_12.12.59.gif)

使用 `startTransition` 使得在新组建加载完成前，仍然显示旧的组件：

```jsx
export default function App() {
    const [toggle, setToggle] = useState(true)
    
    function handleClick() {
        startTransition(() => {
            setToggle(!toggle)
        })
    }
    
    return (
        <div className="container">
            <Suspense fallback={<Spin />}>
                {toggle ? <Button type="primary">Hello</Button> : <Rate defaultValue={5} />}
            </Suspense>
            <button onClick={handleClick}>Switch</button>
        </div>
    )
}
```

![iShot2022-04-30\_12.18.35](../img/iShot2022-04-30\_12.18.35.gif)

与上面的对比，当按下 Switch 后，旧的 Button 组件仍然被展示，直到的 Rate 组件完成加载。

> ⚠️ 当然，别忘了，懒加载只是应用于首次渲染。
>
> 当所有组件完成加载后，之后就不需要所谓的 fallback 或者是 transition

## 配合路由

懒加载还可以配合路由来完成，并且配合 Suspense，在页面切换时，展示 fallback 页面（如骨架屏）。

```jsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

// 不同页面都使用懒加载
const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  </Router>
);
```
