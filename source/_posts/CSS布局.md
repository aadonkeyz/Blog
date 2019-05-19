---
title: CSS布局
categories:
    - HTML&CSS
abbrlink: ee1ff2c0
date: 2019-05-13 16:06:51
---

# 盒模型

## 理解盒模型

{% note info %}
盒模型由四个部分组成：
- **内容区**
- **内边距**
- **边框**
- **外边距**
{% endnote %}

![理解盒模型](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E7%90%86%E8%A7%A3%E7%9B%92%E6%A8%A1%E5%9E%8B.png)

## 盒模型和元素的大小

{% note info %}
1. 盒模型的大小和元素的大小不是一回事
2. 盒模型的大小是`外边距+边框+内边距+内容区`
3. 元素的大小取决于`box-sizing`属性，它有两个值：
    - `content-box`（默认）：此时元素的`width`和`height`属性设置的是`内容区`的宽度和高度
    - `border-box`：此时元素的`width`和`height`属性设置的是`边框+内边距+内容区`的宽度和高度
4. 块级元素在没有设置`width`时，它的盒模型会始终填满其父元素的宽度。
{% endnote %}

## border-radius

...未完待续

## 盒阴影

...未完待续

# 浮动

## float的初衷

首先介绍引入`float`属性的初衷：为了实现文字环绕图片的效果，就像下面的例子：

```html
<div style="width: 500px; border: 1px solid black;">
    <img src="https://mdn.mozillademos.org/files/13340/butterfly.jpg" alt="A pretty butterfly with red, white, and brown coloring, sitting on a large leaf" style="float: left">

    <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci vel, viverra egestas ligula. Curabitur vehicula tellus neque, ac ornare ex malesuada et. In vitae convallis lacus. Aliquam erat volutpat. Suspendisse ac imperdiet turpis. Aenean finibus sollicitudin eros pharetra congue. Duis ornare egestas augue ut luctus. Proin blandit quam nec lacus varius commodo et a urna. Ut id ornare felis, eget fermentum sapien.</p>
</div>
```

![float的初衷](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/float%E7%9A%84%E5%88%9D%E8%A1%B7.png)

## 利用float布局

{% note info %}
在此之后，人们开始利用`float`属性进行布局，不过需要了解如下内容：
- 一定要为浮动元素设置宽度；
- 当前一个元素非浮动，后一个元素浮动时，浮动元素在原水平位置向左或向右浮动；
- 当前一个元素浮动，后一个元素也浮动时，后一个元素会在宽度允许的条件下与前一个元素挤在同一行；
- 当前一个元素浮动，后一个元素不浮动时，根据元素`display`属性的不同会出现不同的结果，这里就不介绍了。
{% endnote %}

## 围住浮动元素

浮动元素脱离了文档流，其父元素也看不到它了，因而也不会包围它。

```html
<div style="width: 500px; border: 1px solid black;">
    <img src="https://mdn.mozillademos.org/files/13340/butterfly.jpg" alt="A pretty butterfly with red, white, and brown coloring, sitting on a large leaf" style="float: left">

    <p>It is fun to float.</p>
</div>
<footer style="width: 500px; border: 1px solid black;">Here is the footer element runs across the bottom of the page.</footer>
```

![父元素围不住float子元素](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E7%88%B6%E5%85%83%E7%B4%A0%E5%9B%B4%E4%B8%8D%E4%BD%8Ffloat%E5%AD%90%E5%85%83%E7%B4%A0.png)

这个效果明显不是我们实际想要的，为了解决这个问题，介绍下面几种让父元素围住子元素的方法。

下面几种方案的效果都是一致的，如下所示：

![父元素围住float子元素](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E7%88%B6%E5%85%83%E7%B4%A0%E5%9B%B4%E4%BD%8Ffloat%E5%AD%90%E5%85%83%E7%B4%A0.png)

### 利用overflow

```html
<div style="width: 500px; border: 1px solid black; overflow: hidden;">
    <img src="https://mdn.mozillademos.org/files/13340/butterfly.jpg" alt="A pretty butterfly with red, white, and brown coloring, sitting on a large leaf" style="float: left">

    <p>It is fun to float.</p>
</div>
<footer style="width: 500px; border: 1px solid black;">Here is the footer element runs across the bottom of the page.</footer>
```

