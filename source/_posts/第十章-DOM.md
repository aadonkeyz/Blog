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
- **Node.DOCUMENT_FRAGMENT_NODE(11)**；
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
- **childNodes**：该属性可以看作一个保存着所有子节点的有序列表。
- **children**：该属性可以看作一个保存着所有子元素的有序列表，**它是非标准的，但是绝大多数浏览器都支持**。
- **firstChild**：指向`childNodes`列表的第一个节点，如果不存在，则为`null`。
- **lastChild**：指向`childNodes`列表的最后一个节点，如果不存在，则为`null`。
- **parentNode**：该属性指向文档中的父节点。
- **previousSibling**：指向上一个同胞节点，如果不存在，则为`null`。
- **nextSibling**：指向下一个同胞节点，如果不存在，则为`null`。
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

---
注意事项：
- 任何一个DOM节点不能同时出现在文档中的多个位置上，所以当使用`appendChild()`、`insertBefore()`或`replaceChild()`方法并且传入的第一个参数为文档树中已经存在的节点时，该节点会从原位置转移到对应的新位置；
- 每个节点都有一个`ownerDocument`属性，表示所属的文档。如果调用`appendChild()`时传入的节点属于不同的文档，则会抛出错误。
{% endnote %}

### 其他方法

{% note info %}
- **cloneNode( )**：用于创建调用这个方法的节点的一个完全相同的副本。该方法接受一个布尔值参数，表示是否执行深复制。在参数为`true`时，执行深复制，也就是复制节点及其整个子节点树；在参数为`false`时，执行浅复制，即只复制节点本身。复制后返回的节点副本属于文档所有，但并没有为它指定父节点。**需要注意的是，该方法不会复制添加到DOM节点中的JavaScript属性，例如事件处理程序等**；
- **normalize( )**：该方法唯一的作用是处理文档树中的文本节点。由于解析器的实现或DOM操作等原因，可能会出现文本节点不包含文本，或者接连出现两个文本节点的情况。当在某个节点上调用这个方法时，就会在该节点的后代节点中查找上述两种情况。如果找到了空文本节点，则删除它；如果找到相邻的文本节点，则将它们合并为一个文本节点。
{% endnote %}

## Document类型

JavaScript通过Document类型表示文档。在浏览器中，`document`对象是HTMLDocument（继承自Document类型）的一个实例，表示整个HTML页面。而且，`document`对象是`window`对象的一个属性，因此可以将其作为全局对象来访问。Document节点具有下列特征：

{% note info %}
- `nodeType`的值为`9`；
- `nodeName`的值为`"#document"`；
- `nodeValue`的值为`null`；
- `parentNode`的值为`null`；
- `ownerDocument`的值为`null`；
- 其子节点可能是一个DocumentType（最多一个）、Element（最多一个）、ProcessingInstruction或Comment。
{% endnote %}

### document对象的属性

{% note info %}
- **documentElement**：该属性指向HTML页面中的`<html>`元素；
- **body**：该属性指向HTML页面中的`<body>`元素；
- **doctype**：该属性指向`<!DOCTYPE>`标签，但是不同浏览器对该属性的支持差别很大；
- **title**：通过该属性可以取得当前页面的标题，也可以修改当前页面的标题并反映在浏览器的标题栏中；
- **URL**：该属性保存着页面完整的URL；
- **domain**：该属性保存着页面的域名；
- **referrer**：该属性保存着链接到当前页面的那个页面的URL。在没有来源页面的情况下，该属性值可能会包含空字符串。

---
**关于`domain`属性，可以将低级域改为高级域，但是不能将高级域改为低级域。**
{% endnote %}

