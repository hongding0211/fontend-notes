# Quick Start

### Route 要用标签包裹

element 替换了原来的 component 属性

```html
<BrowserRoute>
	<Routes>
		<Route path='/xxx' element={<Component />}
		...
	</Routes>
</BrowserRoute>
```

### 组件嵌套路由

```html
<BrowserRouter>
  <Routes>
    <Route path="/" element={<App />}>
      <Route path="expenses" element={<Expenses />} />
      <Route path="invoices" element={<Invoices />} />
    </Route>
  </Routes>
</BrowserRouter>
```

需要在父组件中放置 标签，来容纳嵌套的组件

```html
<div>
  ... static contents...
  <Outlet />
</div>
```

### Nav Link

如果想要 Link 在激活时有独特的样式，就是用 NavLink

```jsx
<NavLink
  style={({ isActive }) => {
    return {
      display: "block",
      margin: "1rem 0",
      color: isActive ? "red" : ""
		    };
  }}
  to='/xxx'
/>
```

### No match case

设置一个 fallback 选项

```html
<Route path="/" element={<App />}>
    <Route path="expenses" element={<Expenses />} />
    <Route path="invoices" element={<Invoices />} />
    <Route
      path="*"
      element={
        <main style={{ padding: "1rem" }}>
          <p>There's nothing here!</p>
        </main>
      }
    />
  </Route>
```

### Index Routes

为这一层路由提供一个默认的 index 页面

```html
<Routes>
  <Route path="/" element={<App />}>
    <Route path="invoices" element={<Invoices />}>
			/* Index */
      <Route
        index
        element={
          <main style={{ padding: "1rem" }}>
            <p>Select an invoice</p>
          </main>
        }
      />
    </Route>
</Routes>
```

* Index 的作用
  * Index routes **render in the parent routes outlet at the parent route's path.**
  * Index routes match when a parent route matches but none of the other children match.
  * Index routes are the **default child route for a parent route.**
  * Index routes render when the user hasn't clicked one of the items in a navigation list yet

### Params

*   **URL Params (.../xxx)**

    继续往下一层级添加一个 :key

    ```html
    <Route path="invoices" element={<Invoices />}>
      <Route path=":invoiceId" element={<Invoice />} />
    </Route>
    ```

    在元素中使用 useParams() 来接住参入的参数
*   **Search Params (...?key1=value1\&key2=value2...)**

    > React Router makes it easy to read and manipulate the search params with `useSearchParams`. It works a lot like `React.useState()` but stores and sets the state in the URL search params instead of in memory.

    直接像 useState 一样声明就好了，调用 set… 时候，自动修改地址为 xxx=xxx

    获取 key 的 value 时也非常自然，直接读取变量就好

### Use Location

`useLocation` returns a location that tells us information about the URL. A location looks something like this

```jsx
{
  pathame: "/invoices",
  search: "?filter=sa",
  hash: "",
  state: null,
  key: "ae4cz2j"
}
```

### Navigate

```jsx
const navigate = useNavigate()

navigate('/xxx/xxx')
navigate(-1)	// 向后退一个
```