### 同时浮动父元素

```html
<div style="width: 500px; border: 1px solid black; float: left;">
    <img src="https://mdn.mozillademos.org/files/13340/butterfly.jpg" alt="A pretty butterfly with red, white, and brown coloring, sitting on a large leaf" style="float: left">

    <p>It is fun to float.</p>
</div>
<footer style="width: 500px; border: 1px solid black;">Here is the footer element runs across the bottom of the page.</footer>
```

### 利用clear

{% note info %}
- `clear`属性的值分别为：`left`、`right`和`both`；
- `clear`属性指定一个元素是否必须移动（清除浮动后）到它之前的浮动元素下方。
{% endnote %}

```html
<div style="width: 500px; border: 1px solid black;">
    <img src="https://mdn.mozillademos.org/files/13340/butterfly.jpg" alt="A pretty butterfly with red, white, and brown coloring, sitting on a large leaf" style="float: left">

    <p>It is fun to float.</p>
</div>
<footer style="width: 500px; border: 1px solid black;">Here is the footer element runs across the bottom of the page.</footer>

<style type="text/css">
div::after {
    content: '.';
    display: block;
    height: 0;
    visibility: hidden;
    clear: both;
}
</style>
```

# 定位

{% note info %}
- `position`属性有四个可选值：`static`、`relative`、`absolute`和`fixed`，默认值为`static`
- 只有当`position`属性为`relative`、`absolute`或`fixed`时，`top`、`right`、`bottom`和`left`才会其作用，**且它们设置的是盒模型的位置而不是元素的位置**
{% endnote %}

## 静态定位

```html
<p>First Paragraph</p>
<p>Second Paragraph</p>
<p id="specialpara">Third Paragraph (with ID)</p>
<p>Fourth Paragraph</p>

<style type="text/css">
p {
    border: 1px solid black;
    width: 200px;
}

p#specialpara {
    position: static
}
</style>
```

![静态定位](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E9%9D%99%E6%80%81%E5%AE%9A%E4%BD%8D.png)

## 相对定位

{% note info %}
- 相对定位是元素相对于它原来在文档流中的位置定位；
- 可以使用`top`、`right`、`bottom`和`left`来调整元素的位置；
- 相对定位的元素仍然处于文档流中，它原来的位置会被保留。
{% endnote %}

```html
<p>First Paragraph</p>
<p>Second Paragraph</p>
<p id="specialpara">Third Paragraph (with ID)</p>
<p>Fourth Paragraph</p>

<style type="text/css">
p {
    border: 1px solid black;
    width: 200px;
}

p#specialpara {
    position: relative;
    top: 25px;
    left: 30px;
}
</style>
```

![相对定位](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E7%9B%B8%E5%AF%B9%E5%AE%9A%E4%BD%8D.png)

## 绝对定位

{% note info %}
- 绝对定位会将元素从文档流中拿出来，它是相对于定位上下文进行定位的；
- 可以使用`top`、`right`、`bottom`和`left`来调整元素的位置；
- 绝对定位默认的定位上下文是`<body>`元素；
- 如果绝对定位的祖先元素中有`position`属性值不为`static`的元素，那么绝对定位就以其为定位上下文。
{% endnote %}

```html
<p>First Paragraph</p>
<p>Second Paragraph</p>
<p id="specialpara">Third Paragraph (with ID)</p>
<p>Fourth Paragraph</p>

<style type="text/css">
p {
    border: 1px solid black;
    width: 200px;
}

p#specialpara {
    position: absolute;
    top: 25px;
    left: 30px;
}
</style>
```

![绝对定位](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E7%BB%9D%E5%AF%B9%E5%AE%9A%E4%BD%8D.png)

## 固定定位

{% note info %}
- 固定定位会将元素从文档流中拿出来，它是根据视口（浏览器窗口或手持设备的屏幕）进行定位的，所以在页面滚动时，它的位置也不会改变；
- 可以使用`top`、`right`、`bottom`和`left`来调整元素的位置。
{% endnote %}

