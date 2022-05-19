# XHR & Fetch

### XHR

#### 创建 XHR 对象

```js
cosnt xhr = new XMLHttpRequest()
```

#### xhr.open()

传入 `请求类型` `URL` `是否异步`（默认 true )

```js
xhr.open('get', 'https://hong97.ltd')
```

#### xhr.send()

传入`请求体`，如果没有，默认传入 `null`

```js
xhr.send(null)
```

#### readyState 属性

| 值   | 状态                 | 描述                              |
| --- | ------------------ | ------------------------------- |
| `0` | `UNSENT`           | 代理被创建，但尚未调用 open() 方法。          |
| `1` | `OPENED`           | `open()` 方法已经被调用。               |
| `2` | `HEADERS_RECEIVED` | `send()` 方法已经被调用，并且头部和状态已经可获得。  |
| `3` | `LOADING`          | 下载中； `responseText` 属性已经包含部分数据。 |
| `4` | `DONE`             | 下载操作已完成。                        |

#### 异步回调

```js
xhr.onreadystatechange = function() {
	if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
        // more actions
    }
}
```

#### 设置头部

```js
xhr.setRequestHeader('header', 'value')
```

### Fetch

#### 语法

`fetch(url: string, option: object)`

#### 与 jQuery.ajax() 的不同

* fetch 即便收到如 4xx, 5xx 的返回，也不会返回一个 rejected 的 promise；只有网络故障时，才返回 rejected 的 promise
* fetch 不跨域发送 cookie，除非设置 `credentials` 属性