```js
document.domain = 'wrox.com'        // 成功
document.domain = 'p2p.wrox.com'    // 抛出错误！
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

除了Document类型之外，Element类型就要算是Web编程中最常用的类型了。Element类型用于表现XML或HTML元素，提供了对元素签名、子节点及特性的访问。Element节点具有以下特征：

{% note info %}
- `nodeType`的值为`1`；
- `nodeName`的值为元素的标签名；
- `nodeValue`的值为`null`；
- `parentNode`可能是Document或Element；
- 其子节点可能是Element、Text、Comment、ProcessingInstruction、CDATASection或EntityReference。
{% endnote %}

要访问元素的标签名，可以使用`nodeName`属性，也可以使用`tagName`属性；这两个属性会返回相同的值。以下面的元素为例：

```html
<div id="myDiv"></div>
```

可以像下面这样获得这个元素及其标签名：

```js
var div = document.getElementById('myDiv')
console.log(div.tagName)                    // DIV
console.log(div.tagName === div.nodeName)   // true
```

这里的元素标签名是`div`，它拥有一个值为`myDiv`的ID。可是，`div.tagName`实际上输出的是`"DIV"`而非`"div"`。**在HTML中，标签名始终都是以全部大写表示；而在XML（有时候也包括XHTML）中，标签名则始终会与源代码中的保持一致。**假如你不确定自己的脚本将会在HTML还是XML文档中执行，最好是在比较之前将标签名转换为相同的大小写形式，如下面的例子所示：

```js
if (element.tagName.toLowerCase === 'div') {
    // 在此执行某些操作
}
```

### HTML元素的属性

所有HTML元素都是由HTMLElement类型表示，不是直接通过这个类型，也是通过它的子类型来表示。HTMLElement类型直接继承自Element并添加了一些属性。添加的这些属性分别对应于每个HTML元素中都存在的下列标准特性。

{% note info %}
- **id**：元素在文档中的唯一标识符；
- **title**：有关元素的附加说明信息，一般通过工具提示条显示出来；
- **lang**：元素内容的语言代码，很少使用；
- **dir**：语言的方向，值为`ltr`或`rtl`，也很少使用；
- **className**：与元素的`class`特性对应，即为元素指定的CSS类。没有将这个属性命名为`class`，是因为`class`是ECMAScript的保留字。
{% endnote %}

上述这些属性都可以用来取得或修改相应的特性值。以下面的HTML元素为例：

```html
<div id="myDiv" class="bd" title="Body text" lang="en" dir="ltr"></div>
```

元素中指定的所有信息，都可以通过下列JavaScript代码取得：

```js
var div = document.getElementById('myDiv')
console.log(div.id)         // myDiv
console.log(div.className)  // bd
console.log(div.title)      // Body text
console.log(div.lang)       // en
console.log(div.dir)        // ltr
```

当然，像下面这样通过为每个属性赋予新的值，也可以修改对应的每个特性：

```js
div.id = "someOtherId"
div.className = 'ft'
div.title = 'Some other text'
div.lang = 'fr'
div.dir = 'rtl'
```

### 操作特性

每个元素都有一或多个特性，这些特性的用途是给出相应元素或内容的附加信息。操作特性的DOM方法主要有三个：**getAttribute( )**、**setAttribute( )**和**removeAttribute( )**。关于这三个方法的用法就不详细介绍了，下面介绍下使用它们时的注意事项：

{% note info %}
- 传递给三个方法的特性名必须与实际的特性名相同（但是不区分大小写）。例如，如果想要得到`class`特性值，应传入`class`而不是`className`，后者只有在通过对象属性访问特性时才用；
- 根据HTML5规范，自定义特性应该加上`data-`前缀以便验证；
- 只有公认的特性会以属性的形式添加到DOM对象中，而用户自定义特性则不会；
- 用户自定义特性，只能通过这三个方法来操作，无法通过对象属性形式来操作；
- 有两类特殊的特性，它们虽然有对应的属性名，但属性的值与通过`getAttribute()`方法返回的值并不相同。它们分别是：`style`特性和像`onclick`这样的事件处理程序。它们的属性值分别为对象和函数，但是通过`getAttribute()`方法返回的都是是字符串；
{% endnote %}

下面的例子展示了公认特性与自定义特性的区别：

```html
<div id="myDiv" data-special="aadonkeyz"></div>
```
```js
var div = document.getElementById('myDiv')

console.log(div.id)                             // myDiv
console.log(div.getAttribute('id'))             // myDiv
console.log(div['data-special'])                // undefined
console.log(div.getAttribute('data-special'))   // aadonkeyz

div.setAttribute('class', 'myClass')
div.setAttribute('data-other', 'other')
console.log(div.className)                      // myClass
console.log(div['data-other'])                  // undefined
console.log(div.getAttribute('data-other'))     // other
```

### attributes属性

Element类型是使用`attributes`属性的唯一一个DOM节点类型。`attributes`属性中包含一个`NamedNodeMap`类数组对象，与`NodeList`类似，也是一个“动态”的集合。元素的每一个特性都由一个Attr节点表示，每个节点都保存在`NamedNodeMap`对象中。`NamedNodeMap`对象拥有下列方法：

{% note info %}
`attributes`属性中包含一系列Attr节点，每个节点的`nodeName`就是特性的名称，而节点的`nodeValue`就是特性值。
- **getNamedItem(name)**：返回`nodeName`属性等于`name`的节点；
- **removeNamedItem(name)**：从列表中移除`nodeName`属性等于`name`的节点；
- **setNamedItem(node)**：向列表中添加节点，以节点的`nodeName`属性为索引；
- **item(pos)**：返回位于数字`pos`位置处的节点。
{% endnote %}

下面简单的演示一下`getNamedItem()`方法的几种使用方式：

```js
// 直接使用
var id = element.attributes.getNamedItem('id').nodeValue

