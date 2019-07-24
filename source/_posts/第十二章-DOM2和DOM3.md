---
title: 第十二章 DOM2和DOM3
categories:
    - JavaScript
    - 《JavaScript高级程序设计》
abbrlink: f431b175
date: 2019-04-23 22:23:08
---

# DOM变化

## doctype新增属性

DocumentType类型新增了3个属性：`publicId`、`systemId`和`internalSubset`。其中，前两个属性表示的是文档类型声明中的两个信息段，这两个信息段在DOM1级中是没有办法访问到的。以下面的HTML文档类型声明为例：

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
    "http://www.w3.org/TR/html4/strict.dtd">
```

`publicId`是`"-//W3C//DTD HTML 4.01//EN"`，而`systemId`是`"http://www.w3.org/TR/html4/strict.dtd"`。在支持DOM2级的浏览器中，应该可以运行下列代码：

```js
console.log(document.doctype.publicId)  // -//W3C//DTD HTML 4.01//EN
console.log(document.doctype.systemId)  // http://www.w3.org/TR/html4/strict.dtd
```

最后一个属性`internalSubset`，用于访问包含在文档类型声明中的额外定义，以下面的代码为例。

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml/strict.dtd"
    [<!ELEMENT name (#PCDATA)>]>
```

访问`document.doctype.internalSubset`将得到`"<!ELEMENT name (#PCDATA)>"`。

## Document类型的变化

### importNode( )方法

Document类型新增了`importNode()`方法，这个方法的用途是从一个文档中取得一个节点，然后将其导入到另一个文档中，使其成为这个文档结构的一部分。需要注意的是，每个节点都有一个`ownerDocument`属性，表示所属的文档。如果调用`appendChild()`时传入的节点属于不同的文档，则会导致错误。但在调用`importNode()`时传入不同文档的的节点则会返回一个新节点，这个新节点的所有权归当前文档所有。

说起来，`importNode()`方法与Element的`cloneNode()`方法非常相似，它接受两个参数：要复制的节点和一个表示是否复制子节点的布尔值。返回的结果是原来节点的副本，但能够在当前文档中使用。来看下面的例子：

```js
var newNode = document.importNode(oldNode, true)
document.body.appendChild(newNode)
```

### defaultView属性

“DOM2级视图”模块添加了一个名为`defaultView`的属性，其中保存着一个指针，指向拥有给定文档的窗口（或框架）。

# 样式

## 访问元素的样式

### 行内样式

任何支持style特性的HTML元素在JavaScript中都有一个对应的`style`属性，其中保存着CSSStyleDeclaration的实例对象。**该对象包含通过HTML的style特性指定的所有样式信息，但不包含与外部样式表或嵌入样式表经层叠而来的样式。**在style特性中指定的任何CSS属性都将表现为`style`对象的相应属性。**对于使用短划线的CSS属性名，必须将其转换成驼峰大小写形式，才能通过JavaScript来访问。**

多数情况下，都可以通过简单地转换属性名的格式来实现转换。其中一个不能直接转换的CSS属性就是`float`。由于`float`是JavaScript的保留字，因此不能用于属性名。“DOM2级”规范规定样式对象上相应的属性名应该是`cssFloat`；Firefox、Safari、Opera和Chrome都支持这个属性，而IE支持的则是`styleFloat`。

请看下面的例子：

```html
<div id="myDiv" style="width: 10px; font-size: 1px;"></div>
```
```js
var div = document.getElementById('myDiv')

console.log(div.style.width)    // 10px
console.log(div.style.fontSize) // 1px

div.style.height = '20px'
```

{% note info %}
- 在标准模式下，所有度量值都必须指定一个度量单位。在混杂模式下，可以将`style.width`设置为`"20"`，浏览器会假设它是`"20px"`；但在标准模式下，将`style.width`设置为`"20"`会导致被忽略——因为没有度量单位。在实践中，最好始终都指定度量单位；
- 如果没有为元素设置style特性，那么`style`对象中可能会包含一些默认的值，但这些值并不能准确地反映该元素的样式信息。例如，在style特性中没有定义`width`，而通过样式表来定义了`width`的话，默认值就不能准确地反映样式信息了。

