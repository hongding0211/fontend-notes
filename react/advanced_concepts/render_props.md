# Render Props

### Render Props

Render Prop 是一种在组件之间共享代码的方法，通过传入一个值为函数的 prop 来实现。

这个传入的函数 prop 会返回一个 React 元素，接受这个函数 prop 的组件会调用这个函数来完成渲染。

> Render Props 的本质就是给一个组件传入一个 render 方法，组件根据这个方法来进行渲染

### 简单案例

构建一个 Mouse 组件，它可以实现自定义鼠标指针效果。通过传入一个 render prop 来指明需要的鼠标样式。

```jsx
// mouse/index.jsx
import { useState } from "react";

// 鼠标指正的具体元素由传入的 render 函数负责渲染
// Mouse 组件只负责监听鼠标移动事件，并且根据传入的 render prop 将自定义鼠标元素渲染到指定位置
export default function Mouse({ render = undefined }) {
    const [position, setPosition] = useState({ x: 0, y: 0 })

    function handleMouseMove(e) {
        setPosition({ x: e.clientX, y: e.clientY })
    }
    
    return (
        <div onMouseMove={handleMouseMove} style={{ position: 'fixed', width: '100vw', height: '100vh' }}>
            {render(position)}
        </div>
    )
}
```

```jsx
// App.js

// 调用 Mouse 组件的时候，传入一个 render 函数，来指定具体渲染的内容
export default function App() {
    return (
        <div className="container">
            <Mouse render={position => {
                return (
                    <div style={{ position: 'fixed', left: position.x, top: position.y }}>
                        <img src={cursor} alt='cursor' />
                    </div>
                )
            }} />
        </div>
    )
}
```

![iShot2022-05-06\_00.21.38](../../.gitbook/assets/iShot2022-05-06\_00.21.38.gif)

#### 传入的 prop 名称不一定必须是 'render'

其实这个 render 函数的名字可以使任意的，也可以利用 `children props`

```jsx
// App.js

export default function App() {
    return (
        <div className="container">
            <Mouse>
                {position => ( ... ) }                }
            </Mouse>
        </div>
    )
}
```
