---
title: FileReader
categories:
    - JavaScript
    - 杂七杂八
abbrlink: a475d371
date: 2019-06-23 08:26:04
---

# FileReader

`FileReader`对象允许Web应用程序异步读取在用户计算机上的文件或原始数据缓冲区的内容，使用`File`或`Blob`对象指定要读取的文件或数据。

`File`对象可以是来自用户在一个`<input>`元素上选择文件后返回的`FileList`对象，也可以来自拖放操作生成的`DataTransfer`对象，还可以是来自一个`HTMLCanvasElement`上执行`mozGetAsFile()`方法后返回的结果。

如果想要创建一个`FileReader`对象实例，使用构造函数直接创建即可

```js
let reader = new FileReader()
```

现在实例对象`reader`已经有了，接下来让我们看看它都有哪些属性、方法和事件处理程序
{% note info %}
属性：
- `error`：一个`DOMException`，表示在读取文件时发生的错误。
- `readyState`：表示状态的数字，`0`表示还没有加载任何数据。`1`表示数据正在加载。`2`表示已完成全部的读取请求。
- `result`：文件的内容。该属性仅在读取操作完成之后才有效，数据的格式取决于使用哪个方法来启动读取操作。

---
方法：
- `abort()`：中止读取操作。在返回时，`readyState`属性为`none`。
- `readAsArrayBuffer()`：开始读取指定的`Blob`中的内容。一旦完成，`result`属性中保存的将是被读取文件的`ArrayBuffer`数据对象。
- `readAsDataURL()`：开始读取指定的`Blob`中的内容。一旦完成，`result`属性中保存的将是一个以`data:`开头的URL格式的字符串。
- `readAsText()`：开始读取指定的`Blob`中的内容。一旦完成，`result`属性中保存的将是一个表示所读取文件内容的字符串。

---
事件处理程序：
- `onabort`：处理`abort`事件。该事件在读取操作被中断时触发。
- `onload`：处理`load`事件。该事件在读取操作完成时触发。
{% endnote %}

# 图片转base64形式的URL

```html
<input id="img-input" type="file" accept="image/*" style="display: none;" onchange="uploadImg()" />
<label for="img-input">上传图片</label>
```

```js
function uploadImg () {
    let input = document.getElementById('img-input')
    let reader = new FileReader()

    reader.onload = function () {
        document.body.style.background = `url(${this.result}) center/cover`

        // 或者使用<img>标签
        // let img = document.createElement('img')
        // img.src = this.result
        // document.body.appendChild(img)
    }
    reader.readAsDataURL(input.files[0])
}
```

