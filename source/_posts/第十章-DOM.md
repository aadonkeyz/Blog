---
title: 第十章 DOM
categories:
  - JavaScript
  - 《JavaScript高级程序设计》
abbrlink: ee93c80b
date: 2019-04-21 11:49:47
---

DOM（文档对象模型）是针对HTML和XML文档的一个API。DOM描绘了一个层次化的节点树，允许开发人员添加、移除和修改页面的某一部分。

# 节点层次

DOM可以将任何HTML或XML文档描绘成一个由多层节点构成的结构。节点分为几种不同的类型，每种类型分别表示文档中不同的信息及（或）标记。每个节点都拥有各自的特点、数据和方法，另外也与其他节点存在某种关系。节点之间的关系构成了层次，而所有页面标记则表现为一个以特定节点为根节点的树形结构。以下面的HTML为例：

```html
<html>
    <head>
        <title>Sample Page</title>
    </head>
    <body>
        <p>Hello World!</p>
    </body>
</html>
```

可以将这个简单的HTML文档表示为如下层次结构：

![10-1节点层次结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/10-1%E8%8A%82%E7%82%B9%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.png)

文档节点是每个文档的根节点。在这个例子中，文档节点只有一个子节点，即`<html>`元素，我们称之为文档元素。**文档元素是文档的最外层元素，文档中的其他所有元素都包含在文档元素中。每个文档只能有一个文档元素。**在HTML页面中，文档元素始终都是`<html>`元素。在XML中，没有预定义的元素，因此任何元素都可能成为文档元素。

每一段标记都可以通过树中的一个节点来表示；HTML元素通过元素节点表示，特性（attribute）通过特性节点表示，文档类型通过文档类型节点表示，而注释则通过注释节点表示。总共有12种节点类型，这些类型有继承自一个基类型。

## Node类型

DOM1级定义了一个Node接口，该接口将由DOM中的所有节点类型实现。这个Node接口在JavaScript中是作为Node类型实现的；除了IE之外，在其他所有浏览器中都可以访问到这个类型。JavaScript中的所有节点类型都继承自Node类型，因此所有节点类型都共享着相同的基本属性和方法。

{% note info %}
每个节点都有一个`nodeType`属性，用于表明节点的类型。节点类型有Node类型中定义的下列12个数值常量来表示，任何节点类型必居其一：
- **Node.ELEMENT_NODE(1)**；
- **Node.ATTRIBUTE_NODE(2)**；
- **Node.TEXT_NODE(3)**；
- **Node.CDATA_SECTION_NODE(4)**；
- **Node.ENTITY_REFERENCE_NODE(5)**；
- **Node.ENTITY_NODE(6)**；
- **Node.PROCESSING_INSTRUCTION_NODE(7)**；
- **Node.COMMENT_NODE(8)**；
- **Node.DOCUMENT_NODE(9)**；
- **Node.DOCUMENT_TYPE_NODE(10)**；
- **Node.DOCUMENT_PRAGMENT_NODE(11)**；
- **Node.NOTATION_NODE(12)**；
{% endnote %}

通过比较上面这些常量，可以很容易地确定节点的类型，例如：

```js
// 在IE中无效
if (someNode.nodeType === Node.ELEMENT_NODE) {
    console.log('Node is an element')
}
```

为了确保跨浏览器兼容，最好还是将`nodeType`属性与数字值进行比较，如下所示：

```js
// 适用于所有浏览器
if (someNode.nodeType === 1) {
    console.log('Node is an element')
}
```

### 节点关系属性

{% note info %}
文档中所有的节点之间都存在这样或那样的关系，节点间的各种关系可以用传统的家族关系来描述，相当于把文档树比喻成家谱。每个节点都有如下属性，用于记录彼此之间的关系：
- **childNodes**：该属性可以看作一个保存着所有子节点的有序列表；
- **firstChild**：指向`childNodes`列表的第一个节点，如果不存在，则为`null`；
- **lastChild**：指向`childNodes`列表的最后一个节点，如果不存在，则为`null`；
- **parentNode**：该属性指向文档中的父节点；
- **previousSibling**：指向上一个同胞节点，如果不存在，则为`null`；
- **nextSibling**：指向下一个同胞节点，如果不存在，则为`null`；
- **ownerDocument**：指向表示整个文档的文档节点。
{% endnote %}

