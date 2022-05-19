# Quick Start

## Installation

### Redux Core

```bash
yarn add redux
```

### Redux Toolkit

```bash
yarn add @reduxjs/toolkit
```

> [**Redux Toolkit**](https://redux-toolkit.js.org/) is our official recommended approach for writing Redux logic. It wraps around the Redux core, and contains packages and functions that we think are essential for building a Redux app. Redux Toolkit builds in our suggested best practices, simplifies most Redux tasks, prevents common mistakes, and makes it easier to write Redux applications.

### React Redux

```bash
yarn add react-redux
```

### Create a React Redux App

```bash
# Redux + Plain JS template
npx create-react-app my-app --template redux

# Redux + TypeScript template
npx create-react-app my-app --template redux-typescriptnpm install react-redux
```

## Concepts

### Main Concepts

*   所有的数据存储在一个树结构中（只有一个根结点）

    > The whole global state of your app is stored in an object tree inside a single _store._
*   改变数据的唯一办法就是创建一个 `action (object)`，然后把它 `dispatch` 给 `store`

    > The only way to change the state tree is to create an _action_, an object describing what happened, and _dispatch_ it to the store. To specify how state gets updated in response to an action, you write pure _reducer_ functions that calculate a new state based on the old state and the action.

### Quick Example

比如下面的这个 Todo List 存储的数据

```jsx
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
```

`action` 用来描述修改数据的动作，把每一个动作的`type` 都明确地标注出来有利于更好地理解。

```jsx
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

把`action` 和`state` 结合在一起就是`reducer`，`reducer` 本质上就是一个函数，把 `state` 和`action` 作为**参数**传入，然后**返回**新的`state`

```jsx
function visibilityFilter(state = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter
  } else {
    return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([{ text: action.text, completed: false }])
    case 'TOGGLE_TODO':
      return state.map((todo, index) =>
        action.index === index
          ? { text: todo.text, completed: !todo.completed }
          : todo
      )
    default:
      return state
  }
}
```

每一个“小数据”都可以用一个 `reducer` 来处理，然后最后把它们全部合在一起。返回一个 `object` ，用建值对来存储所有的小`reducer`

```jsx
function todoApp(state = {}, action) {
  return {
    todos: todos(state.todos, action),
    visibilityFilter: visibilityFilter(state.visibilityFilter, action)
  }
}
```

### Immutability 不可变性

Redux 中的 `state` 都是 `immutable` 的，修改时，必须在副本上进行修改，然后赋值给新的 `state`

### Actions

> An **action** is a plain JavaScript object that has a `type` field. **You can think of an action as an event that describes something that happened in the application**.

```jsx
const someAction = {
	type: 'domain/eventName',
	payload: 'xxx'   // optional
}
```

一个 `action` 包含 `type` 和一个可选的属性（通常命名为 `payload` ，用来补充描述动作内容)

`type` 属性一般遵循 `'domain/eventName'` 的命名法则，**domain** 用来描述动作属于那个类别

**Action Creator**

可以用一个函数来动态创建动作

```jsx
const createSomeAction = text => {
	return {
		type: 'domain/eventName',
		payload: text
	}
}
```

### Reducers

> A **reducer** is a function that receives the current `state` and an `action` object, decides how to update the state if necessary, and returns the new state: `(state, action) => newState`. **You can think of a reducer as an event listener which handles events based on the received action (event) type.**

```
reducer` **本质上就是个函数**，传入当前的 `state` 和一个 `action` ，并且返回新的 `state
```

Reducer 必须遵循以下规则：

* 只根据传入的 `state` 和 `action` 来计算新的 `state`
* 不可以直接修改现有的 `state`
* 不要使用异步操作

**Reducer 内部的处理逻辑通常是：**

*   根据传入的

    ```
    action
    ```

    来决定是否要修改

    * 如果需要修改，则拷贝一份，并返回新的 `state`
    * 如果不需要，则返回原来的 `state` （即传入的参数 `state` ）

```jsx
const initialState = { value1: 0, value2: 0 ... }

