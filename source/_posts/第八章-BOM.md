---
title: 第八章 BOM
categories:
    - JavaScript
    - 《JavaScript高级程序设计》
abbrlink: f3d3ee5b
date: 2019-05-21 14:43:43
---

ECMAScript是JavaScript的核心，但如果要在Web中使用JavaScript，那么BOM（浏览器对象模型）则无疑才是真正的核心。BOM提供了很多对象，用于访问浏览器的功能，这些功能与任何网页内容无关

# window

DOM的核心对象是`window`，它表示浏览器的一个实例。在浏览器中，`window`对象有双重角色，它既是通过JavaScript访问浏览器窗口的一个接口，又是ECMAScript规定的`Global`对象。这意味着在网页中定义的任何一个对象、变量和函数，都以`window`作为其`Global`对象。

## 全局作用域

{% note info %}
- 在全局作用域中声明的变量、函数都会变成`window`对象的属性和方法
- 抛开全局变量会成为`window`对象的属性不谈，定义全局变量与在`window`对象上直接定义属性还是有一点差别：全局变量不能通过`delete`操作符删除，而直接在`window`对象上定义的属性可以
{% endnote %}

## 窗口关系及框架

{% note info %}
- 如果页面中包含框架，那么每个框架都有自己的`window`对象，且这些`window`对象都有一个包含框架名称的`name`属性。需要注意的是，除非最外层窗口是通过`window.open()`打开的，否则其`window`对象的`name`属性不会包含任何值
- 每个`window`对象上都有一个`frames`属性，它是该框架所包含的子框架的集合，可以通过索引访问各个子框架
- `top`对象始终指向最外层框架，也就是浏览器窗口
- `parent`对象始终指向当前框架的直接上层框架
- 在没有框架的情况下，`top`等于`parent`等于`window`
- `self`对象始终指向当前的`window`，引入`self`的目的只是为了与`top`和`parent`对应起来
- 所有这些对象都是`window`对象的属性，所以也可以像这样使用`window.parent.parent.frames[0]`

---
在使用框架的情况下，浏览器中会存在多个`Global`对象。在每个框架中定义的全局变量会自动成为框架中`window`对象的属性。由于每个`window`对象都包含原生类型的构造函数，因此每个框架都有一套自己的构造函数，这些构造函数一一对应，但并不相等。例如，`top.Object`并不等于`top.frames[0].Object`。这个问题会影响到对跨框架传递的对象使用`instanceof`操作符
{% endnote %}

```html
<html>
    <head>
        <title>Frameset Example</title>
    </head>
    <frameset rows="150, *">
        <frame src="frame.htm" name="topFrame">
        <frameset cols="50%, 50%">
            <frame src="anotherframe.htm" name="leftFrame">
            <frame src="yetanotherframe.htm" name="rightFrame">
        </frameset>
    </frameset>
</html>
```

下图展示了在最高层窗口中，通过代码来访问上面例子中每个框架的不同方式
![8-1框架](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/8-1%E6%A1%86%E6%9E%B6.png)

## 窗口属性

{% note info %}
- `screenLeft`：窗口相对于视口左边的距离
- `screenTop`：窗口相对于视口上边的距离
- `innerWidth`：视口宽度
- `innerHeight`：视口高度
- `outerWidth`：浏览器宽度
- `outerHeight`：浏览器高度
{% endnote %}

## 窗口方法

{% note info %}
- `moveTo(x, y)`：将窗口移动到`(x, y)`坐标处
- `moveBy(deltaX, deltaY)`：将窗口水平和垂直方向分别移动`deltaX`和`deltaY`个像素
- `resizeTo(width, height)`：将窗口的宽度和高度调整为`width`和`height`个像素
- `resizeBy(xDelta, yDelta)`：将窗口的宽度和高度分别增加`xDelta`和`yDelta`个像素
- `open(url, windowName[, windowFeatures])`：在名称为`windowName`的窗口中加载`url`。如果`windowName`不存在，那么打开新的窗口
- `close()`：关闭通过`window.open()`打开的窗口
- `setTimeout()`
- `clearTimeout()`
- `setInterval()`
- `clearInterval()`
- `alert()`
- `confirm()`
- `promt()`
- `print()`
{% endnote %}

# location

`location`对象提供了与当前窗口中加载的文档有关的信息和一些导航功能，它既是`window`对象的属性，也是`document`对象的属性，所以`window.location === document.location`