---
“DOM2级样式”规范还为`style`对象定义了一些属性和方法。这些属性和方法在提供元素的style特性值的同时，也可以修改样式。下面列出了这些属性和方法：
- **cssText**：通过它能过访问到style特性中的CSS代码；
- **length**：应用给元素的CSS属性的数量；
- **item(index)**：返回给定位置的CSS属性的名称，实际上`div.style[0]`等价于`div.style.item(0)`；
- **parentRule**：表示CSS信息的CSSRule对象
- **getPropertyCSSValue(propertyName)**：返回给定属性的CSSValue对象；
- **getPropertyPriority(propertyName)**：如果给定的属性使用了`!important`设置，则返回`"important"`，否则返回空字符串；
- **getPropertyValue(propertyName)**：返回给定属性的字符串值；
- **removeProperty(propertyName)**：从样式中删除给定属性；
- **setProperty(propertyName, value, priority)**：将给定属性设置为相应的值，并加上优先权标志（`"important"`或者一个空字符串）。

---
注意事项：
- 通过`cssText`属性可以访问style特性中的CSS代码。在读取模式下，`cssText`返回浏览器对style特性中CSS代码的内部表示。在写入模式下，赋给`cssText`的值会重写整个style特性的值；也就是说，以前通过style特性指定的样式信息都将丢失；
- CSSValue对象包含`cssText`和`cssValueType`两个属性。其中，`cssText`属性的值与`getPropertyValue()`方法返回的值相同。而`cssValueType`属性则是一个数值常量，表示值的类型，`0`表示继承的值，`1`表示基本的值，`2`表示值列表，`3`表示自定义的值。
{% endnote %}

### 计算的样式

虽然`style`对象能够提供支持style特性的任何元素的样式信息，但它不包含那些从其他样式表层叠而来并影响到当前元素的样式信息。

“DOM2级样式”增强了`document.defaultView`，提供了`getComputedStyle()`方法。这个方法接收两个参数：要取得计算样式的元素和一个伪元素字符串（例如`":after"`）。如果不需要伪元素信息，第二个参数可以是`null`。`getComputedStyle()`方法返回一个CSSStyleDeclaration对象，其中包含当前元素的所有计算的样式

{% note info %}
- `window.getComputedStyle`等价于`document.defaultView.getComputedStyle`
- 所有计算的样式都是只读的，试图修改会抛出错误
{% endnote %}

```js
var div = document.getElementById('myDiv')

var computedStyle1 = document.defaultView.getComputedStyle(div, null)
var computedStyle2 = getComputedStyle(div, null)

console.log(JSON.stringify(computedStyle1) === JSON.stringify(computedStyle2)) // true
console.log(computedStyle1.width)    // 10px
```

## 操作样式表

### 局限性

{% note warning %}
**如果样式已经通过行内样式的方式定义了，那么怎么操作样式表也修改不了对应的样式！**
{% endnote %}

### 样式表的属性和方法

CSSStyleSheet类型表示的是样式表，包括通过`<link>`元素包含的样式表和在`<style>`元素中定义的样式表。CSSStyleSheet继承自StyleSheet，后者可以作为一个基础接口来定义非CSS样式表。
{% note info %}
从StyleSheet接口继承而来的属性如下，**其中除了`disable`属性，其他属性均为只读属性**：
- **disabled**：表示样式是否被禁用的布尔值。这个属性是可读/写的，将这个值设置为`true`可以禁用样式表；
- **href**：如果样式表是通过`<link>`包含的，则是样式表的URL，否则为`null`；
- **media**：当前样式表支持的所有媒体类型的集合。与所有DOM集合一样，这个集合也有一个`length`属性和一个`item()`方法，也可以使用方括号语法取得集合中特定的项。如果集合是空列表，表示样式表适用于所有媒体；
- **ownerNode**：指向拥有当前样式表的节点的指针，样式表可能是在HTML中通过`<link>`或`<style>`引入的（在XML中可能是通过指令引入的）。如果当前样式表是其他样式表通过`@import`导入的，则这个属性值为`null`；
- **parentStyleSheet**：在当前样式表是通过`@import`导入的情况下，这个属性是一个指向导入它的样式表的指针；
- **title**：`ownerNode`中`title`属性的值；
- **type**：表示样式表类型的字符串。对CSS样式表而言，这个字符串是`"type/css"`。

