---
title: DOM
categories:
  - JavaScript
abbrlink: ee93c80b
date: 2019-04-21 11:49:47
---

DOM（文档对象模型）是针对 HTML 和 XML 文档的一个 API。DOM 描绘了一个层次化的节点树，允许开发人员添加、移除和修改页面的某一部分。

# 节点层次

DOM 可以将任何 HTML 或 XML 文档描绘成一个由多层节点构成的结构。节点分为几种不同的类型，每种类型分别表示文档中不同的信息及（或）标记。每个节点都拥有各自的特点、数据和方法，另外也与其他节点存在某种关系。节点之间的关系构成了层次，而所有页面标记则表现为一个以特定节点为根节点的树形结构。以下面的 HTML 为例：

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

可以将这个简单的 HTML 文档表示为如下层次结构：

![DOM 层次结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/DOM%20%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.png)

文档节点是每个文档的根节点。在这个例子中，文档节点只有一个子节点，即 `<html>` 元素，我们称之为文档元素。**文档元素是文档的最外层元素，文档中的其他所有元素都包含在文档元素中。每个文档只能有一个文档元素。**在 HTML 页面中，文档元素始终都是 `<html>` 元素。在 XML 中，没有预定义的元素，因此任何元素都可能成为文档元素。

每一段标记都可以通过树中的一个节点来表示。HTML 元素通过元素节点表示，特性通过特性节点表示，文档类型通过文档类型节点表示，而注释则通过注释节点表示。总共有12种节点类型，这些类型有继承自一个基类型。

# NodeList 与 HTMLCollection

{% note info %}
- `NodeList` 是节点集合，是一个类数组对象。有两种类型的 `NodeList`：
  - `Live NodeList`：它会随着 DOM 的变化而自动更新。
  - `Static NodeList`：无论 DOM 如何变化，它的值都不变。可以理解为当生成 `Static NodeList` 时，对那一时刻的 `NodeList` 作了一次浅拷贝。
- `HTMLCollection` 是元素集合，是一个类数组对象。`HTMLCollection` 是动态的，会随着 DOM 的变化而变化。
{% endnote %}

# Node 类型

DOM1 级定义了一个 `Node` 接口，该接口将由 DOM 中的所有节点类型实现。这个 `Node` 接口在 JavaScript 中是作为 `Node` 类型实现的。除了 IE 之外，在其他所有浏览器中都可以访问到这个类型。JavaScript 中的所有节点类型都继承自 `Node` 类型，因此所有节点类型都共享着相同的基本属性和方法。

{% note info %}
每个节点都有一个 `nodeType` 属性，用于表明节点的类型。节点类型有 `Node` 类型中定义的下列 12 个数值常量来表示，任何节点类型必居其一：
- `Node.ELEMENT_NODE`: 1。
- `Node.ATTRIBUTE_NODE`: 2。
- `Node.TEXT_NODE`: 3。
- `Node.CDATA_SECTION_NODE`: 4。
- `Node.ENTITY_REFERENCE_NODE`: 5。
- `Node.ENTITY_NODE`: 6。
- `Node.PROCESSING_INSTRUCTION_NODE`: 7。
- `Node.COMMENT_NODE`: 8。
- `Node.DOCUMENT_NODE`: 9。
- `Node.DOCUMENT_TYPE_NODE`: 10。
- `Node.DOCUMENT_FRAGMENT_NODE`: 11。
- `Node.NOTATION_NODE`: 12。
{% endnote %}

通过比较上面这些常量，可以很容易地确定节点的类型，例如：

```js
// 在IE中无效
if (someNode.nodeType === Node.ELEMENT_NODE) {
  console.log('Node is an element')
}
```

为了确保跨浏览器兼容，最好还是将 `nodeType` 属性与数字值进行比较，如下所示：

```js
// 适用于所有浏览器
if (someNode.nodeType === 1) {
  console.log('Node is an element')
}
```

## Node 的属性