```html
<p>First Paragraph</p>
<p>Second Paragraph</p>
<p id="specialpara">Third Paragraph (with ID)</p>
<p>Fourth Paragraph</p>
<p>Fifth Paragraph</p>
<p>Sixth Paragraph</p>
<p>Seven Paragraph</p>
<p>Eighth Paragraph</p>
<p>Ninth Paragraph</p>
<p>Tenth Paragraph</p>


<style type="text/css">
p {
    border: 1px solid black;
    width: 200px;
}

p#specialpara {
    position: fixed;
    top: 25px;
    left: 30px;
}
</style>
```

![固定定位1](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E5%9B%BA%E5%AE%9A%E5%AE%9A%E4%BD%8D1.png)

![固定定位2](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E5%9B%BA%E5%AE%9A%E5%AE%9A%E4%BD%8D2.png)

# Flexbox

Flexbox是一种布局方式，我们称之为弹性布局。设置了`display: flex`的元素被称为容器（flex container），而它的子元素则称之为项目（flex item）。对于容器和项目，它们各自有着不同的CSS属性。

## 容器的属性

### flex-direction

Flexbox中有两根轴：主轴和交叉轴。这两个轴代表什么取决容器上的`flex-direction`属性，它共有四个可供选择的值：`row`（默认）、`row-reverse`、`column`和`column-reverse`。

![flex-direction](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/flex-direction.png)

### justify-content

`justify-content`属性用于定义项目在主轴上的对齐方式。

![justify-content](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/justify-content.png)

### align-items

`align-items`属性用于定义项目在交叉轴上的对齐方式。

![align-items1](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/align-items1.png)
![align-items2](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/align-items2.png)

### align-content

`align-content`属性用于定义多根轴线的对齐方式，如果项目只有一根轴线，该属性不起作用。

![align-content1](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/align-content1.png)
![align-content2](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/align-content2.png)
![align-content3](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/align-content3.png)

### flex-wrap

`flex-wrap`属性用于定义当主轴上项目过多时，是否允许折行。

![flex-wrap](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/flex-wrap.png)

### flex-flow

`flex-flow`属性是`flex-direction`和`flex-wrap`的简写形式。

## 项目的属性

### order

`order`属性用于定义项目的排列顺序。数值越小，排列越靠前，默认为0。

![order](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/order.png)

### flex-grow

`flex-grow`属性用于定义项目的放大比例，默认为0，即就算存在剩余空间，也不放大。

![flex-grow](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/flex-grow.png)

### flex-shrink

`flex-shrink`属性用于定义项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

![flex-shrink](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/flex-shrink.png)

### flex-basis

`flex-basis`属性用于定义项目在未放大或缩小前，占据的主轴空间大小，默认为`auto`，即项目本来的大小。

![flex-basis](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/flex-basis.png)

### flex

`flex`属性是`flex-grow`、`flex-shrink`和`flex-basis`的简写形式。

### align-self

`align-self`属性允许单个项目有与其他项目不一样的对其方式。可覆盖容器的`align-items`属性。

# display属性

{% note info %}
- `none`：完全移除元素；
- `inline`：将元素变为行内元素，它的`height`和`weight`将不会起作用；
- `block`：将元素变为块级元素；
- `inline-block`：将元素变成具有`height`和`weight`的行内元素；
- `list-item`：元素表现得像是`<li>`一样；
- `table`：元素表现得像是`<table>`一样；
- `flex`：将元素变为flex container，其子元素变为flex item；
- `inline-flex`：将元素变为行内flex container；
- `grid`：将元素变为grid container。
{% endnote %}

# 元素层叠

![元素层叠等级](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E5%85%83%E7%B4%A0%E5%B1%82%E5%8F%A0%E7%AD%89%E7%BA%A7.jpg)

{% note info %}
- 元素层叠是相对的；
- `z-index`属性只有当对应元素的`postion`属性不为`static`时才有效。
{% endnote %}

下面的例子有助于理解元素层叠是相对的。

```html
<div id="outer">
    <div id="a"></div>
    <div id="b"></div>
</div>
    

<style type="text/css">
#outer {
    width: 200px;
    height: 200px;
    background: black;
    position: relative;
    z-index: -100;
}

#a {
    width: 100px;
    height: 100px;
    background: red;
    position: relative;
    z-index: -1000;
}

#b {
    width: 100px;
    height: 100px;
    background: blue;
}
</style>
```

![元素层叠例子](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E5%85%83%E7%B4%A0%E5%B1%82%E5%8F%A0%E4%BE%8B%E5%AD%90.png)