---
CSSStyleSheet类型还支持下列属性和方法：
- **cssRules**：样式表中包含的样式规则的集合，集合中的每一项都是一个CSSRule对象。IE不支持这个属性，但有一个类似的`rules`属性；
- **ownerRules**：如果样式表是通过`@import`导入的，这个属性就是一个指针，指向表示导入的规则，否则为`null`；
- **deleteRule(index)**：删除`cssRules`集合中指定位置的规则；
- **insertRule(rule, index)**：向`cssRules`集合中指定的位置插入`rule`字符串。
{% endnote %}

### 获取样式表

应用于文档的所有样式表是通过`document.styleSheets`集合来表示的。通过这个集合的`length`属性可以获知文档中样式表的数量，而通过方括号语法或`item()`方法可以访问每一个样式表。来看一个例子：

```js
var sheet = null
for (var i = 0, len = document.styleSheets.length; i < len; i++) {
    sheet = document.styleSheets[i]
    console.log(sheet)
}
```

DOM为`<link>`或`<style>`元素定义了一个`sheet`属性，通过它可以直接取得对应的样式表；除了IE，其他浏览器都支持这个属性，IE支持的是`styleSheet`属性。

```js
function getStyleSheet (element) {
    return element.sheet || element.styleSheet
}

// 取得第一个<link>元素引入的样式表
var link = document.getElementByTagName('link')[0]
var sheet = getStyleSheet(link)
```

### CSSRule

CSSRule对象表示样式表中的每一条规则。实际上，CSSRule是一个供其他多种类型继承的基类型，其中最常见的就是CSSStyleRule类型，表示样式信息。CSSStyleRule对象包含下列属性：

{% note info %}
- **cssText**：返回整条规则对应的文本；
- **parentRule**：如果当前规则是导入的规则，这个属性引用就是导入的规则，否则这个值为`null`；
- **parentStyleSheet**：当前规则所属的样式表；
- **selectorText**：返回当前规则的选择符文本；
- **style**：一个CSSStyleDeclaration对象，可以通过它设置和取得规则中特定的样式值；
- **type**：表示规则类型的常量值。
{% endnote %}

### 使用样式表

前面已经介绍足够多的准备知识了，现在直接举一个例子：

```html
<html>
    <head>
        <title>test</title>
        <style id="mySheet">
            #myDiv {
                border: 1px solid red;
            }
        </style>
    </head>
    <body>
        <div id="myDiv"></div>
        <script type="text/javascript">
            var sheet = document.getElementById('mySheet').sheet

            // 取得规则列表
            var rules = sheet.cssRules || sheet.rules

            // 取得第一条规则
            var rule = rules[0]

            console.log(rule.cssText)           // #myDiv { border: 1px solid red; }
            console.log(rule.parentRule)        // null
            console.log(rule.selectorText)      // #myDiv
            console.log(rule.type)              // 1
            console.log(rule.style.border)      // 1px solid red

            // 改变第一条规则中的内容
            rule.style.border = '1px solid black'
            rule.style.width = '20%'

            // 创建第二条规则，并放到样式表最前面
            sheet.insertRule('body {background: silver}', 0)
            
            // 删除旧的规则
            sheet.deleteRule(1)
        </script>
    </body>
</html>
```

## 元素大小

### 偏移量

{% note info %}
首先要介绍的属性涉及偏移量，包括元素在屏幕上占用的所有可见的空间。元素的可见大小由其高度、宽度决定，包括所有内边距、滚动条和边框大小（注意，不包括外边距）。通过下列属性可以取得元素的偏移量：
- **offsetParent**：该属性指向最近的包含该元素的**定位元素**。如果外层没有定位元素，则返回最近的`<td>`、`<th>`、`<table>`或者`<body>`。如果元素的`style.display`为`none`，则`offsetParent`返回`null`；
- **offsetLeft**：元素的左外边框至`offsetParent`的左内边框之间的像素距离；
- **offsetTop**：元素的上外边距至`offsetParent`的上内边框之间的像素距离；
- **offsetHeight**：元素在垂直方向上占用的空间大小，以像素计。包括元素的高度、（可见的）水平滚动条的高度、上边框高度和下边框高度；
- **offsetWidth**：元素在水平方向上占用的空间大小，以像素计。包括元素的宽度、（可见的）垂直滚动条的宽度、左边框宽度和右边框宽度。