{% note info %}
文档中所有的节点之间都存在这样或那样的关系，节点间的各种关系可以用传统的家族关系来描述，相当于把文档树比喻成家谱。每个节点都有如下属性，用于记录彼此之间的关系：
- `childNodes`：该属性可以看作一个保存着所有子节点的有序列表，它是一个 `Live NodeList` 对象。
- `firstChild`：指向 `childNodes` 列表的第一个节点，如果不存在，则为 `null`。
- `lastChild`：指向 `childNodes` 列表的最后一个节点，如果不存在，则为 `null`。
- `parentNode`：该属性指向文档中的父节点。
- `previousSibling`：指向上一个同胞节点，如果不存在，则为 `null`。
- `nextSibling`：指向下一个同胞节点，如果不存在，则为 `null`。
- `ownerDocument`：指向表示整个文档的文档节点。
- `children`：该属性可以看作一个保存着所有子元素的有序列表，它是一个 `HTMLCollection` 对象。
- `childElementCount`：返回子元素的个数。
- `firstElementChild`：指向第一个子元素。
- `lastElementChild`：指向最后一个子元素。
- `previousElementSibling`：指向前一个同辈元素。
- `nextElementSibling`：指向后一个同辈元素。
{% endnote %}

## Node 的方法

{% note info %}
因为节点的关系属性都是只读的，所以 DOM 提供了一些操作子节点的方法，：
- `appendChild()`：该方法接受单个参数：要新增的节点。操作成功后返回新增的节点。该操作会将新增的节点添加到 `childNodes` 列表的末尾。添加节点后，`childNodes` 中相关节点的关系会自动更新。
- `insertBefore()`：该方法接受两个参数：要插入的节点和作为参照的节点。操作成功后返回插入的节点。如果第二个参数是 `null`，则会将要插入的节点添加到 `childNodes` 列表的末尾。**如果省略第二个参数，会抛出错误**。
- `replaceChild()`：该方法接受两个参数：要插入的节点和要替换的节点。操作成功后返回被替换的节点。
- `removeChild()`：该方法接受单个参数：要移除的节点。操作成功后返回被移除的节点。
- `cloneNode()`：该方法接受一个布尔值参数，表示是否执行深复制。在参数为 `true` 时，执行深复制，也就是复制节点及其整个子节点树。在参数为 `false` 时，执行浅复制，即只复制节点本身。复制后返回的节点副本属于文档所有，但并没有为它指定父节点。**需要注意的是，该方法不会复制添加到 DOM 节点中的 JavaScript 属性，例如事件处理程序等**。
- `importNode()`：该方法接受两个参数：要复制的节点和一个表示是否复制子节点的布尔值。返回的结果是原来节点的副本，但能够在当前文档中使用。
- `normalize()`：该方法唯一的作用是处理文档树中的文本节点。由于解析器的实现或 DOM 操作等原因，可能会出现文本节点不包含文本，或者接连出现两个文本节点的情况。当在某个节点上调用这个方法时，就会在该节点的后代节点中查找上述两种情况。如果找到了空文本节点，则删除它。如果找到相邻的文本节点，则将它们合并为一个文本节点。

---
注意事项：
- 任何一个 DOM 节点不能同时出现在文档中的多个位置上，所以当使用 `appendChild()`、`insertBefore()` 或 `replaceChild()` 方法并且传入的第一个参数为文档树中已经存在的节点时，该节点会从原位置转移到对应的新位置。
- 如果调用 `appendChild()` 时传入的节点属于不同的文档，则会抛出错误。
- 每个节点都有一个 `ownerDocument` 属性，表示所属的文档。
{% endnote %}

# Document 类型

JavaScript 通过 `Document` 类型表示文档。在浏览器中，`document` 对象是 `HTMLDocument`（继承自 `Document` 类型）的一个实例，表示整个 HTML 页面。而且，`document` 对象是 `window` 对象的一个属性，因此可以将其作为全局对象来访问。`Document` 节点具有下列特征：

{% note info %}
- `nodeType` 的值为 `9`。
- `nodeName` 的值为 `"#document"`。
- `nodeValue` 的值为 `null`。
- `parentNode` 的值为 `null`。
- `ownerDocument` 的值为 `null`。
- 其子节点可能是一个 `DocumentType`（最多一个）、`Element`（最多一个）、`ProcessingInstruction` 或 `Comment`。
{% endnote %}

## document 的属性