关于`childNodes`属性，该属性中保存着一个`NodeList`对象，`NodeList`对象是一种类数组对象，用于保存一组有序的节点。**`NodeList`对象的独特之处在于，它实际上是基于DOM结构动态执行查询的结果，因此DOM结构的变化能够自动反映在`NodeList`对象中。**如果要访问`NodeList`对象中的元素，可以使用方括号语法或者`item()`方法：

```js
var firstChild = someNode.childNodes[0]
var secondChild = someNode.childNodes.item(1)
var count = someNode.childNodes.length
```

下图形象的展示了这些属性的含义：

![10-2节点关系属性](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/10-2%E8%8A%82%E7%82%B9%E5%85%B3%E7%B3%BB%E5%B1%9E%E6%80%A7.png)

### 操作子节点

{% note info %}
因为节点的关系属性都是只读的，所以DOM提供了一些操作子节点的方法，：
- **appendChild( )**：该方法接受单个参数：要新增的节点。操作成功后返回新增的节点。该操作会将新增的节点添加到`childNodes`列表的末尾。添加节点后，`childNodes`中相关节点的关系会自动更新；
- **insertBefore( )**：该方法接受两个参数：要插入的节点和作为参照的节点。操作成功后返回插入的节点。如果第二个参数是`null`，则会将要插入的节点添加到`childNodes`列表的末尾。如果省略第二个参数，会抛出错误；
- **replaceChild( )**：该方法接受两个参数：要插入的节点和要替换的节点。操作成功后返回被替换的节点；
- **removeChild( )**：该方法接受单个参数：要移除的节点。操作成功后返回被移除的节点。

**任何一个DOM节点不能同时出现在文档中的多个位置上，所以当使用`appendChild()`、`insertBefore()`或`replaceChild()`方法并且传入的第一个参数为文档树中已经存在的节点时，该节点会从原位置转移到对应的新位置。**
{% endnote %}

### 其他方法

{% note info %}
- **cloneNode( )**：用于创建调用这个方法的节点的一个完全相同的副本。该方法接受一个布尔值参数，表示是否执行深复制。在参数为`true`时，执行深复制，也就是复制节点及其整个子节点树；在参数为`false`时，执行浅复制，即只复制节点本身。复制后返回的节点副本属于文档所有，但并没有为它指定父节点。**需要注意的是，该方法不会复制添加到DOM节点中的JavaScript属性，例如事件处理程序等**；
- **normalize( )**：该方法唯一的作用是处理文档树中的文本节点。由于解析器的实现或DOM操作等原因，可能会出现文本节点不包含文本，或者接连出现两个文本节点的情况。当在某个节点上调用这个方法时，就会在该节点的后代节点中查找上述两种情况。如果找到了空文本节点，则删除它；如果找到相邻的文本节点，则将它们合并为一个文本节点。
{% endnote %}

## Document类型

JavaScript通过Document类型表示文档。在浏览器中，`document`对象是HTMLDocument（继承自Document类型）的一个实例，表示整个HTML页面。而且，`document`对象是`window`对象的一个属性，因此可以将其作为全局对象来访问。Document节点具有下列特征：

{% note info %}
- `nodeType`的值为9；
- `nodeName`的值为`"#document"`；
- `nodeValue`的值为`null`；
- `parentNode`的值为`null`；
- `ownerDocument`的值为`null`；
- 其子节点可能是一个DocumentType（最多一个）、Element（最多一个）、ProcessingInstruction或Comment。
{% endnote %}

### document的属性

{% note info %}
- **documentElement**：该属性指向HTML页面中的`<html>`元素；
- **body**：该属性指向HTML页面中的`<body>`元素；
- **doctype**：该属性指向`<!DOCTYPE>`标签，但是不同浏览器对该属性的支持差别很大；
- **title**：通过该属性可以取得当前页面的标题，也可以修改当前页面的标题并反映在浏览器的标题栏中；
- **URL**：该属性保存着页面完整的URL；
- **domain**：该属性保存着页面的域名；
- **referrer**：该属性保存着链接到当前页面的那个页面的URL。在没有来源页面的情况下，该属性值可能会包含空字符串。
{% endnote %}

关于`domain`属性，可以修改这个属性，但是不允许将其值设置为URL不包含的域，否则会抛出错误。且如果域名一开始是“松散的”，那么不能将它再设置为“紧绷的”，否则会抛出错误。

```js
document.domain = 'wrox.com'        // 松散的（成功）
document.domain = 'p2p.wrox.com'    // 紧绷的（抛出错误！）
```

### 查找元素