{% endnote %}

{% note warning %}
- `offsetLeft`和`offsetTop`的值都是相对于最近的**定位元素**，也就是`offsetParent`。
- 所有这些偏移量属性都是只读的，而且每次访问它们都需要重新计算。因此，应该尽量避免重复访问这些属性；如果需要重复使用其中某些属性的值，可以将它们保存在局部变量中，以提高性能。
{% endnote %}

![12-1偏移量](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/12-1%E5%81%8F%E7%A7%BB%E9%87%8F.png)

要想知道某个元素在页面上的偏移量，将这个元素的`offsetLeft`和`offsetTop`与其`offsetParent`的相同属性相加，如此循环直至根元素，就可以得到一个基本准确的值。以下两个函数就可以用于分别取得元素的左和上偏移量

```js
function getElementLeft (element) {
    var actualLeft = element.offsetLeft
    var current = element.offsetParent

    while (current !== null ) {
        actualLeft += current.offsetLeft
        current = current.offsetParent
    }

    return actualLeft
}

function getElementTop (element) {
    var actualTop = element.offsetTop
    var current = element.offsetParent

    while (current !== null ) {
        actualTop += current.offsetTop
        current = current.offsetParent
    }

    return actualTop
}
```

### 客户区大小

{% note info %}
元素的客户区大小，指的是元素内容及其内边距所占据的空间大小。有关客户区大小的属性有两个：
- **clientWidth**：元素内容区宽度加上左右内边距宽度。
- **clientHeight**：元素内容区高度加上上下内边距高度。
- **clientLeft**：元素左边框的厚度。
- **clientTop**：元素上边框的厚度。
{% endnote %}

![12-2客户区大小](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/12-2%E5%AE%A2%E6%88%B7%E5%8C%BA%E5%A4%A7%E5%B0%8F.png)


### 滚动大小

{% note info %}
滚动大小指的是包含滚动内容的元素的大小。有些元素（例如`<html>`元素），即使没有执行任何代码也能自动添加滚动条；但另外一些元素，则需要通过CSS的`overflow`属性进行设置才能滚动。以下是与滚动大小相关的属性：
- **scrollHeight**：在没有滚动条的情况下，元素内容区高度加上上下内边距高度。
- **scrollWidth**：在没有滚动条的情况下，元素内容区宽度加上左右内边距宽度。
- **scrollLeft**：被隐藏在内容区域左侧的像素数。通过设置这个属性可以改变元素的滚动位置。
- **scrollTop**：被隐藏在内容区域上方的像素数。通过设置这个属性可以改变元素的滚动位置。
{% endnote %}

![12-3滚动大小](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/12-3%E6%BB%9A%E5%8A%A8%E5%A4%A7%E5%B0%8F.png)

### getBoundingClientRect( )方法

通过`Element.getBoundingClientRect()`的形式调用该方法，会返回一个对象，用于指示目标元素在页面中相对于视口的位置信息和目标元素的大小信息，该对象包含`top`、`right`、`bottom`、`left`、`width`、`height`、`x`和`y`。其中不同浏览器对`width`、`height`、`x`和`y`属性的支持程度不一样。

{% note warning %}
**当计算边界矩形时，会考虑视口区域（或其他可滚动元素）内的滚动操作，也就是说，当滚动位置发生了改变，`top`和`left`属性值就会立即随之发生改变（因此，它们的值是相对于视口的，而不是绝对的）。如果你需要获得相对于整个网页左上角定位的属性值，那么只要给`top`和`left`属性值加上当前的滚动位置（通过`window.scrollX`和`window.scrollY`），这样就可以获取与当前的滚动位置无关的值**
{% endnote %}