{% note info %}
- `documentElement`：该属性指向 HTML 页面中的 `<html>` 元素。
- `body`：该属性指向 HTML 页面中的 `<body>` 元素。
- `doctype`：该属性指向 `<!DOCTYPE>` 标签，但是不同浏览器对该属性的支持差别很大。
- `title`：通过该属性可以取得当前页面的标题，也可以修改当前页面的标题并反映在浏览器的标题栏中。
- `URL`：该属性保存着页面完整的URL。
- `domain`：该属性保存着页面的域名。
- `referrer`：该属性保存着链接到当前页面的那个页面的 URL。在没有来源页面的情况下，该属性值可能会包含空字符串。
- `anchors`：该属性保存着文档中所有带 `name` 特性的 `<a>` 元素，是一个 `HTMLCollection` 对象。
- `applets`：该属性保存着文档中所有的 `<applet>` 元素，是一个 `HTMLCollection` 对象，因为不再推荐使用 `<applet>` 元素，所以这个集合已经不建议使用了。
- `forms`：该属性保存着文档中所有的 `<form>` 元素，是一个 `HTMLCollection` 对象，与 `document.getElementsByTagName('form')` 得到的结果相同。
- `images`：该属性保存着文档中所有的 `<img>` 元素，是一个 `HTMLCollection` 对象，与 `document.getElementsByTagName('img')` 得到的结果相同。
- `links`：该属性保存着文档中所有带 `href` 特性的 `<a>` 元素，是一个 `HTMLCollection` 对象。
---
**关于 `domain` 属性，可以将低级域改为高级域，但是不能将高级域改为低级域。**
{% endnote %}

```js
// 成功
document.domain = 'wrox.com'

// 抛出错误！
document.domain = 'p2p.wrox.com'
```

# Element 类型

除了 `Document` `类型之外，Element` 类型就要算是 Web 编程中最常用的类型了。`Element` 类型用于表现 XML 或 HTML 元素，提供了对元素签名、子节点及特性的访问。`Element` 节点具有以下特征：

{% note info %}
- `nodeType` 的值为 `1`。
- `nodeName` 的值为元素的标签名。
- `nodeValue` 的值为 `null`。
- `parentNode` 可能是 `Document` 或 `Element`。
- 其子节点可能是 `Element`、`Text`、`Comment`、`ProcessingInstruction`、`CDATASection` 或 `EntityReference`。
{% endnote %}

要访问元素的标签名，可以使用 `nodeName` 属性，也可以使用 `tagName` 属性。这两个属性会返回相同的值。以下面的元素为例：

```html
<div id="myDiv"></div>
```

可以像下面这样获得这个元素及其标签名：

```js
var div = document.getElementById('myDiv')

// DIV
console.log(div.tagName)                    

// true
console.log(div.tagName === div.nodeName)   
```

这里的元素标签名是 `div`，它拥有一个值为 `myDiv` 的 ID。可是，`div.tagName` 实际上输出的是 `"DIV"` 而非 `"div"`。**在 HTML 中，标签名始终都是以全部大写表示。而在 XML（有时候也包括 XHTML）中，标签名则始终会与源代码中的保持一致。**假如你不确定自己的脚本将会在 HTML 还是 XML 文档中执行，最好是在比较之前将标签名转换为相同的大小写形式，如下面的例子所示：

```js
if (element.tagName.toLowerCase === 'div') {
  // 在此执行某些操作
}
```

## HTML 元素的属性

所有 HTML 元素都是由 `HTMLElement` 类型表示，不是直接通过这个类型，也是通过它的子类型来表示。`HTMLElement` 类型直接继承自 `Element` 并添加了一些属性。添加的这些属性分别对应于每个 HTML 元素中都存在的下列标准特性。

{% note info %}
- `id`：元素在文档中的唯一标识符。
- `title`：有关元素的附加说明信息，一般通过工具提示条显示出来。
- `lang`：元素内容的语言代码，很少使用。
- `dir`：语言的方向，值为 `ltr` 或 `rtl`，也很少使用。
- `className`：与元素的 `class` 特性对应，即为元素指定的 CSS 类。没有将这个属性命名为 `class`，是因为 `class` 是 ECMAScript 的保留字。
{% endnote %}

