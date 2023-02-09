# useEffect vs useLayoutEffect

## 执行顺序

* `useInsertionEffect` 在 DOM 插入之前执行
* `useLayoutEffect` 在 repaint 之前执行
* `useEffect` 在 DOM 操作完成后执行

所以他们的执行顺序如下：

```javascript
function App() {
  useInsertionEffect(() => {
    console.log('use insertion effect')  # 1
  }, [])
  useLayoutEffect(() => {
    console.log('use layout effect')     # 2
  }, [])
  useEffect(() => {
    console.log('use effect')            # 3
  }, [])
  return (
    <div id='box' style={{width: 100, height: 100, background: 'blue'}} />
  );
}

// Output:
// use insertion effect
// use layout effect
// use effect
```

> `useInsertionEffect` 因为发生在 DOM 插入前执行，所以即便在 `dev` 环境下（effect通常会被执行两边），它也只会执行一次

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

