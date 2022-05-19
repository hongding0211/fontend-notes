# DOM 操作

### 节点类型

常用的 DOM 节点类型包括:

* Document
* Element
* Text
* DocumentType
* Attr

#### 判断某一节点类型

```js
document.nodeType === Node.DOCUMENT_NODE	// true
```

#### Node 常见属性

* nodeName
* nodeValue

> Element Node 的 nodeName 是标签名，而 nodeValue 为 null

### 节点关系

* childNodes
* parentNode
* previousSibling, nextSibling
  * 第一个元素的 previousSibling、最后一个的 nextSibling 都指向 null
* firstChild, lastChild

#### childNodes

`childNodes()` 方法返回一个 `NodeList` 实例，注意这是一个**类数组**对象，可以用索引访问，也有 `length` 属性。

> 但是 NodeList 实例并不是 Array 实例，他们只是在使用上比较类似。

### Document Node

`document` 常见属性

* doucumentElement： 的引用
* body: 的引用
* title：文档标题
* URL（只读）
* referrer（只读）
*   domain

    只有 domain 是可以修改的，可以将域名设置成上级域名，但不能设置子域名，另外不能设置不同的域名

    ```js
    // 页面来自 me.hong97.ltd
    document.domain = 'hong97.ltd'	// ok
    document.domain = 'baidu.com'	// not ok
    ```

### Element Node

element node 是 HTML 中最常用的 Node。

可以通过 `nodeName` 或者 `tagName` 来获取标签名。

#### 操作属性

三种操作属性的方法：

* getAttribute()
* setAttribute()
* removeAttribute()

1.  除了使用 `getAttribute()` 来获取属性外，Element Node 对象还暴露了许多属性提供直接访问。如：id, style, onClick...

    > getAttribute() 返回的是字符串，而直接访问某些属性可以返回对象
    >
    > 如 .onClick 返回的是函数，而使用 getAttribute('onClick') 返回的是字符串
    >
    > 因此，getAttribute() 常用来获取自定义属性
2.  element node 有一个 `attributes` 属性，他是一个 `NamedNodeMap` 实例，类似于 NodeList 实例，可以通过索引或者 key 访问

    ```js
    const ul = document.getElementById('list')
    ul.attributes['id'] = 'list'
    ```

    > 一般来说，使用上面三种 API 操作属性更加常见一些

### Text Node

```html
<h1 id="title"></h1>
<script>
    const text = document.createTextNode('Hello World')
	document.getElementById('title').appendChild(text)
</script>
```

### 常用 DOM 操作

#### 获取元素

* getElementByID()
* getElementsByTagName()
* getElementsByClassName()

> 类似于 getElementsByTagName() 这样的 API，他返回的不是“数组”，而是一个 HTML Collection，类似于 NodeList ，是一个类数组行为的对象。

#### 创建元素

`document.createElement()`

```js
const div = document.createElement('div')	// 传入表情名
```

#### 操作元素

*   `appandChild()`，在某个元素的 childNodes 末尾附上一个元素

    ```js
    // 在 ul 标签内追加一个 li 标签
    document.getElementsByTagName('ul')[0].appendChild(li)
    ```
*   `insertBefore()`

    ```js
    const insertedNode = parentNode.insertBefore(newNode, referenceNode);
    ```

    > 如果参考节点为 null，则效果和 appendChild() 相同
    >
    > 另外，使用`appendChild()`, `insertBefore()` 时，如果插入的结点已经存在，那么会将结点移动至新的位置。
*   `replaceChild()`

    ```js
    parentNode.replaceChild(newChild, oldChild);
    ```

    > 如果 newChild 存在，那么它会从原始位置中删除
*   `cloneNode()`

    ```js
    dupNode = node.cloneNode(deep)
    ```

    > 传入的 deep 参数，如果为 true，表示深克隆（递归克隆）
*   `removeChild()`

    ```js
    const oldChild = parentNode.removeChild(childNode);
    ```

### DOM 扩展

#### querySelector()

返回匹配到的第一个元素，没有匹配返回 null

#### quesrySelectorAll()

返回所有匹配到的节点（NodeList 实例），没有匹配返回一个空实例。

#### classList

classList 属性是一个 `DomTokenList` 实例，它有下面四种方法：

* add(value)
* remove(value)
* toogle(value)
* contains(value)

#### innerHTML

读取 innerHTML 会以字符串形式返回所有后代元素；而写入时，会将字符串解析成 DOM 子树

> 现代浏览器都不会解析 innerHTML 中的 标签

#### outerHTML

返回包括调用它的元素及其后代元素

```html
<ul>
    <li>1</li>
<li>2</li>
<li>3</li>
</ul>

<script>
    // <ul>
    //     <li>1</li>
    //     <li>2</li>
    //     <li>3</li>
    // </ul>
    console.log(document.querySelector('ul').outerHTML)
</script>
```

#### innerText

读取时，以深度优先的顺序递归地将所有子元素的 Text Nodes 拼接成一个字符串并返回。

写入时，移除所有后代节点，并且将部分标签(如**)进行 HTML 编码，是他完全成为一个 Text Node**