![12-4元素大小](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/12-4%E5%85%83%E7%B4%A0%E5%A4%A7%E5%B0%8F.png)

对于不支持`Element.getBoundingClientRect()`方法的浏览器，可以通过其他手段取得相同的信息。一般来说，`right`和`left`的差值与`offsetWidth`的值相等，而`bottom`和`top`的差值与`offsetHeight`的值相等。polyfill代码如下：

```js
function getBoundingClientRect (element) {
    if (element.getBoundingClientRect) {
        return element.getBoundingClientRect()
    } else {
        var scrollTop = document.documentElement.scrollTop
        var scrollLeft = document.documentElement.scrollLeft

        var actualTop = getElementTop(element)
        var actualLeft = getElementLeft(element)

        return {
            top: actualTop - scrollTop,
            bottom: actualTop - scrollTop + element.offsetHeight,
            left: actualLeft - scrollLeft,
            right: actualLeft - scrollLeft + element.offsetWidth
        }
    }
}

function getElementLeft (element) {
    var actualLeft = element.offsetLeft
    var current = element.offsetParent

    while (current !== null ) {
        actualLeft += current.offsetLeft
        current = current.offsetParent
    }

    return actualLeft
}

function getElementTop (element) {
    var actualTop = element.offsetTop
    var current = element.offsetParent

    while (current !== null ) {
        actualTop += current.offsetTop
        current = current.offsetParent
    }

    return actualTop
}
```

# 遍历

“DOM2级遍历和范围”模块定义了两个用于辅助完成顺序遍历DOM结构的类型：NodeIterator和TreeWalker。这两个类型能够基于给定的起点对DOM结构执行**深度优先**的遍历操作。

## NodeIterator

{% note info %}
NodeIterator类型是两者中比较简单的一个，可以使用`document.createNodeIterator()`方法创建它的新实例。这个方法接受下列参数：
- **root**：作为遍历起点的节点；
- **whatToShow**：要访问的节点类型的数字代码；
- **filter（可选）**：该参数代表过滤器，可以是一个函数，一个包含`acceptNode()`方法的对象或者`null`。如果`filter`参数是一个函数或者一个包含`acceptNode()`方法的对象，那么当你想要访问给定的节点时，让函数/方法返回`NodeFilter.FILTER_ACCEPT`。否则返回`NodeFilter.FILTER_SKIP`或`NodeFilter.FILTER_REJECT`。

---
`whatToShow`参数是一个位掩码，参数的值以常量形式在NodeFilter类型中定义，如下所示：
- **NodeFilter.SHOW_ALL**：对应`-1`，代表所有类型的节点
- **NodeFilter.SHOW_ELEMENT**：对应`1`，代表元素节点；
- **NodeFilter.SHOW_TEXT**：对应`4`，代表文本节点；
- **NodeFilter.SHOW_PROCESSING_INSTRUCTION**：对应`64`，代表处理指令节点；
- **NodeFilter.SHOW_COMMENT**：对应`128`，代表注释节点；
- **NodeFilter.SHOW_DOCUMENT**：对应`256`，代表文档节点；
- **NodeFilter.SHOW_DOCUMENT_TYPE**：对应`512`，代表文档类型节点；
- **NodeFilter.SHOW_DOCUMENT_FRAGMENT**：对应`1024`，代表文档片段节点。

除了`NodeFilter.SHOW_ALL`之外，可以使用按位或操作符来组合多个选项，例如：`NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT`。

---
在NodeIterator中，不管`filter`返回的是`NodeFilter.FILTER_SKIP`还是`NodeFilter.FILTER_REJECT`，遍历都会只跳过当前节点，但还会对其子节点进行访问。

---
NodeIterator类型的两个主要方法是`nextNode()`和`previousNode()`，这两个方法的作用与名字一致，它们会返回当前遍历到的节点。在根节点调用`previousNode()`方法和在最后一个节点处调用`nextNode()`都会返回`null`。

由于`nextNode()`和`previousNode()`方法都基于NodeIterator在DOM结构中的内部指针工作，所以DOM结构的变化会反映在遍历的结果中。
{% endnote %}

