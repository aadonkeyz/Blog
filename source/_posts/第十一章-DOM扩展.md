---
title: 第十一章 DOM扩展
categories:
  - JavaScript
  - 《JavaScript高级程序设计》
abbrlink: 8c860f1b
date: 2019-04-23 09:16:43
---

# 选择符API

Selectors API是由W3C发起指定的一个标准，致力于让浏览器原生支持CSS查询。Selectors API Level1级的核心是两个方法：`querySelector()`和`querySelectorAll()`。在兼容的浏览器中，可以通过Document及Element类型的实例调用它们。目前已完全支持Selectors API Level1级的浏览器有IE8+、Firefox3.5+、Safari3.1+、Chrome和Opera10+。

## querySelector( )方法

该方法接受单个CSS选择符作为参数，返回与该模式匹配的第一个元素，如果没有找到匹配的元素则返回`null`。

```js
// 取得body元素
var body = document.querySelector('body')

// 取得ID为"myDiv"元素
var myDiv = document.querySelector('#myDiv')

// 取得类为"selected"的第一个元素
var selected = document.querySelector('.selected')

// 取得类为"button"的第一个图像元素
var img = document.querySelector('img.button')
```

通过Document类型调用`querySelector()`方法时，会在文档元素的范围内查找匹配的元素。而通过Element类型调用`querySelector()`方法时，只会在该元素后代元素的范围内查找匹配的元素。CSS选择符可以简单也可以复杂，视情况而定。如果传入了不被支持的选择符，`querySelector()`方法会抛出错误。

## querySelectorAll( )方法

`querySelectorAll()`方法接受的参数与`querySelector()`方法一样，都是一个CSS选择符，但返回的是所有匹配元素而不仅仅是一个元素。**这个方法返回的是一个带有`NodeList`所有属性和方法的一个快照，而非不断对文档进行搜索的动态查询。**这样实现可以避免使用`NodeList`对象通常会引起的大多数性能问题。

`querySelectorAll()`的用法类似于`querySelector()`，就不介绍了。

## matchesSelector( )方法

Selectors API Level2级规范为Element类型新增了一个方法`matchesSelector()`。这个方法接收一个参数，即CSS选择符，如果调用元素与该选择符匹配，返回`true`；否则返回`false`。

```js
if (document.body.matchesSelector('body.page1')) {
    // true
}
```

在取得某个元素引用的情况下，使用这个方法能够方便地检测它是否会被`querySelector()`或`querySelectorAll()`方法返回。

因为浏览器的支持问题，你可以使用下面的polyill代码：

```js
if (!Element.prototype.matchesSelector) {
    Element.prototype.matchesSelector = 
        Element.prototype.mozMatchesSelector ||
        Element.prototype.msMatchesSelector || 
        Element.prototype.oMatchesSelector || 
        Element.prototype.webkitMatchesSelector ||
        function (s) {
            var matches = (this.document || this.ownerDocument).querySelectorAll(s),
                i = matches.length

            while (--i >= 0 && matches.item(i) !== this) {}

            return i > -1     
        }
}
```

# 元素遍历

对于元素间的空格，IE9及之前版本不会返回文本节点，而其他所有浏览器都会返回文本节点。这样，就导致了在使用`childNodes`和`firstChild`等属性时的行为不一致。为了弥补这一差异，而同时又保持DOM规范不变，Element Traversal规范新定义了一组属性：

{% note info %}
- **childElementCount**：返回子元素（不包括文本节点和注释）的个数；
- **firstElementChild**：指向第一个子元素；
- **lastElementChild**：指向最后一个子元素；
- **previousElementSibling**：指向前一个同辈元素；
- **nextElementSibling**：指向后一个同辈元素；
{% endnote %}

# HTML5

## 与类相关的扩充

{% note info %}
- **getElementsByClassName( )**：该方法接收一个参数，即一个包含一个或多个类名的字符串，返回带有指定类的所有元素的`HTMLCollection`。传入多个类名时，类名的先后顺序不重要。该方法可以通过`document`对象及所有HTML元素调用；
- **classList**：该属性是一个只读属性，保存着一个元素的类属性的实时`DOMTokenList`集合。`DOMTokenList`有一个表示自己包含多少元素的`length`属性，而要取得每个元素可以使用方括号语法或`item()`方法；
- **classList.add(value)**：将给定的字符串值添加到`classList`中。如果值已经存在，就不添加了；
- **classList.contains(value)**：表示`classList`中是否已经存在给定的值，如果存在则返回`true`，否则返回`false`；
- **classList.remove(value)**：从`classList`中删除给定的字符串；
- **classList.toggle(value)**：如果`classList`中已经存在给定的值，删除它。如果`classList`中没有给定的值，添加它。
{% endnote %}

## 焦点管理

HTML5添加了辅助管理DOM焦点的功能。首先就是`document.activeElement`属性，这个属性始终会引用DOM中当前获得了焦点的元素。元素获得焦点的方式有页面加载、用户输入和在代码中调用`focus()`方法。

```js
var button = document.getElementById('myButton')
button.focus()
console.log(document.activeElement === button)  // true
```

**默认情况下，文档加载期间，`document.activeElement`的值为`null`。文档刚刚加载完成时，`document.activeElement`中保存的是`document.body`元素的引用。**

另外就是新增了`document.hasFocus()`方法，这个方法用于确定文档是否获得了焦点。

```js
var button = document.getElementById('myButton')
button.focus()
console.log(document.hasFocus())  // true
```