// 后台自动调用getNamedItem()方法
var id = element.attributes['id'].nodeValue

// 先取得特性节点，然后修改它的nodeValue
element.attributes['id'].nodeValue = 'someOtherId'
```

一般来说，由于`attributes`属性上的方法不够方便，开发人员更多的会使用`getAttribute()`、`setAttribute()`和`removeAttribute()`方法。不过当你想要遍历元素的特性时，`attributes`属性倒是可以派上用场。在需要将DOM结构序列化为XML或HTML字符串时，多数都会涉及遍历元素特性。以下代码展示了如何迭代元素的每一个特性，然后将它们构造成`name='value' name='value'`这样的字符串格式。

```js
function outputAttributes (element) {
    var pairs = new Array(),
        attrName,
        attrValue,
        i,
        len;
    
    for (i = 0, len = element.attributes.length; i < len; i++) {
        attrName = element.attributes[i].nodeName
        attrValue = element.attributes[i].nodeValue
        pairs.push(attrName + '="' + attrValue + '"')
    }

    return pairs.join(' ')
}
```

{% note info %}
关于以上代码的运行结果，以下是两点必要的说明：
- 针对`attributes`中包含的特性，不同浏览器返回的顺序不同。这些特性在XML或HTML代码中出现的先后顺序，不一定与它们出现在`attributes`中的顺序一致；
- IE7及更早的版本会返回HTML元素中所有可能的特性，包括没有指定的特性。换句话说，返回100多个特性的情况会很常见。
{% endnote %}

针对IE7及更早版本中存在的问题，可以对上面的函数加以改进，让它只返回指定的特性。每个特性节点都有一个名为`specified`的属性，这个属性的值如果为`true`，则意味着要么是在HTML中指定了相应特性，要么是通过`setAttribute()`方法设置了该特性。在IE中，所有未设置过的特性的该属性值都为`false`，而在其他浏览器中根本不会为这类特性生成对应的特性节点（因此，在这些浏览器中，任何特性节点的`specified`值始终为`true`）。改进后的代码如下所示：

```js
function outputAttributes (element) {
    var pairs = new Array(),
        attrName,
        attrValue,
        i,
        len;
    
    for (i = 0, len = element.attributes.length; i < len; i++) {
        attrName = element.attributes[i].nodeName
        attrValue = element.attributes[i].nodeValue
        
        if (element.attributes[i].specified) {
            pairs.push(attrName + '="' + attrValue + '"')
        }
    }

    return pairs.join(' ')
}
```

### 创建元素

使用`document.createElement()`方法可以创建新元素。这个方法只接受一个参数，即要创建元素的标签名。这个标签名在HTML文档中不区分大小写，而在XML（包括XHTML）文档中，则是区分大小写的。在使用该方法创建新元素的同时，也为新元素设置了`ownerDocument`属性。例如，使用以下代码可以创建一个`<div>`元素：

```js
var div = document.createElement('div')
```

在IE中，可以以另一种方式使用`document.createElement()`，即为这个方法传入完整的元素标签，也可以包含属性，如下面的例子所示：

```js
var div = document.createElement('<div id="myNewDiv" class="box"></div>')
```

这种方式有助于避开在IE7及更早版本中动态创建元素的某些问题。**建议只在需要避开IE7及更早版本中存在的问题时，才使用这种方式！**

### 查找元素的子节点

{% note info %}
- `getElementsByTagName()`
- `querySelector()`
- `querySelectorAll()`
{% endnote %}

## Text类型

文本节点由Text类型表示，包含的是可以照字面量解释的纯文本内容。纯文本中可以包含转义后的HTML字符，但不能包含HTML代码。Text节点具有以下特性：

{% note info %}
- `nodeType`的值为`3`；
- `nodeName`的值为`#text`；
- `nodeValue`的值为节点所包含的文本；
- `parentNode`是一个Element；
- 不支持（没有）子节点。
{% endnote %}

