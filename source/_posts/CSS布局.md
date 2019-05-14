---
title: CSS布局
categories:
    - HTML&CSS
abbrlink: ee1ff2c0
date: 2019-05-13 16:06:51
---

# 盒模型

## 理解盒模型

![理解盒模型](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/%E7%9B%92%E6%A8%A1%E5%9E%8B/%E7%90%86%E8%A7%A3%E7%9B%92%E6%A8%A1%E5%9E%8B.png)

## 叠加外边距

{% note info %}
**垂直方向的外边距会叠加，水平的则不会**。
{% endnote %}

## 盒子有多大

{% note info %}
- 为没有设置`width`的盒子添加水平边框、内边距和外边距，会导致内容宽度减少，减少量等于水平边框、内边距和外边距的和;
- 为设置了`width`的盒子添加水平边框、内边距和外边距，会导致盒子扩展得更宽。**实际上，盒子的`width`属性设定的只是盒子内容区的宽度，而非盒子要占据的水平宽度**；
- 块级元素在没有设置`width`时，默认会始终填满其父元素的宽度。
{% endnote %}

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
- 当前一个元素浮动，后一个元素不浮动时，会出现元素层叠现象。
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
![父元素围不住float子元素]()

这个效果明显不是我们实际想要的，为了解决这个问题，介绍下面几种让父元素围住子元素的方法。

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
- 应用了`clear`属性的元素不会与在它之前的对应浮动元素产生元素层叠现象。
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