{% note info %}
- **document.getElementById( )**：该方法接受单个参数：要取得的元素ID。如果找到相应的元素则返回该元素，如果不存在则返回`null`。注意，这里的ID必须与页面中元素的`id`特性（attribute）严格匹配，包括大小写。如果页面中多个元素的ID值相同，则该方法只返回文档中第一次出现的元素。
- **document.getElementsByTagName( )**：该方法接受单个参数：要取得元素的标签名。该方法返回的是包含零或多个元素的`HTMLCollection`对象。如果想要取得文档中的所有元素，可以向该方法传入`"*"`；
- **document.getElementsByName( )**：该方法是只有HTMLDocument类型才有的方法，它接受单个参数：要取得元素的`name`特性值。该方法返回的是包含零或多个元素的`NodeList`对象（书上说的是返回`HTMLCollection`对象，MDN上说返回`NodeList`对象，不同浏览器可能实现的不同吧）。
{% endnote %}

{% note info %}
`HTMLCollection`对象与`NodeList`对象非常相似（它们都是动态的，都是类数组对象），它们之间只有两个区别：
- `HTMLCollection`对象是元素集合，而`NodeList`对象是节点集合；
- `HTMLCollection`对象多出了一个`namedItem()`方法，该方法可以通过元素的`name`特性取得集合中对应的元素，如果多个元素的`name`特性相同，则返回第一个元素。`HTMLCollection`对象支持按名称访问项，如果向方括号中传入字符串形式的索引值，在后台会自动调用`namedItem()`方法，即`elements['myImg'] === htmls.namedItem('myImg')`。
{% endnote %}

### 特殊集合

{% note info %}
除了属性和方法，`document`对象还有一些特殊的集合。这些集合都是`HTMLCollection`对象，为访问文档常用的部分提供了快捷方式，包括：
- **document.anchors**：包含文档中所有带`name`特性的`<a>`元素；
- **document.applets**：包含文档中所有的`<applet>`元素，因为不再推荐使用`<applet>`元素，所以这个集合已经不建议使用了；
- **document.forms**：包含文档中所有的`<form>`元素，与`document.getElementsByTagName('form')`得到的结果相同；
- **document.images**：包含文档中所有的`<img>`元素，与`document.getElementsByTagName('img')`得到的结果相同；
- **document.links**：包含文档中所有带`href`特性的`<a>`元素。
{% endnote %}

### 文档写入

有一个`document`对象的功能已经存在很多年了，那就是将输出流写入到网页中的能力。这个能力体现在下列4个方法中：**write( )**、**writeln( )**、**open( )**和**close( )**。其中`write()`和`writeln()`方法都接受单个字符串参数，即要写入到输出流中的文本。`write()`会原样写入，而`writeln()`则会在字符串的末尾添加一个换行符（`\n`）。在页面被加载的过程中，可以使用这两个方法向页面中动态地加入内容，如下面的例子所示：

```html
<html>
    <head>
        <title>document.write() Example</title>
    </head>
    <body>
        <p>
            The current date and time is:
            <script type="text/javascript">
                document.write('<strong>' + (new Date()).toString() + '</strong>')
            </script>
        </p>
    </body>
</html>
```

这个例子展示了在页面加载过程中输出当前日期和时间的代码。其中，日期被包含在一个`<strong>`元素中，就像在HTML页面中包含普通的文本一样。这样做会创建一个DOM元素，而且可以在将来访问该元素。通过`write()`和`writeln()`输出的任何HTML代码都将如此处理。

此外，还可以使用`write()`和`writeln()`方法动态地包含外部资源，例如JavaScript文件等。在包含JavaScript文件时，必须注意不能直接包含字符串`"</script>"`，因为这会导致该字符串被解释为脚本块的结束，它后面的代码将无法执行。为避免这个问题，可以使用`"<\/script>"`来替代。

前面的例子使用`document.write()`在页面被呈现的过程中直接向其中输出了内容。如果在文档加载结束后再调用`document.write()`，那么输出的内容将会重写整个页面，如下面的例子所示：

```html
<html>
    <head>
        <title>document.write() Example</title>
    </head>
    <body>
        <p>The is some content that you won't get to see because it will be overwritten</p>
        <script type="text/javascript">
            window.onload = function () {
                document.write('Hello World')
            }
        </script>
    </body>
</html>
```

方法`open()`和`close()`分别用于打开和关闭网页的输出流，我没有研究具体用法，就不提了。

## Element类型
## Text类型
## Comment类型
## CDATASection类型
## DocumentType类型
## DocumentFragment类型
## Attr类型

# DOM操作技术
## 动态脚本
## 动态样式
## 操作表格
## 使用Nodelist