上述这些属性都可以用来取得或修改相应的特性值。以下面的HTML元素为例：

```html
<div id="myDiv" class="bd" title="Body text" lang="en" dir="ltr"></div>
```

元素中指定的所有信息，都可以通过下列 JavaScript 代码取得：

```js
var div = document.getElementById('myDiv')

// myDiv
console.log(div.id)         

// bd
console.log(div.className)  

// Body text
console.log(div.title)      

// en
console.log(div.lang)       

// ltr
console.log(div.dir)        
```

当然，像下面这样通过为每个属性赋予新的值，也可以修改对应的每个特性：

```js
div.id = "someOtherId"
div.className = 'ft'
div.title = 'Some other text'
div.lang = 'fr'
div.dir = 'rtl'
```

## 操作特性

每个元素都有一或多个特性，这些特性的用途是给出相应元素或内容的附加信息。操作特性的 DOM 方法主要有三个：`getAttribute()`、`setAttribute()` 和 `removeAttribute()`。关于这三个方法的用法就不详细介绍了，下面介绍下使用它们时的注意事项：

{% note warning %}
- **传递给三个方法的特性名必须与实际的特性名相同，但是不区分大小写**。例如，如果想要得到 `class` 特性值，应传入 `class` 而不是 `className`，后者只有在通过对象属性访问特性时才用。
- 根据 HTML5 规范，自定义特性应该加上 `data-` 前缀以便验证。
- 用户自定义特性，只能通过这三个方法来操作，无法通过对象属性形式来操作。
- 有两类特殊的特性，它们虽然有对应的属性名，但属性的值与通过 `getAttribute()` 方法返回的值并不相同。它们分别是：`style` 特性和像 `onclick` 这样的事件处理程序。它们的属性值分别为对象和函数，但是通过 `getAttribute()` 方法返回的都是是字符串。
- 在使用 `setAttribute()` 为元素设置属性时，无论你传递的是什么类型的值，它都会将该值转换为字符串在设置到元素的属性上。所以对于一些特殊的属性，比如 `<input>` 标签上的 `check` 属性，只要出现了，无论你给它传递任何属性值（哪怕是空字符串也不行），它的属性值就是 `true`。只有使用 `removeAttribute()` 移除 `check` 属性，才会让该属性值变为 `false`。所以有的属性最好不要使用 `setAttribute()` 来设置。
{% endnote %}

下面的例子展示了公认特性与自定义特性的区别：

```html
<div id="myDiv" data-special="aadonkeyz"></div>
<input id="myInput"></input>
```
```js
var div = document.getElementById('myDiv')

// myDiv
console.log(div.id)                         
// myDiv
console.log(div.getAttribute('id'))             

// undefined
console.log(div['data-special'])                
// aadonkeyz
console.log(div.getAttribute('data-special'))   

div.setAttribute('class', 'myClass')
// myClass
console.log(div.className)         

div['data-other'] = 'other'
// other
console.log(div['data-other'])
// null
console.log(div.getAttribute('data-other'))
div.setAttribute('data-other', 'other')
// other 
console.log(div.getAttribute('data-other'))     

let input = document.getElementById('myInput')

input.setAttribute('checked', '')
// true
console.log(input.checked)

input.setAttribute('checked', false)
// true
console.log(input.checked)

input.removeAttribute('checked')
// false
console.log(input.checked)                      
```

## attributes

`Element` 类型是使用 `attributes` 属性的唯一一个 DOM 节点类型。`attributes` 属性中包含一个 `NamedNodeMap` 类数组对象，它是一个“动态”的集合。

{% note info %}
`attributes` 属性中包含一系列 Attr 节点，每个节点的 `nodeName` 就是特性的名称，而节点的 `nodeValue` 就是特性值。
- `getNamedItem(name)`：返回 `nodeName` 属性等于 `name` 的节点。
- `removeNamedItem(name)`：从列表中移除 `nodeName` 属性等于 `name` 的节点。
- `setNamedItem(node)`：向列表中添加节点，以节点的 `nodeName` 属性为索引。
- `item(pos)`：返回位于数字 `pos` 位置处的节点。
{% endnote %}

