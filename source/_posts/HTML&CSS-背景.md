---
title: 背景
categories:
    - HTML&CSS
abbrlink: 28d95575
date: 2019-05-16 10:12:34
---

# 背景结构图

![背景结构图](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/%E8%83%8C%E6%99%AF%E7%BB%93%E6%9E%84%E5%9B%BE.png)

# background-color

通过这个属性可以指定盒模型的背景颜色。

# background-image

格式为：`background-image: url(图片路径/图片文件名)`。

# background-repeat

该属性用于定义背景图片的重复方式。

![background-repeat](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/background-repeat.png)

# background-position

该属性用于定义背景图片的位置，该属性的值可以是：关键字、百分比、绝对或相对单位的数值，也可以将关键字与百分比或数值配合使用。

关键字包括：`top`、`right`、`bottom`、`left`和`center`。

![background-position](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/background-position.png)

# background-size

该属性用于定义背景图片的大小。

![background-size](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/background-size.png)

# background-attachment

该属性用于定义背景图片是否随元素滚动而移动。这个属性的默认值是`scroll`，即背景图片随元素移动。如果把它的值改为`fixed`，那么背景图片不会随元素滚动而移动。

# background-clip

该属性用于定义背景图片的覆盖范围。

![background-clip](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/background-clip.png)

# background-origin

该属性用于定义背景图片的起点位置。

![background-origin](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/background-origin.png)

# background

{% note info %}
`background`属性是有关背景的简写属性，这里需要注意的是，当`background-position`和`background-size`同时出现时，应写为`background-position/background-size`。且如果想指定`background-size`，则必须指定`background-position`，否则背景图片不会呈现。
{% endnote %}

# 多背景图片

直接上例子：

```css
#background-demo {
background:
    url(images/demo1.png) 20px -10px no-repeat,
    url(images/demo2.png) 145px 0px no-repeat,
    url(images/demo3.png) 14px -30px no-repeat, #ffbd75;
}
```

{% note info %}
**在`background`中先定义的背景图片显示在上方，或者说更接近前景**。所以demo1.png显示在最上方，demo3.png在最下方。
{% endnote %}

# 背景渐变

`background-image`属性除了可以指定背景图片外，还可以定义背景的渐变效果。

## linear-gradient

{% note info %}
`linear-gradient`使用步骤：
1. **确定渐变方向**：渐变方向有两种定义方式：关键字和角度。当未指定渐变方向时，默认为`to bottom`，也叫作`180deg`；
2. **在渐变方向的不同位置处，定义渐变点**：渐变点的个数不能少于两个。如果为同一个渐变点设定两种颜色，会得到突变效果。
{% endnote %}

首先通过下图理解渐变方向。

![渐变方向](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/%E6%B8%90%E5%8F%98%E6%96%B9%E5%90%91.png)

然后就可以上例子了。

```html
<div id="outer">
    <div id="a">a</div>
    <div id="b">b</div>
    <div id="c">c</div>
    <div id="d">d</div>
    <div id="e">e</div>
    <div id="f">f</div>
</div>
    

<style type="text/css">
#outer {
    display: flex;
    width: 700px;
    align-items: center;
    flex-wrap: wrap;
    justify-content: space-between;
    color: white;
    font-size: 30px;
}
div {
    margin-top: 40px;
    width: 200px;
    height: 200px;
}

#a {
    background-image: linear-gradient(red, blue);
}

#b {
    background-image: linear-gradient(to bottom right, red, blue);
}

#c {
    background-image: linear-gradient(90deg, red, blue);
}

#d {
    background-image: linear-gradient(red 20%, yellow 50%, blue 80%);
}

#e {
    background-image: linear-gradient(red 30%, blue 30%);
}

#f {
    background-image: linear-gradient(45deg, red 30%, blue 30%);
}
</style>
```

![linear-gradient](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/linear-gradient.png)

## radial-gradient

{% note info %}
`linear-gradient`时需要设置渐变方向。而使用`radial-gradient`时则需要设置渐变的形状、大小和位置：
1. **形状**：`circle`或`ellipse`（默认）；
2. **大小**：
    - `closest-side`：渐变从圆形（或椭圆形）中心开始，到距离中心最近的边终止；
    - `closest-corner`：渐变从圆形（或椭圆形）中心开始，到距离中心最近的角终止；
    - `farthest-side`：渐变从圆形（或椭圆形）中心开始，到距离中心最远的边终止；
    - `farthest-corner`：渐变从圆形（或椭圆形）中心开始，到距离中心最远的角终止；
    - `12rem`：会生成一个直径为`12rem`的圆；
    - `40px 30px`：会生成一个X方向宽40像素、Y方向高30像素的椭圆形。
3. **位置**：使用`at`定义渐变中心的位置，例如`at center`或`at top 100px right`。
{% endnote %}

```html
<div id="outer">
    <div id="a">a</div>
    <div id="b">b</div>
    <div id="c">c</div>
    <div id="d">d</div>
    <div id="e">e</div>
    <div id="f">f</div>
</div>
    

<style type="text/css">
#outer {
    display: flex;
    width: 700px;
    align-items: center;
    flex-wrap: wrap;
    justify-content: space-between;
    color: white;
    font-size: 30px;
}
div {
    margin-top: 40px;
    width: 200px;
    height: 200px;
}

#a {
    background-image: radial-gradient(red, blue);
}

#b {
    background-image: radial-gradient(closest-side, red 50%, blue 50%);
}

#c {
    background-image: radial-gradient(closest-corner, red 50%, blue 50%);
}

#d {
    background-image: radial-gradient(circle closest-side at top 45px left 45px, red 20%, yellow 20%, blue 50%);
}

#e {
    background-image: radial-gradient(ellipse 200px 100px at bottom left , red 20%, yellow 50%, blue 80%);
}

#f {
    background-image: radial-gradient(circle closest-corner at 100%, red, red 50%, yellow 75%, blue 75%);
}
</style>
```

![radial-gradient](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E8%83%8C%E6%99%AF/radial-gradient.png)

## 重复渐变

规则相同，就是在前面加上`repeating-`前缀