可以通过`nodeValue`属性或`data`属性访问Text节点中包含的文本，这两个属性中包含的值相同。对`nodeValue`的修改也会通过`data`反映出来，反之亦然。

{% note info %}
使用下列方法可以操作节点中的文本：
- **appendData(text)**：将`text`添加到节点的末尾；
- **deleteData(offset, count)**：从`offset`指定的位置开始删除`count`个字符；
- **insertData(offset, text)**：在`offset`指定的位置插入`text`；
- **replaceData(offset, count, text)**：用`text`替换从`offset`指定的位置开始到`offset + count`为止处的文本；
- **splitText(offset)**：从`offset`指定的位置将当前文本节点分成两个文本节点；
- **substringData(offset, count)**：提取从`offset`指定的位置开始到`offset + count`为止处的字符串。

除了这些方法之外，文本节点还有一个`length`属性，保存着节点中字符的数目。而且，`nodeValue.length`和`data.length`中也保存着同样的值。
{% endnote %}

### 创建文本节点

可以使用`document.createTextNode()`方法创建新文本节点，这个方法接受一个参数：要插入节点中的文本。

一般情况下，每个元素只有一个文本子节点。不过，在某些情况下也可能包含多个文本子节点，这种情况下，相邻的文本子节点会连起来显示，中间不会有空格。

### 规范化文本节点

DOM文档中存在相邻的同胞文本节点很容易导致混乱，因为分不清哪个文本节点表示哪个字符串。另外，DOM文档中出现相邻文本节点的情况也不在少数，于是就催生了一个能够将相邻文本节点合并的方法。这个方法是由Node类型定义的（因而在所有节点类型中都存在），名叫`normalize()`。该方法在Node类型中介绍过了，此处就不介绍用法了。

### 分割文本节点

Text类型提供了一个作用与`normalize()`相反的方法：`splitText()`。这个方法会将一个文本节点分成两个文本节点，即按照指定的位置分割`nodeValue`值。原来的文本节点将包含从开始到指定位置之前的内容，新文本节点将包含剩下的文本。这个方法会返回一个新文本节点，该节点与原节点的`parentNode`相同。

## Comment类型

注释在DOM中是通过Comment类型来表示的。Comment节点具有下列特征：

{% note info %}
- `nodeType`的值为`8`；
- `nodeName`的值为`#comment`；
- `nodeValue`的值是注释的内容；
- `parentNode`可能是Document或Element；
- 不支持（没有）子节点。
{% endnote %}

Comment类型与Text类型继承自相同的基类，因此它拥有除`splitText()`之外的所有字符串操作方法。与Text类型相似，也可以通过`nodeValue`或`data`属性来取得注释的内容。

使用`document.createComment()`并为其传递注释文本也可以创建注释节点。

## CDATASection类型

CDATASection类型只针对基于XML的文档，表示的是CDATA区域。与Comment类似，CDATASection类型继承自Text类型，因此拥有除`splitText()`之外的所有字符串操作方法。CDATASection节点具有下列特征：

{% note info %}
- `nodeType`的值为`4`；
- `nodeName`的值为`#cdata-section`；
- `nodeValue`的值是CDATA区域的内容；
- `parentNode`可能是Document或Element；
- 不支持（没有）子节点。
{% endnote %}

在真正的XML文档中，可以使用`document.createCDataSection()`来创建CDATA区域，只需为其传入节点的内容即可。

## DocumentType类型

DocumentType类型在Web浏览器中并不常用，仅有Firefox、Safari和Opera支持它。DocumentType包含着与文档的`doctype`有关的所有信息，它具有下列特征：

{% note info %}
- `nodeType`的值为`10`；
- `nodeName`的值为`doctype`的名称；
- `nodeValue`的值为`null`；
- `parentNode`是Document；
- 不支持（没有）子节点。
{% endnote %}

在DOM1级中，DocumentType对象不能动态创建，而只能通过解析文档代码的方式来创建。支持它的浏览器会把DocumentType对象保存在`document.type`中。

{% note info %}
DOM1级描述了DocumentType对象的3个属性：
- **name**：表示文档类型的名称；
- **entities**：由文档类型描述的实体的`NamedNodeMap`对象；
- **notations**：由文档类型描述的符号的`NamedNodeMap`对象。
{% endnote %}

通常，浏览器中的文档使用的都是HTML或XHTML文档类型，因而`entities`和`notations`都是空列表（列表中的项来自行内文档类型声明）。但不管怎样，只有`name`属性是有用的。这个属性中保存的是文档类型的名称，也就是出现在`<!DOCTYPE`和`>`之间的文本。