下面简单的演示一下 `getNamedItem()` 方法的几种使用方式：

```js
// 直接使用
var id = element.attributes.getNamedItem('id').nodeValue

// 后台自动调用 getNamedItem() 方法
var id = element.attributes['id'].nodeValue

// 先取得特性节点，然后修改它的 nodeValue
element.attributes['id'].nodeValue = 'someOtherId'
```

一般来说，由于 `attributes` 属性上的方法不够方便，开发人员更多的会使用 `getAttribute()`、`setAttribute()` 和 `removeAttribute()` 方法。不过当你想要遍历元素的特性时，`attributes` 属性倒是可以派上用场。在需要将 DOM 结构序列化为 XML 或 HTML 字符串时，多数都会涉及遍历元素特性。以下代码展示了如何迭代元素的每一个特性，然后将它们构造成 `name='value' name='value'` 这样的字符串格式。

```js
function outputAttributes (element) {
  var pairs = new Array();
  var attrName;
  var attrValue;
  var i;
  var len;
  
  for (i = 0, len = element.attributes.length; i < len; i++) {
    attrName = element.attributes[i].nodeName;
    attrValue = element.attributes[i].nodeValue;
    pairs.push(attrName + '="' + attrValue + '"');
  }

  return pairs.join(' ');
}
```

{% note warning %}
关于以上代码的运行结果，以下是两点必要的说明：
- 针对 `attributes` 中包含的特性，不同浏览器返回的顺序不同。这些特性在 XML 或 HTML 代码中出现的先后顺序，不一定与它们出现在 `attributes` 中的顺序一致。
- IE7 及更早的版本会返回 HTML 元素中所有可能的特性，包括没有指定的特性。换句话说，返回 100 多个特性的情况会很常见。
{% endnote %}

针对 IE7 及更早版本中存在的问题，可以对上面的函数加以改进，让它只返回指定的特性。每个特性节点都有一个名为 `specified` 的属性，这个属性的值如果为 `true`，则意味着要么是在 HTML 中指定了相应特性，要么是通过 `setAttribute()` 方法设置了该特性。在 IE 中，所有未设置过的特性的该属性值都为 `false`，而在其他浏览器中根本不会为这类特性生成对应的特性节点（因此，在这些浏览器中，任何特性节点的 `specified` 值始终为 `true`）。改进后的代码如下所示：

```js
function outputAttributes (element) {
  var pairs = new Array();
  var attrName;
  var attrValue;
  var i;
  var len;
    
  for (i = 0, len = element.attributes.length; i < len; i++) {
    attrName = element.attributes[i].nodeName;
    attrValue = element.attributes[i].nodeValue;
    
    if (element.attributes[i].specified) {
      pairs.push(attrName + '="' + attrValue + '"');
    }
  }

  return pairs.join(' ');
}
```

## 创建元素

使用 `document.createElement()` 方法可以创建新元素。这个方法只接受一个参数，即要创建元素的标签名。这个标签名在 HTML 文档中不区分大小写，而在 XML（包括 XHTML）文档中，则是区分大小写的。在使用该方法创建新元素的同时，也为新元素设置了 `ownerDocument` 属性。例如，使用以下代码可以创建一个 `<div>` 元素：

```js
var div = document.createElement('div')
```

在 IE 中，可以以另一种方式使用 `document.createElement()`，即为这个方法传入完整的元素标签，也可以包含属性，如下面的例子所示：

```js
var div = document.createElement('<div id="myNewDiv" class="box"></div>')
```

这种方式有助于避开在 IE7 及更早版本中动态创建元素的某些问题。**建议只在需要避开 IE7 及更早版本中存在的问题时，才使用这种方式！**

# 查找元素