{% note info %}
- `href`：保存当前的完整URL。如果改变该属性，页面会根据新的URL进行导航。需要注意的是，`location.href = 'aadonkeyz.com'`和`location.href = 'https://aadonkeyz.com'`的结果是不同的，前者用`'aadonkeyz.com'`替换当前URL最后的路径片段（即最后一个`/`后的内容），而后者会将页面导航至**https://aadonkeyz.com**。
- `protocol`：保存URL中的协议，包括`:`。如`https:`、`http:`。
- `host`：保存URL中的域名和端口。如`localhost:4000`、`aadonkeyz.com`。
- `hostname`：保存URL中的域名。
- `port`：保存URL中的端口。如果没有URL中没有端口号，保存空字符串。
- `pathname`：保存URL中包含`/`在内的路径信息，如`/en-US/docs/Web/API/Location`。
- `search`：保存URL中包含`?`在内的查询参数信息。如`?id=1&num=2`。
- `hash`：保存URL中包含`#`在内的片段信息。如`#location`。
- `username`：保存URL中的用户名。
- `password`：保存URL中的密码。
- `origin`（只读属性）：保存URL中的源信息，即`protoclo://domain:port`。

---
- `assign(url)`：跟直接修改`href`属性值效果一摸一样。
- `reload()`：顾名思义。
- `replace(url)`：与`assign()`唯一的不同在于，导航到新的URL后，不会在`history`对象中留下记录。
- `toString()`：返回当前的完整URL。
{% endnote %}

# navigator

`navigator`对象用于识别客户端浏览器

# screen

`screen`对象用于表明客户端的能力

# history

`history`对象保存着从窗口被打开那一刻算起的用户上网历史记录，可以把它当作一个栈，前进方向是栈顶，后退方向则相反。因为`history`是`window`对象的属性，因此每个浏览器窗口、每个标签页乃至每个框架，都有自己的`history`对象与特定的`window`对象关联。出于安全方面的考虑，开发人员无法得知用户浏览过的URL。不过，借由用户访问过的页面列表，同样可以在不知道实际URL的情况下实现后退和前进。

{% note info %}
1. `length`（只读）：保存着历史记录的数量。这个数量包括所有历史记录，即所有向后和向前的记录。对于加载到窗口、标签页或框架中的第一个页面而言，`history.length === 0`。
2. `state`（只读）：返回一个能代表历史记录中最新记录的值。

---
1. `go()`：该方法接受一个参数，这个参数可以是数值，也可以是字符串。如果是数值，则代表浏览器要前进/后退的页数（正数前进，负数后退）。如果是字符串，则代表浏览器要跳转到历史记录中包含该字符串的第一个位置（可能前进，也可能后退）。
    - `history.go(m)`：前进m页。
    - `history.go(-n)`：后退n页。
    - `history.go('wrox.com')`：跳转到历史记录中第一个出现`wrox.com`的位置，如果没有，则什么也不做。
2. `back()`：等价于`history.go(-1)`。
3. `forward()`：等价于`history.go(1)`。
4. `pushState(stateObject, title, URL)`：该方法会创建新的浏览记录并将其添加到历史记录的栈顶，浏览器的浏览状态跳转到历史记录的栈顶。此时浏览器会显示新的URL，但是并不会加载这个URL，甚至不会检查这个URL是否存在。虽然调用该方法的时候没有加载这个URL，但是在以后如果通过前进或后退再次浏览到该记录时，浏览器会加载这个记录对应的URL。
    - `stateObject`：这个参数是一个JavaScript对象，它会成为用`pushState()`创建的新浏览记录的状态对象。当用户导航到新创建的状态时，会触发popstate事件，并且此时的`history.state`是对应浏览记录的状态对象的一个拷贝。
    - `title`：Firefox目前忽略这个参数，但未来可能会用到。在此处传一个空字符串应该可以安全的防范未来这个方法的更改。或者，你可以为跳转的状态传递一个短标题。
    - `URL`：该参数定义了新的历史URL记录。注意，调用`pushState()`后浏览器并不会立即加载这个URL，但可能会在稍后某些情况下加载这个URL，比如在用户重新打开浏览器时。新URL不必须为绝对路径。如果新URL是相对路径，那么它将被作为相对于当前URL处理。新URL必须与当前URL同源，否则`pushState()`会抛出一个异常。该参数是可选的，缺省为当前URL。
5. `repalceState(stateObject, title, URL)`：与`pushState()`类似，不过它是创建新的浏览记录来替换栈顶的记录。

[**`history.pushState()`和`history.replaceState()`的参考链接**](https://developer.mozilla.org/en-US/docs/Web/API/History_API)

---
这里就不得不介绍一下popstate事件了，每当浏览器在历史记录中进行切换时（前进或后退），都会触发该事件。如果切换到的条目是由`history.pushState()`或`history.replaceState()`方法创建的，那么在事件处理程序中，`event.state`中保存着对应条目的状态对象。**调用`history.pushState()`或`history.replaceState()`方法不会触发popstate事件，只有浏览器在历史记录中进行切换时才会触发。**

最后，在这里给上[**popstate事件的参考链接**](https://developer.mozilla.org/en-US/docs/Web/API/WindowEventHandlers/onpopstate)
{% endnote %}
