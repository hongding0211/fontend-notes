# 异步操作

> [Redux Essentials, Part 5: Async Logic and Data Fetching | Redux](https://redux.js.org/tutorials/essentials/part-5-async-logic)

## Concepts

默认情况下，Redux store 并不处理任何异步操作，它只负责同步地\*\*分发操作、更新 state 以及通知 UI 组件数据发生变更。\*\*而如果要实现异步操作，则让他们发生在 Redux 外部

但是，如果想要通过 dispatch 把 action 分发出去后，在 Redux 内部处理异步请求的话，就需要引入相应的中间件，它可以实现

* 当 action 被 dispatched 后，可以执行而外的操作
* 暂停、修改、延迟、替换或中断已经 dispatched 的 action
* 可以直接接触到 dispatch 和 getState

## Redux-thunk

![](https://raw.githubusercontent.com/HongDing97/imgs/main/20211202142702.png)

`redux-thunk` 相当于在操作 store 前，先做了拦截，等到完成后在真实地 dispatch

### Trunk Functions

A thunk function will always be called with `(dispatch, getState)` as its arguments, and you can use them inside the thunk as needed

Thunks typically dispatch plain actions using action creators, like `dispatch(increment())`

```jsx
const logAndAdd = amount => {
  return (dispatch, getState) => {
    const stateBefore = getState()
    console.log(`Counter before: ${stateBefore.counter}`)
    dispatch(incrementByAmount(amount))
    const stateAfter = getState()
    console.log(`Counter after: ${stateAfter.counter}`)
  }
}

store.dispatch(logAndAdd(5))
thunks` 一般写在 xxxSlice.js 中，但是 createSlice 不支持定义 `thunks
```

> The word "thunk" is a programming term that means ["a piece of code that does some delayed work"](https://en.wikipedia.org/wiki/Thunk).

上面的操作仍然是同步操作，接下来在 `thunks` 中写入异步操作

```
thunks` 可以接受 `setTimeout` `Promise` `async/await
```

## createAsyncThunk API

通常，异步操作用来完成 API 请求，而一次 API 请求，通常包含以下几个部分：

* 请求开始前
* 请求中
* 请求成功，得到所需要的数据
* 请求失败，可能会得到失败的提示信息

createAsyncThunk`API 可以帮你创建一个`thunk`, 它会根据`Promise`对象的结果，自动分发一个 "start/success/failure"` action

```
// xxxSlice.js
export const fetchXXX = createAsyncThunk(
	'xxx/fetchXX',
	async () => {
		... some async process
		return xxx   // 返回一个 promise 对象
	}
)
```

`createAsyncThunk` 接受两个参数：

* `action` type 字符串
* 一个返回 `promise` 对象的 `"payload creator" 回调函数`

通常情况下，在这个 `"payload creator" 回调函数` 中，会发送 AJAX 请求，我们一般使用 async/await 关键字，而不是使用 .then 链

在上面的例子中，`xxx/fetchXXX` 会作为 `action` 的前缀

当我们调用 `dispatch(fetchXXX())` 时，`thunk` 会先分发一个 `xxx/fetchXXX/pending` 的 `action`

而当 promise 对象 fulfilled 时，它就会再分发一个 `xxx/fetchXXX/fulfilled` 的 `action` ，而 promise 对象中的 value 即会当做这个 `action` 的 `payload`

### 响应对应 `action` (pending/fulfilled)

利用 `createAsyncThunk` API 就可以完成异步操作的创建，但是我们的 reducer 仍然需要响应这些 thunk 分发的 action （.../pending .../fulfilled)，这是我们需要使用 `extraReducers` 来指明这些内容

```jsx
const xxxSlice = createSlice({
  name: 'xxx',
  initialState,
  reducers: {
    // omit existing reducers here
  },
  extraReducers(builder) {
    builder
      .addCase(fetchXXX.pending, (state, action) => {
        state.status = 'loading'
				...
      })
      .addCase(fetchXXX.fulfilled, (state, action) => {
        state.status = 'succeeded'
				...
      })
      .addCase(fetchXXX.rejected, (state, action) => {
        state.status = 'failed'
				...
      })
  }
})
```

extraReducers 的另一张写法是：

```jsx
extraReducers: {
        [xxx.fulfilled]: (state, action) => {
					...
        }
    }
```

⚠️ 注意上面 xxx.fulfilled 是用 \[] 包裹起来的，表明他是一个 key

## Thunk 传入参数

在上面的例子中，`"payload creator" 回调函数` 没有接受任何参数，但实际上它可以接受两个参数：

* 第一个参数可以是任意你想要传入的一个 "payload"
*   第二个参数是一个 `thunkAPI` 对象，它包括：

    * `dispatch` and `getState`: the actual `dispatch` and `getState` methods from our Redux store. You can use these inside the thunk to dispatch more actions, or get the latest Redux store state (such as reading an updated value after another action is dispatched).
    * `extra`: the "extra argument" that can be passed into the thunk middleware when creating the store. This is typically some kind of API wrapper, such as a set of functions that know how to make API calls to your application's server and return data, so that your thunks don't have to have all the URLs and query logic directly inside.
    * `requestId`: a unique random ID value for this thunk call. Useful for tracking status of an individual request.
    * `signal`: An `AbortController.signal` function that can be used to cancel an in-progress request.
    * `rejectWithValue`: a utility that helps customize the contents of a `rejected` action if the thunk receives an error.

    > 针对 `thunkAPI` 可以利用解构赋值