{% note info %}
只能在 `document` 上调用的方法：
- `getElementById()`：输入要取得的元素 ID。如果没找到，返回 `null`。如果有多个则返回第一个。**注意，这里的 ID 必须与页面中元素的 `id` 特性严格匹配，包括大小写。**
- `getElementsByName()`：输入取得元素的 `name` 特性值。返回一个 `Live NodeList` 对象。
---
既能在 `document` 上调用，也能在 `HTMLElement` 上调用的方法：
- `getElementsByTagName()`：输入要取得元素的标签名。返回一个 `HTMLCollection` 对象。如果想要取得文档中的所有元素，可以向该方法传入 `"*"`。
- `getElementsByClassName()`：输入要取得元素的类。返回一个 `HTMLCollection` 对象。
- `querySelector()`：输入要取得元素的 css selector。如果没找到，返回 `null`。如果有多个则返回第一个。**如果输入的 css selector 不合法，会抛出错误。**
- `querySelectorAll()`：输入要取得元素的 css selector。返回一个 `Static NodeList` 对象。**如果输入的 css selector 不合法，会抛出错误。**
{% endnote %}

# 样式

## CSSStyleDeclaration

{% note info %}
`CSSStyleDeclaration` 对象代表一个 CSS 声明块。通过以下三种方式都能够访问到各自的 `CSSStyleDeclaration` 对象：
- `HTMLElement.style`：行内样式。
- `CSSStyleSheet`：样式表。
- `window.getComputedStyle`：只读的计算样式。
---
`CSSStyleDeclaration` 对象上有如下属性和方法：
- `cssText`
- `length`
- `parentRule`
- `getPropertyPriority()`
- `getPropertyValue()`
- `removeProperty()`
- `setProperty()`
{% endnote %}

## 行内样式

任何支持 `style` 特性的 HTML 元素在 JavaScript 中都有一个对应的 `style` 属性，其中保存着一个 `CSSStyleDeclaration` 对象。该对象包含通过 HTML 的 `style` 特性指定的所有样式信息，但不包含外部样式表或嵌入样式表层叠而来的样式。对于使用短划线的 CSS 属性名，必须将其转换成驼峰大小写形式，才能通过 JavaScript 来访问。

{% note warning %}
- 由于 `float` 是 JavaScript 的保留字，因此不能用于属性名。DOM2 级规范规定样式对象上相应的属性名应该是 `cssFloat`.Firefox、Safari、Opera 和 Chrome 都支持这个属性，而 IE 支持的则是 `styleFloat`。
- 在标准模式下，所有度量值都必须指定一个度量单位。在混杂模式下，可以将 `style.width` 设置为 `"20"`，浏览器会假设它是 `"20px"`。但在标准模式下，将 `style.width` 设置为 `"20"` 会导致被忽略。在实践中，最好始终都指定度量单位。
- 如果没有为元素设置 style 特性，那么 `style` 对象中可能会包含一些默认的值，但这些值并不能准确地反映该元素的样式信息。例如，在 style 特性中没有定义 `width`，而通过样式表来定义了 `width` 的话，默认值就不能准确地反映样式信息了。
{% endnote %}

```html
<div id="myDiv" style="width: 10px; font-size: 1px;"></div>
```
```js
var div = document.getElementById('myDiv')

// 10px
console.log(div.style.width)

// 1px
console.log(div.style.fontSize) 

div.style.height = '20px'
```

## 计算样式

`window.getComputedStyle(element [, pseudoElt])` 方法会解析一个元素的样式，然后返回一个包含元素所有 CSS 属性值的对象。**所有计算样式都是只读的，试图修改会抛出错误。**

```html
<div id="myDiv" style="width: 10px; font-size: 1px;"></div>
```
```js
var div = document.getElementById('myDiv')

var computedStyle = window.getComputedStyle(div)

// 10px
console.log(computedStyle.width)
```

## 样式表

### 局限性

{% note warning %}
**如果样式已经通过行内样式的方式定义了，那么怎么操作样式表也修改不了对应的样式！**
{% endnote %}

### CSSRule

`CSSRule` 对象表示样式表中的每一条规则。实际上，`CSSRule` 是一个供其他多种类型继承的基类型，其中最常见的就是 `CSSStyleRule` 类型，表示样式信息。`CSSStyleRule` 对象包含下列属性：

{% note info %}
- `cssText`：返回整条规则对应的文本。
- `parentRule`：如果当前规则是导入的规则，这个属性引用就是导入的规则，否则这个值为 `null`。
- `parentStyleSheet`：当前规则所属的样式表。
- `selectorText`：返回当前规则的选择符文本。
- `style`：一个 CSSStyleDeclaration 对象，可以通过它设置和取得规则中特定的样式值。
- `type`：表示规则类型的常量值。
{% endnote %}

