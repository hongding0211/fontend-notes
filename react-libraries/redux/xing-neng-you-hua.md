# 性能优化

## useSelector

每当一个 action 被 dispatch 后，所有的 `useSelector()` 都会被执行一次，那么所有使用了这个 hook 的组件都会重新渲染

然而事实上，一个 action 往往并不会涉及到所有 state 的修改，但因为 action 的分发，却导致不相关联的组件重新渲染，这样子的行为造成了许多不必要的开销

为了解决这一问题，我们可以检查传入的参数是否与上一次的相同，如果相同则返回和上一次相同的一个“引用”，这样就不会触发组件的刷新

Redux Toolkit 中包含一个 `createSelector` 方法，它可以用来创建 "momorized selector functions"

### createSelector