## DocumentFragment类型

在所有节点类型中，只有DocumentFragment在文档中没有对应的标记。DOM规定文档片段（document fragment）是一种“轻量级”的文档，可以包含和控制节点，但不会像完整的文档那样占用额外的资源。DocumentFragment节点具有下列特征：

{% note info %}
- `nodeType`的值为`11`；
- `nodeName`的值为`#document-fragment`；
- `nodeValue`的值为`null`；
- `parentNode`是值为`null`；
- 其子节点可能是Element、Text、Comment、ProcessingInstruction、CDATASection或EntityReference。
{% endnote %}

虽然不能把文档片段直接添加到文档中，但可以将它作为一个“仓库”来使用，即可以在里面保存将来可能会添加到文档中的节点。要创建文档片段，可以使用`document.createDocumentFragment()`方法，如下所示：

```js
var fragment = document.createDocumentFragment()
```

文档片段继承了Node的所有方法，通常用于执行那些针对文档的DOM操作。如果将文档中的节点添加到文档片段中，就会从文档树中移除该节点，也不会从浏览器中再看到该节点。添加到文档片段中的新节点同样也不属于文档树。可以通过`appendChild()`或`insertBefore()`将文档片段中的内容添加到文档中。在将文档片段作为参数传递给这两个方法时，实际上只会将文档片段的所有字节点添加到相应位置上，文档片段本身永远不会称为文档树的一部分。

```html
<ul id="myList"></ul>
<style>
    var fragment = document.createDocumentFragment();
    var ul = document.getElementById('myList');
    var li = null;

    for (let i = 0; i < 3; i++) {
        li = document.createElement('li');
        li.appendChild(document.createTextNode('Item' + (i+1)));
        fragment.appendChild(li);
    }

    ul.appendChild(fragment);
</style>
```

## Attr类型

元素的特性在DOM中以Attr类型来表示。在所有浏览器中，都可以访问Attr类型的构造函数和原型。从技术角度讲，特性就是存在于元素的`attributes`属性中的节点。特性节点具有下列特征：

{% note info %}
- `nodeType`的值为`2`；
- `nodeName`的值为特性的名称；
- `nodeValue`的值为特性的值；
- `parentNode`是值为`null`；
- 在HTML中不支持（没有）子节点；
- 在XML中子节点可以是Text或EntityReference。
{% endnote %}

尽管它们也是节点，但特性却不被认为是DOM文档树的一部分。开发人员最常使用的是`getAttribute()`、`setAttribute()`和`removeAttribute()`方法，很少直接引用特性节点。

{% note info %}
Attr对象有3个属性：
- **name**：特性的名称，与`nodeName`的值相同；
- **value**：特性的值，与`nodeValue`的值相同；
- **specified**：是一个布尔值，用以区别特性是在代码中指定的，还是默认的。
{% endnote %}

使用`document.createAttribute()`并传入特性的名称可以创建新的特性节点。例如，要为元素添加`align`特性，可以使用下列代码：

```js
var attr = document.createAttribute('align')
attr.value = 'left'

element.attributes.setNamedItem(attr)
```

# DOM操作技术

## 动态脚本

创建动态脚本有两种方式：插入外部文件和直接插入JavaScript代码。

### 插入外部文件

```js
function loadScript (url) {
    var script = document.createElement('script')

    script.type = 'text/javascript'
    script.src = url

    document.body.appendChild(script)
}

loadScript('clent.js')
```

### 插入JavaScript代码

```js
function loadScriptString (code) {
    var script = document.createElement('script')

    script.type = 'text/javascript'
    try {
        script.appendChild(document.createTextNode(code))
    } catch (err) {
        script.text = code
    }

    document.body.appendChild(script)
}

loadScriptString('function sayHi () {console.log("hi")}')
```

以这种方式加载的代码会在全局作用域中执行，而且当脚本执行后将立即可用。实际上，这样执行代码与在全局作用域中把相同的字符串传递给`eval()`是一样的。

## 动态样式

能够把CSS样式包含到HTML页面中的元素有两个。其中，`<link>`元素用于包含来自外部的文件，而`<style>`元素用于指定嵌入的样式。与动态脚本类似，所谓动态样式是指在页面刚加载时不存在的样式；动态样式是在页面加载完成后动态添加到页面中的。

创建动态样式也有两种方式：通过`<link>`和通过`<style>`。