### 样式表的属性和方法

`CSSStyleSheet` 类型表示的是样式表，包括通过 `<link>` 元素包含的样式表和在 `<style>` 元素中定义的样式表。`CSSStyleSheet` 继承自 `StyleSheet`，后者可以作为一个基础接口来定义非 CSS 样式表。

{% note info %}
从 `StyleSheet` 接口继承而来的属性如下，**其中除了 `disable` 属性，其他属性均为只读属性**：
- `disabled`：表示样式是否被禁用的布尔值。这个属性是可读/写的，将这个值设置为 `true` 可以禁用样式表。
- `href`：如果样式表是通过 `<link>` 包含的，则是样式表的 URL，否则为 `null`。
- `media`：当前样式表支持的所有媒体类型的集合。与所有 DOM 集合一样，这个集合也有一个 `length` 属性和一个 `item()` 方法，也可以使用方括号语法取得集合中特定的项。如果集合是空列表，表示样式表适用于所有媒体。
- `ownerNode`：指向拥有当前样式表的节点的指针，样式表可能是在 HTML 中通过 `<link>` 或 `<style>` 引入的（在 XML 中可能是通过指令引入的）。如果当前样式表是其他样式表通过 `@import` 导入的，则这个属性值为 `null`。
- `parentStyleSheet`：在当前样式表是通过 `@import` 导入的情况下，这个属性是一个指向导入它的样式表的指针。
- `title`：`ownerNode` 中 `title` 属性的值。
- `type`：表示样式表类型的字符串。对 CSS 样式表而言，这个字符串是 `"type/css"`。

---
`CSSStyleSheet` 类型还支持下列属性和方法：
- `cssRules`：样式表中包含的样式规则的集合，集合中的每一项都是一个 `CSSRule` 对象。IE 不支持这个属性，但有一个类似的 `rules` 属性。
- `ownerRules`：如果样式表是通过 `@import` 导入的，这个属性就是一个指针，指向表示导入的规则，否则为 `null`。
- `deleteRule(index)`：删除 `cssRules` 集合中指定位置的规则。
- `insertRule(rule, index)`：向 `cssRules` 集合中指定的位置插入 `rule` 字符串。
{% endnote %}

### 获取样式表

应用于文档的所有样式表是通过 `document.styleSheets` 集合来表示的。通过这个集合的 `length` 属性可以获知文档中样式表的数量，而通过方括号语法或 `item()` 方法可以访问每一个样式表。来看一个例子：

```js
var sheet = null
for (var i = 0, len = document.styleSheets.length; i < len; i++) {
  sheet = document.styleSheets[i]
  console.log(sheet)
}
```

DOM 为 `<link>` 或 `<style>` 元素定义了一个 `sheet` 属性，通过它可以直接取得对应的样式表。除了 IE，其他浏览器都支持这个属性，IE 支持的是 `styleSheet` 属性。

```js
function getStyleSheet (element) {
  return element.sheet || element.styleSheet
}

// 取得第一个<link>元素引入的样式表
var link = document.getElementByTagName('link')[0]
var sheet = getStyleSheet(link)
```

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

      // #myDiv { border: 1px solid red; }
      console.log(rule.cssText)           

      // null
      console.log(rule.parentRule)        

      // #myDiv
      console.log(rule.selectorText)    
      
      // 1
      console.log(rule.type)           
      
      // 1px solid red
      console.log(rule.style.border)      

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

# 元素大小

## 偏移量

{% note info %}
首先要介绍的属性涉及偏移量，包括元素在屏幕上占用的所有可见的空间。元素的可见大小由其高度、宽度决定，包括所有内边距、滚动条和边框大小（注意，不包括外边距）。通过下列属性可以取得元素的偏移量：
- `offsetParent`：该属性指向最近的包含该元素的定位元素。如果外层没有定位元素，则返回最近的 `<td>`、`<th>`、`<table>` 或者 `<body>`。如果元素的 `style.display` 为 `none`，则 `offsetParent` 返回 `null`。
- `offsetLeft`：元素的左外边框至 `offsetParent` 的左内边框之间的像素距离。
- `offsetTop`：元素的上外边距至 `offsetParent` 的上内边框之间的像素距离。
- `offsetHeight`：元素在垂直方向上占用的空间大小，以像素计。包括元素的高度、（可见的）水平滚动条的高度、上边框高度和下边框高度。
- `offsetWidth`：元素在水平方向上占用的空间大小，以像素计。包括元素的宽度、（可见的）垂直滚动条的宽度、左边框宽度和右边框宽度。
{% endnote %}

