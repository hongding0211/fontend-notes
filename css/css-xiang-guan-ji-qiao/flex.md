# Flex

### 除了之后一行其他行元素都是用 space-between

使用 `after` 伪类

```css
.xxx-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
}

.xxx-container:after {
    content: "";
    flex: auto;
}
```

![bUTLlg](https://raw.githubusercontent.com/HongDing97/imgs/main/uPic/bUTLlg.png)
