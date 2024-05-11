# Some details

### 父组件和子组件的 useEffect 谁先执行？

**子组件先执行**

```javascript
function Child() {
  useEffect(() => {
    console.log('Child')
  })

  return (
    <p>Child Component</p>
  )
}

function App() {
  useEffect(() => {
    console.log('App')
  })
  return <Child />
}

// Child
// App
```



