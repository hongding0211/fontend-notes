# Immer

## draft & state

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Immer 在 produce 中会丢给你一份当前 state 的 `draft` ，你的所有修改都在 `draft` 上进行。`draft` 是对原始 state 的一层 proxy。当所有的变更完成后，Immer 根据 `draft` 上的变更，生成一个新的 `nextState`&#x20;

## produce

```javascript
produce(baseState, recipe: (draftState) => void): nextState
```

### 柯里化

```javascript
const foo1 = {
    cnt: 0,
}

const addCnt = produce((draft, n = 1) => {
    draft.cnt += n
})

const foo2 = addCnt(foo1, 2)

// foo1 {cnt: 0}
// foo2 {cnt: 2}
```

## current & original

`current` 对当前的 draft 创建一份副本，`original` 对原始 state 创建一份副本

```javascript
const foo1 = {
    cnt: 0,
}

const foo2 = produce(foo1, draft => {
    draft.cnt++

    const orignalState = origin(draft)
    const snapshot = current(draft)

    console.log(orignalState.cnt)  // 0
    console.log(snapshot.cnt)      // 1
})
```

`current` 创建的是一个不含 Proxy 的 Plain object。与直接访问 `draft` 相比，`current` 可以安全地泄露到 `produce` 外部。从另一个角度来说，`current` 可以当做**draft 的某一个阶段的一个 snapshot。**

## auto freezing

Immer 会自动地 freeze 所有 `produce` 返回的对象。这样，当你意外地尝试修改一个 state 时，就会抛出异常。从而保证了数据的 immutability。

Immer 采用的是递归 freeze，因此 当数据比较大的时候，可能会造成性能开销。

通过 `setAutoFreeze(true / false)` 来关闭 Immer 的默认 auto freeze 功能。

## 工作原理

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>绿圈 - 原始数据；蓝色圆环 - Proxy</p></figcaption></figure>

当 `producer` 开始执行的时候，只有 `draft` 的根节点的有一层 `proxy` 。一旦你往下访问了任意一个非原始数据类型（引用类型）的时候，就会自动为这个结点创建一层 `proxy` ，最终生成如上图这样的一棵 proxied tree。

当你在对 `draft` 上的某一个节点尝试修改时，Immer 会对该结点做一个浅拷贝，并且标记为 `modified` 。之后如果对该结点的任何读写操作，都不会发生在原始 state 上了。

当一个节点被标记为 `modified` 后，他的所有父节点也会被同样标记为 `modified`&#x20;

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

当整个 `producer` 完成后，Immer 遍历整颗 proxy tree，如果一个结点被标记为 `modified` ，那么就复制他的 copy；若没有，则仍然使用原始的结点。
