# 相关概念

### Concepts

Hook 可以让你在不使用 Class 组件的情况下使用 state 等 React 特性。

使用自定义 hooks 还可以抽象一些逻辑进行复用

### 使用规则

* ✅ 只有在 Component 函数，或者自定义 Hook 的最顶层使用。
* ❌ 不能在任何分支，循环，嵌套函数中使用。
* ❌ 不能在普通 JavaScript 函数中使用。

#### 为什么不能在分支，循环中使用？

**因为需要保证所有 Hooks 在每次渲染时都按照一样的顺序被调用。**

React 依照你调用 Hooks 的顺序来保证对应关系。考虑下面一个问题：

```jsx
const [foo, setFor] = useState(0)
const [bar, setBar] = setState(false)
```

> **🙋 React 如何在每次渲染时，将 foo, bar 保存的 state 值正确交给组件呢？**
>
> **他必须通过顺序来保证：**
>
> 因为这样 React 就可以知道第一个调用的 useState() 是要获取 foo 的值；
>
> 而第二个调用的 useState() 是要获取 bar 的值。

```jsx
// 首次渲染
useState('Mary')           // 1. 使用 'Mary' 初始化变量名为 name 的 state
useEffect(persistForm)     // 2. 添加 effect 以保存 form 操作
useState('Poppins')        // 3. 使用 'Poppins' 初始化变量名为 surname 的 state
useEffect(updateTitle)     // 4. 添加 effect 以更新标题

// 二次渲染
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
useEffect(persistForm)     // 2. 替换保存 form 的 effect
useState('Poppins')        // 3. 读取变量名为 surname 的 state（参数被忽略）
useEffect(updateTitle)     // 4. 替换更新标题的 effect
```

> 🔹 如果你需要通过条件来控制如 useEffect 的执行，把条件判断放到 useEffect 内部。