### 通过link

```js
function loadStyle (url) {
    var link = document.createElement('link')

    link.rel = 'stylesheet'
    link.type = 'text/css'
    link.href = url

    var head = document.getElementByTagName('head')[0]
    head.appendChild(link)
}

loadStyle('style.css')
```

{% note info %}
注意事项：
- 必须将`<link>`元素添加到`<head>`而不是`<body>`元素，才能保证在所有浏览器中的行为一致；
- 加载外部样式文件的过程是异步的。
{% endnote %}

### 通过style

```js
function loadStyleString (css) {
    var style = document.createElement('style')

    style.type = 'text/css'
    try {
        style.appendChild(document.createTextNode(css))
    } catch (err) {
        style.styleSheet.cssText = css
    }

    var head = document.getElementByTagName('head')[0]
    head.appendChild(link)
}

loadStyleString('body {background-color: red}')
```

## 操作表格

为了方便构建表格，HTML DOM为`<table>`、`<tbody>`和`<tr>`元素添加了一些属性和方法：

{% note info %}
为`<table>`元素添加的属性和方法：
- **caption**：保存着对`<caption>`元素（如果有）的引用；
- **tHead**：保存着对`<thead>`元素（如果有）的引用；
- **tBodies**：是一个`<tbody>`元素的`HTMLCollection`；
- **tFoot**：保存着对`<tfoot>`元素（如果有）的引用；
- **rows**：是一个表格中所有行的`HTMLCollection`；
- **createCaption( )**：创建`<caption>`元素，将其放到表格中，返回引用；
- **createTHead( )**：创建`<thead>`元素，将其放到表格中，返回引用；
- **createTFoot( )**：创建`<tfoot>`元素，将其放到表格中，返回引用；
- **deleteCaption( )**：删除`<caption>`元素；
- **deleteTHead( )**：删除`<thead>`元素；
- **deleteTFoot( )**：删除`<tfoot>`元素；
- **deleteRow(pos)**：删除指定位置的行；
- **insertRow(pos)**：向`rows`集合中的指定位置插入一行。

---
为`<tbody>`元素添加的属性和方法：
- **rows**：保存着`<tbody>`元素中行的`HTMLCollection`；
- **deleteRow(pos)**：删除指定位置的行；
- **insertRow(pos)**：向`rows`集合中的指定位置插入一行，返回对新插入行的引用。

---
为`<tr>`元素添加的属性和方法：
- **cells**：保存着`<tr>`元素中单元格的`HTMLCollection`；
- **deleteCell(pos)**：删除指定位置的单元格；
- **insertCell(pos)**：向`cells`集合中的指定位置插入一个单元格，返回对新插入单元格的引用。
{% endnote %}

想创建下面的HTML表格：

```html
<table border="1" width="100%">
    <tbody>
        <tr>
            <td>row 1, col 1</td>
            <td>row 1, col 2</td>
        </tr>
        <tr>
            <td>row 2, col 1</td>
            <td>row 2, col 2</td>
        </tr>
    </tbody>
</table>
```

创建的代码：

```js
var table = document.createElement('table')
table.border = 1
table.width = '100%'

var tbody = document.createElement('tbody')
table.appendChild(tbody)

tbody.insertRow(0)
tbody.rows[0].insertCell(0)
tbody.rows[0].cells[0].appendChild(document.createTextNode('row 1, col 1'))
tbody.rows[0].insertCell(1)
tbody.rows[0].cells[1].appendChild(document.createTextNode('row 1, col 2'))

tbody.insertRow(1)
tbody.rows[1].insertCell(0)
tbody.rows[1].cells[0].appendChild(document.createTextNode('row 2, col 1'))
tbody.rows[1].insertCell(1)
tbody.rows[1].cells[1].appendChild(document.createTextNode('row 2, col 2'))

document.body.appendChild(table)
```

## 使用Nodelist

理解`Nodelist`及其“近亲”`NamedNodeMap`和`HTMLCollection`，是从整体上透彻理解DOM的关键所在。这三个集合都是“动态的”；换句话说，每当文档结构发生变化时，它们都会得到更新。因此，它们始终都会保存着最新、最准确的信息。从本质上说，所有`Nodelist`对象都是在访问DOM文档时实时运行的查询。

例如，下列代码会导致无限循环：

```js
var divs = document.getElementsByTagName('div'),
    i,
    div;

for (i = 0; i < divs.length; i++) {
    div = document.createElement('div')
    document.body.appendChild(div)
}
```