通过检测文档是否获得了焦点，可以知道用户是不是正在与页面交互。

## HTMLDocument的变化

{% note info %}
HTML5扩展了HTMLDocument，为`document`对象新增了三个属性：
- **readyState**：该属性有两个可能的值：`loading`和`complete`。通过该属性可以知道文档是否加载完成；
- **compatMode**：在标准模式下，`document.compatMode`的值等于`CSS1Compat`，而在混杂模式下，`document.compatMode`的值等于`BackCompat`；
- **head**：该属性保存着`<head>`元素的引用。
{% endnote %}

## 字符集属性

{% note info %}
HTML5新增了两个个与文档字符集有关的属性：
- **charset**：表示文档中实际使用的字符集，也可以用来指定新字符集。默认情况下，这个属性的值为`"UTF-16"`，但可以通过`<meta>`元素、响应头部或直接设置`charset`属性修改这个值；
- **defaultCharset**：表示根据默认浏览器及操作系统的设置，当前文档默认的字符集应该是什么。如果文档没有使用默认的字符集，那`charset`和`defaultCharset`属性的值可能会不一样。
{% endnote %}

## 自定义数据属性

HTML5规定可以为元素添加非标准的属性，但要添加前缀`data-`，目的是为元素提供与渲染无关的信息，或者提供语义信息。这些属性可以任意添加、随便命名，只要以`data-`开头即可。

添加了自定义属性之后，可以通过元素的`dataset`属性来访问/修改自定义属性的值。`dataset`属性的值是`DOMStringMap`的一个实例，也就是一个键值对的映射。

{% note info %}
自定义数据属性转化为`DOMStringMap`的键值对时会遵循以下规则：
- 前缀`data-`被去除（包括减号）；
- 对于每个在ASCII小写字母`a`到`z`前面的减号，会被去除，并且字母会转变成对应的大写字母；
- 其他字符（包含其他减号）都不发生变化。
{% endnote %}

```html
<div id="myDiv" data-app-id="12345" data-my-name="Nicholas"></div>
```
```js
var div = document.getElementById('myDiv')

console.log(div.dataset.appId)
console.log(div.dataset.myName)

div.dataset.appId = 23456
div.dataset.myName = 'Michael'

// 会为元素添加data-new-attr属性，其值为"new"
div.dataset.newAttr = 'new'
```

## 插入标记

虽然DOM为操作节点提供了细致入微的控制手段，但在需要给文档插入大量新HTML标记的情况下，通过DOM操作仍然非常麻烦，因为不仅要创建一系列DOM节点，而且还要小心地按照正确的顺序把他们连接起来。相对而言，使用插入标记的技术，直接插入HTML字符串不仅更简单，速度也更快。

### innerHTML属性

在读模式下，`innerHTML`属性返回与调用元素的所有子节点（包括元素、注释和文本节点）对应的HTML标记。在写模式下，`innerHTML`会根据指定的值创建新的DOM树，然后用这个DOM树完全替换调用元素原先的所有子节点。

需要注意的是，为`innerHTML`设置的值与返回的值不一定相同，如下所示：

```js
div.innerHTML = "hello & welcome,<b>\"reader\"</b>"
console.log(div.innerHTML)  // hello &amp; welcome,<b>"reader"</b>
```

{% note info %}
使用`innerHTML`的限制：
- 不可以使用`innerHTML`插入`<script>`标签；
- 并不是所有元素都支持`innerHTML`属性，不支持的元素有：`<col>`、`<colgroup>`、`<frameset>`、`<head>`、`<html>`、`<style>`、`<table>`、`<tbody>`、`<thead>`、`<tfoot>`和`<tr>`。
{% endnote %}

### outerHTML属性

在读模式下，`outerHTML`返回调用它的元素及所有子节点的HTML标签。在写模式下，`outerHTML`会根据指定的HTML字符串创建新的DOM子树，然后用这个DOM子树完全替换调用元素。

其他方面，`outerHTML`属性与`innerHTML`属性相同。

### insertAdjacentHTML( )方法

{% note info %}
插入标记的最后一个新增方式是`insertAdjacentHTML()`方法。它接收两个参数：插入位置和要插入的HTML文本。第一个参数必须是下列字符串之一：
- **beforebegin**：在当前元素之前插入一个紧邻的同辈元素；
- **afterbegin**：在当前元素的第一个子元素之前再插入新的子元素；
- **beforeend**：在当前元素的最后一个子元素之后再插入新的子元素；
- **afterend**：在当前元素之后插入一个紧邻的同辈元素。
{% endnote %}

## scrollIntoView( )方法

该方法可以在所有HTML元素上调用，通过滚动浏览器窗口或某个容器元素，调用元素就可以出现在视口中。

{% note info %}
该方法的参数比较有意思，如下所示：
1. 如果传递一个对象，该对象包含如下属性：
    - **behavior**：定义缓动动画，`"auto"`、`"instant"`或`"smooth"`，默认为`"auto"`；
    - **block**：定义垂直对齐，`"start"`、`"center"`、`"end"`或`"nearest"`，默认为`"start"`；
    - **inline**：定义水平对齐，`"start"`、`"center"`、`"end"`或`"nearest"`，默认为`"nearest"`。
2. 如果传递`true`或不传递参数，则此时相当于传递了`{block: "start", inline: "nearest"}`；
3. 如果传递`false`，则此时相当于传递了`{block: "end", inline: "nearest"}`。
{% endnote %}
