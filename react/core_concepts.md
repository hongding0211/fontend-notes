# 核心概念

### JSX

#### 防止注入攻击

React DOM 在渲染输入内容之前，默认会进行转义（如 `<`， `>`），所有内容都会变成字符串，一次来避免 XSS 攻击。

#### JSX 转换

Babel 会将 JSX 转译成 `React.createElement()` 的调用，并实际上返回一个对象。可以理解成 JSX 是一种语法糖。

```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
// 上面的代码等同于
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

最终，他会返回一个类似这样的对象：

```js
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

### createRoot()

将 React DOM 的根节点挂载到某一真实 DOM 上。

```jsx
const el = <h1>Hello World</h1>;
const root = ReactDOM.createRoot(document.getElementById('root'))
// 然后再通过 render() 来渲染
root.render(el)
```

> 实际情况下，root.render() 一般只会调用一次；更多的是通过状态来更新视图

### 组件 & Props

函数组件，接受一个 props 的参数

```jsx
function Foo(props) {
    return <h1>Hello {props.msg}</h1>
}
```

类组件：

```jsx
class Foo extends React.Component {
    render() {
        return <h1>Hello {this,props.msg}</h1>
    }
}
```

#### props

JSX 所接收到的**属性**和\*\*子组件\*\* children 都会被放入 props 中。

props 是只读的，对于函数式组件，保证了它是一个 pure function。

### 类组件 & state

组件每次更新是都会调用 render() 方法，但是只要在相同的 DOM 节点中渲染该组件，就仅仅会创建一个类组件的实例。

#### state

state 的初始化在类组件的构造函数 constructor 中：

```jsx
class App extends React.Component {
	constructor(props) {
		super(props)
		this.state = {
			msg: 'Hello World'
		}
	}

	render() {
		return (
			<h1>{this.state.msg}</h1>
		)
	}
}
```

> 类组件的构造函数内，必须调用 super() 方法传入 props

#### state 的使用注意事项：

1.  使用 `this.setState({key: value})` 来修改 state，而不是直接对 state 进行修改。（构造函数除外）

    > 直接对 state 修改不会重新渲染组件。
2. state 更新是异步的，react 可能会同时将多个 setState() 合并为一个

#### setState()

setState 除了可以接受一个 state 对象，还可以接受一个函数。

函数传入当前的 state 和 props，然后返回新的对象来更新 state：

```jsx
this.setState((state, props) => { count: state.count + props.step })
```

> state 的数据流是单向自上而下的。

### 事件处理

\###　阻止提交

在原生 HTML 操作中，可以在回调函数中 return false 来避免表单提交；而 react 中需要显式的使用 preventDefault()。

```jsx
function App() {
	function handleSubmit(e) {
		// explicitly prevent submit
		e.preventDefault()
	}

	return ( 
		<div>
			<form onSubmit={handleSubmit}></form>
		</div>
	 )
}
```

#### 类组件 this 绑定

在类组件中为元素添加事件回调函数，需要额外的绑定实例的 this 。这是因为 class 内的方法默认不会绑定 this

```jsx
// 在构造函数中绑定
constructor(props) {
    ...
    this.handleClick = this.handleClick.bind(this);
}

// 也可以在元素上直接绑定
<button onClick={this.handleClick.bind(this)} />

class Button extends React.Component {
    // 或者还可以使用 class fields
    handleClick = () => {
    	...
	}
}
```

另外一种方法是使用箭头函数

```jsx
// 此时的 this 自动绑定了当前上下文
<button onClick={() => this.handleClick()} />
```

> 使用箭头函数绑定回调函数虽然也可以，但是每次调用 render() 时，都会创建一个不同的回调函数。
>
> 如果将这个回调作为 props 传入下一个子组件中，那么还会触发子组件重新渲染。
>
> 因此，可以使用 bind() 方法来规避这个问题

#### 回调函数中传入额外参数

下面两种方式都可以

```jsx
// 使用箭头函数式，需要显式的将 e 也传入到第二个参数中
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>

// 而使用 bind 方法，不许要显式的传入 e，它被默认作为第二个参数传入
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

### key

* key 用来帮助 React 来识别哪些元素发生了变化，key 最好选用独一无二的值，只有万不得已时才使用 index。
* key 只需要在兄弟节点之间保持唯一性即可。
*   key 只是在 React 内部使用，在组件内无法获取到；如果想实现类似的功能，使用 id 属性。

    ```jsx
    // Post 组件内部可以读到 id 属性
    const content = posts.map((post) =>
    <Post
        key={post.id}
        id={post.id}
        title={post.title} />
    )
    ```

### 受控组件

诸如 input, textarea, select 表单元素，在 React 中使用它们时，可以让组件的 state 来接管表单元素中的内容。

使 state 成为元素内容的”单一来源“。这种方式称为受控组件。

```jsx
function App() {
	const [content, setContent] = useState('')

	function handleInputChange(e) {
		setContent(e.target.value)		
	}

	return (
		<input value={content} onChange={handleInputChange} />
	)
}
```

### 状态提升

React 中数据流是单向向下的。因此，如果多个组件需要共享同一个数据源，可以把它们放到一个父组件中，在父组件中管理 state 来保持彼此之间数据的同步。