// state 设置一个默认参数，即定义了 state 的初始值
function xxxReducer (state = initialState, action) {
	if (action.type === xxx ) {
		...
		return {
			...newState
		}
	} else if (action.type === xxx ) {
		...
	}

	...

	// no changes is commited
	return state	
}
```

### Store

`state` 存储在 `store` 中，创建时，传入一个 `reducer` ，然后通过 `store.getState()` 来获取数据

```jsx
import { configureStore } from '@reduxjs/toolkit'

const store = configureStore({ reducer: counterReducer })

console.log(store.getState())
// {value: 0}
```

### Dispatch

```
store.dispatch()` 是更新数据的唯一方式，往里面传入一个 `action
store.dispatch({ type: 'counter/increment' })

console.log(store.getState())
// {value: 1}
```

通常通过 `action creator` 来创建一个 `action` ，然后再传入

```jsx
const increment = () => {
  return {
    type: 'counter/increment'
  }
}

store.dispatch(increment())

console.log(store.getState())
// {value: 2}
```

### Selectors

通过封装一个函数来从数据树中选出需要的内容

```jsx
const selectCounterValue = state => state.value

const currentValue = selectCounterValue(store.getState())
console.log(currentValue)
// 2
```

*   `Selector` 可以在当 `reducer` 内部结构变化时，不需要修改外部 component 的代码，而保持解耦

    It would be nice if we didn't have to keep rewriting our components every time we made a change to the data format in our reducers. One way to avoid this is to define reusable selector functions in the slice files, and have the components use those selectors to extract the data they need instead of repeating the selector logic in each component. That way, if we do change our state structure again, we only need to update the code in the slice file.

⚠️ 不要过度使用 `selector` ，不是所有的数据选择都需要用到它

如果要向 `selector` 中传入参数，使用高阶函数的方法。`selector` 本身只能接受一个 state 参数

```jsx
export const selectXXX = xxxParam => state => ...
```

### Redux Dataflow

* 初始化
  * 通过 `root reducer` 来创建一个 `store`
  * 创建完毕后，`store` 会调用 `root reducer` 一次，根据它返回的值来初始化`state`
  * 当 UI 第一次渲染时，UI 组件会访问 Redux 存储的 `state` ，然后把数据渲染到网页上。同时这些组件还会**订阅** `state` 的更新，当某些 `state` 更新时，UI 也会更新。
* 更新
  * 某一些事件主动`dispatch` 一个 `action`
  * `store` 调用 `reducer` ，并存储新的 `state`
  * `store` 通知**订阅了的组件**数据发生变更
  * UI 组件检查此次更新是否2与自己有关，如果和自己相关的数据发生了变动，重新渲染界面

## Usages

***

### Redux App 结构

* /src
  * /app
    * store.js: Redux `store` 实例
  * /components
    * /component1
      * component1Slice.js: 编写 `reducer`
    * /component2
    * ...

### Redux Store

使用 `configureStore` 方法从 Redux Toolkit 创建 `store` ，需要传入一个含有 `reducer` 属性的

```jsx
/* /src/app/store.js */

import { configureStore } from '@reduxjs/toolkit'
import counterReducer from '../features/counter/counterSlice'

export default configureStore({
  reducer: {
    component1: component1Reducer,
		...
  }
})
```

💡 一个 App 可以有很多个 \`reducer\` 组成，每一个 \`reducer\` 负责一个模块的数据操作。在创建 \`store\` 时，以键值对的方式传入它们，每个 \`reducer\` 的 \`key\` 代表了他们在 \`state\` 中的键名

比如这里创建了一个名为 counter 的 `reducer` ，可以看到在 `state` 中存储了他的数据

```jsx
export default configureStore({
	reducer: {
		counter: counterReducer
	}
})
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e6b685bb-201e-494f-a0ed-db19dbbd3b3e/Untitled.png)

```jsx
// Raw data
{
  counter: {
    value: 0,
    status: 'idle'
  }
}
```

### Redux Slice

一个 `slice` 负责处理一个模块的功能

> The name comes from splitting up the root Redux state object into multiple "slices" of state.

**使用 `createSlice()` 方法来创建**

```jsx
/* /src/components/xxx/xxxSlice.js */

import { createSlice } from '@reduxjs/toolkit'

export const xxxSlice = createSlice({
	name: 'xxx',
	initialState: {
		value: 0,
		...
	}
	reducers: {
		action1: state => {
			state.value += 1 // 在 Redux Toolkit 中，你可以直接修改原来的 state
		},
		action2: (state, action) => {
			state.value += action.payload
		}
		...
	}
})

export const { action1, action2, ... } = xxxSlice.actions
export default xxxSlice.reducer
```

