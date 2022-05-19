# Events 事件

## 事件冒泡

一个事件触发后，他会从底层一层一层向上传递

![](../.gitbook/assets/image-20220406160016294.png)

## 事件捕获

事件捕获从上层向下

![](../.gitbook/assets/image-20220406161303500.png)

## addEventListener

addEventListener 有三个参数：

* 事件名称；
* 事件处理函数；
* 冒泡(true)还是捕获(false)