基础概念介绍完毕，上代码：

```html
<html>
    <head>
        <title>test</title>
    </head>
    <body>
        <div id="myDiv">
            <p><b>Hello</b> world!</p>
            <ul>
                <li>List item 1</li>
                <li>List item 2</li>
                <li>List item 3</li>
            </ul>
        </div>
        <script type="text/javascript">
            var div = document.getElementById('myDiv')

            var iterator = document.createNodeIterator(div, NodeFilter.SHOW_ALL, null)

            var currentNode = iterator.nextNode()
            while (currentNode !== null) {
                console.log(currentNode)
                currentNode = iterator.nextNode()
            }

            console.log('===========================================')

            var filter = function (node) {
                return node.nodeType === 1 ?
                    NodeFilter.FILTER_ACCEPT :
                    NodeFilter.FILTER_SKIP
            }
            // 两个filter是等价的
            // var filter = {
            //     acceptNode: function (node) {
            //         return node.nodeType === 1 ?
            //             NodeFilter.FILTER_ACCEPT :
            //             NodeFilter.FILTER_SKIP
            //     }
            // }

            iterator = document.createNodeIterator(div, NodeFilter.SHOW_ALL, filter)

            currentNode = iterator.nextNode()
            while (currentNode !== null) {
                console.log(currentNode)
                currentNode = iterator.nextNode()
            }
        </script>
    </body>
</html>
```

## TreeWalker

{% note info %}
TreeWalker是NodeIterator的一个更高级的版本，可以使用`document.createTreeWalker()`方法创建它的新实例。除了包括`nextNode()`和`previousNode()`在内的相同的功能之外，这个类型还提供下列用于在不同方向上遍历DOM结构的方法：
- **parentNode( )**：遍历到当前的父节点；
- **firstChild( )**：遍历到当前节点的第一个子节点；
- **lastChild( )**：遍历到当前节点的最后一个子节点；
- **nextSibling( )**：遍历到当前节点的下一个同辈节点；
- **previousSibling( )**：遍历到当前节点的上一个同辈节点。

---
在TreeWalker中，如果`filter`返回`NodeFilter.FILTER_SKIP`，遍历会跳过当前的节点，但还会对子节点进行访问。如果返回`NodeFilter.FILTER_REJECT`则会跳过当前节点及该节点的整个子树。
{% endnote %}

<!-- # 范围

为了让开发人员更方便地控制页面，“DOM2级遍历和范围”模块定义了“范围”（range）接口。通过范围可以选择文档中的一个区域，而不必考虑节点的界限。比如，在富文本编辑器中经常使用这个接口。

## DOM中的范围

DOM2级在Document类型中定义了`createRange()`方法。在兼容DOM的浏览器中，这个方法属于`document`对象。

与节点类似，新创建的范围也直接与创建它的文档关联在一起，不能用于其他文档。创建了范围之后，接下来就可以使用它在后台选择文档中的特定部分。而创建范围并设置了其位置之后，还可以针对范围内的内容执行很多种操作，从而实现对底层DOM树的更精细的控制。

{% note info %}
每个范围由一个Range类型的实例表示，这个实例拥有很多属性和方法。下列属性提供了当前范围在文档中的位置信息：
- **startContainer**：包含范围起点的节点；
- **startOffset**：范围起点在`startContainer`中的偏移量；
- **endContainer**：包含范围终点的节点；
- **endOffset**：范围终点在`endContainer`中的偏移量；
- **collapsed**：一个用于判断范围起始位置和终止位置是否相同的布尔值；
- **commonAncestorContainer**：包含`startContainer`和`endContainer`最深的节点。
{% endnote %}

### 用范围实现选择

{% note info %}
要使用范围来选择文档中的一部分，可以使用下列方法：
- **setStart(refNode, offset)**
- **setEnd(refNode, offset)**
- **setStartBefore(refNode)**
- **setStartAfter(refNode)**
- **setEndBefore(refNode)**
- **setEndAfter(refNode)**
- **selectNode(refNode)**
- **selectNodeContents(refNode)**
{% endnote %} -->