💡 之前提到的 \`{type: 'xxx/xxx'}\` \`action creator\` 都可以不需要自己手写，Redux Toolkit 中的 \`createSlice()\` 方法帮我们完成了这部分内容。

```jsx
xxxSlice.actions.action1()   // 创建了一个 action1
// {type: 'xxx/action1'}
```

* **Mutable**

在 `createSlice()` 中，可以直接对传进来的 `state` 进行修改

*   `createSlice()` 底层实现

    `createSlice` uses a library called [Immer](https://immerjs.github.io/immer/) inside. Immer uses a special JS tool called a `Proxy` to wrap the data you provide, and lets you write code that "mutates" that wrapped data. But, **Immer tracks all the changes you've tried to make, and then uses that list of changes to return a safely immutably updated value**, as if you'd written all the immutable update logic by hand.
* **可选的 `action` 参数**

注意上面的 action2 ， 只有当必要时，我们再传入 `action` 参数。

* 包装 `Action` `Payload`

当 `action` 被 `dispatch` 时，可以传入一个 `payload` 参数，这是其实可以”拦截“ `payload` 参数，把它稍作修改，再真正地执行出去

```jsx
dispatch(xxxAction(payload))
```

在 `createSlice()` 方法中，针对某一个 `action` 按照下列写法：

```jsx
const xxxSlice = createSlice({
	...
	reducers: {
		xxxAction: {
			reducer(state, action) {
				... // 一些对 state 的操作
			},
			prepare(param1, param2 ...) {
				return {
					payload: {...}  // 根据传进来的参数构造一个 payload
				}
			}
		}
	}
})
```

> 其实默认情况下，参入的唯一参数就被直接包装成了 `payload` ，但是我们也可以手动都修改让他重新包装

```jsx
// 使用上面的 action
dispatch(xxxAction(param1, param2 ...))
```

### 使用 Thunks 实现异步操作

> 上述的操作都是同步的，当 `action` 被 `dispatched` 后，`store` 执行相应的 `reducer` 计算新的 `state`

```
trunk` 是一个特殊的 **Redux 函数，\**他可以容纳异步操作，需要下面的\**两层函数**来编写一个 `trunk
```

* 内部 `trunk` 函数，接受两个参数：`dispatch` `getState` 两个函数
* 外部创建函数，创建并返回 `trunk`

```jsx
const asyncAction = paramxxx => {
	return async (dispatch, getState) => {
		const xxx = getState() // 获取当前的 state
		await ...
		...
		dispatch(...)          // dispatch 来更改数据
	}
}
```

然后就可以像其他的 `action` 一样，直接调用 `dispatch` 来调用

```jsx
store.dispatch(asyncAction)
```

### 在 Component 中使用

* 使用 **`useSelector` 来渲染数据**

使用`useSelector` hook来实现对数据的访问，而不是直接把 `store` 引入到组件中

> Our components can't talk to the Redux store directly, because we're not allowed to import it into component files. But, `useSelector` takes care of talking to the Redux store behind the scenes for us. If we pass in a selector function, it calls `someSelector(store.getState())` for us, and returns the result.

实现要先在 `slice` 中定义好 `selector`

```jsx
// selector
export const selectXXX = state => return state.xxx.xxx
```

然后通过 `useSelector` hook 来得到数据

```jsx
const xxx = useSelector(selectXXX)
```

当然，也可以直接向 `useSelector` 传入匿名函数

```jsx
const xxx = useSelector(state => state.xxx + 2)
```

只要 `store` 中的数据发生变更，`useSelector` 就会重新运行一次 `selector` 。如果返回的值发生了变化，那么它就会让组件重新渲染一次

* 使用 `**useDispatch` 分发 `action`\*\*

```jsx
const dispatch = useDispatch()

dispatch(xxxAction())    // xxxAction() 是一个 action creator，所以要调用它
```

### 为 App 指明 Store

在跟节点上，包裹 `<Provider/>` 标签来指名使用的 `store`

```jsx
import store from './app/store'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```