{% note warning %}
- `offsetLeft` 和 `offsetTop` 的值都是相对于最近的**定位元素**，也就是 `offsetParent`。
- 所有这些偏移量属性都是只读的，而且每次访问它们都需要重新计算。因此，应该尽量避免重复访问这些属性。如果需要重复使用其中某些属性的值，可以将它们保存在局部变量中，以提高性能。
{% endnote %}

![元素大小-偏移量](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E5%85%83%E7%B4%A0%E5%A4%A7%E5%B0%8F-%E5%81%8F%E7%A7%BB%E9%87%8F.png)

要想知道某个元素在页面上的偏移量，将这个元素的 `offsetLeft` 和 `offsetTop` 与其 `offsetParent` 的相同属性相加，如此循环直至根元素，就可以得到一个基本准确的值。以下两个函数就可以用于分别取得元素的左和上偏移量

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

## 客户区

{% note info %}
元素的客户区大小，指的是元素内容及其内边距所占据的空间大小。有关客户区大小的属性有两个：
- `clientWidth`：元素内容区宽度加上左右内边距宽度。
- `clientHeight`：元素内容区高度加上上下内边距高度。
- `clientLeft`：元素左边框的厚度。
- `clientTop`：元素上边框的厚度。
{% endnote %}

![元素大小-客户区](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E5%85%83%E7%B4%A0%E5%A4%A7%E5%B0%8F-%E5%AE%A2%E6%88%B7%E5%8C%BA.png)


## 滚动

{% note info %}
滚动大小指的是包含滚动内容的元素的大小。有些元素（例如 `<html>` 元素），即使没有执行任何代码也能自动添加滚动条。但另外一些元素，则需要通过 CSS 的 `overflow` 属性进行设置才能滚动。以下是与滚动大小相关的属性：
- `scrollHeight`：在没有滚动条的情况下，元素内容区高度加上上下内边距高度。
- `scrollWidth`：在没有滚动条的情况下，元素内容区宽度加上左右内边距宽度。
- `scrollLeft`：被隐藏在内容区域左侧的像素数。通过设置这个属性可以改变元素的滚动位置。
- `scrollTop`：被隐藏在内容区域上方的像素数。通过设置这个属性可以改变元素的滚动位置。
{% endnote %}

![元素大小-滚动](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E5%85%83%E7%B4%A0%E5%A4%A7%E5%B0%8F-%E6%BB%9A%E5%8A%A8.png)

## getBoundingClientRect

通过 `Element.getBoundingClientRect()` 的形式调用该方法，会返回一个对象，用于指示目标元素在页面中相对于视口的位置信息和目标元素的大小信息，该对象包含 `top`、`right`、`bottom`、`left`、`width`、`height`、`x` 和 `y`。其中不同浏览器对 `width`、`height`、`x` 和 `y`属性的支持程度不一样。

{% note warning %}
- `bottom` 和 `right` 的含义与绝对定位中的不同，请看下图。
- 当计算边界矩形时，会考虑视口区域（或其他可滚动元素）内的滚动操作，也就是说，当滚动位置发生了改变，`top`、 `right`、 `bottom` 和 `left`属性值就会立即随之发生改变。
{% endnote %}

![元素大小-getBoundingClientRect](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E5%85%83%E7%B4%A0%E5%A4%A7%E5%B0%8F-getBoundingClientRect.png)

对于不支持 `Element.getBoundingClientRect()` 方法的浏览器，可以通过其他手段取得相同的信息。一般来说，`right` 和 `left` 的差值与 `offsetWidth` 的值相等，而 `bottom` 和 `top` 的差值与 `offsetHeight` 的值相等。polyfill 代码如下：

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
```