---
title: 媒体查询
categories:
  - HTML&CSS
abbrlink: d62e352c
date: 2019-05-20 15:29:39
---

# 媒体类型

{% note info %}
- `all`：所有媒体
- `braile`：盲文触觉设备
- `embossed`：盲文打印机
- `handheld`：手持设备
- `print`：打印预览或打印机
- `projection`：项目演示
- `screen`：彩屏设备
- `speech`：听觉类似的媒体
- `tty`：不适用像素的设备，如电传打字机
- `tv`：电视
{% endnote %}

# 媒体特性

{% note info %}
以下特性，除`scan`和`grid`外，都可以加上`min`或`max`前缀以指定范围
- `width`：视口宽度
- `height`：视口高度
- `device-width`：渲染表面的宽度（可以认为是设备屏幕的宽度）
- `device-height`：渲染表面的高度（可以认为是设备屏幕的高度）
- `orientation`：设备方向是水平还是垂直
- `aspect-ratio`：视口的宽高比。如`16:9`的宽屏显示器可以写成`aspect-ratio: 16/9`
- `color`：颜色组分的位深。如`min-color: 16`表示设备至少支持16位深
- `color-index`：设备颜色查找表中的条目数，值必须是数值，且不能为负
- `monochrome`：单色帧缓冲中表示每个像素的位数，值必须是数值（整数），且不能为负
- `resolution`：屏幕或打印分辨率。如`min-resolution: 300dpi`，也可以接受每厘米多少点，如`min-resolution: 118dpcm`
- `scan`：针对电视的逐行扫描（`progressive`）和隔行扫描（`interlace`）
- `grid`：设备基于栅格还是位图
{% endnote %}

# 媒体查询方式

## link标签的media属性

在`<link>`标签的`media`属性中指定媒体查询是CSS2的方式

```html
<link rel="stylesheet" type="text/css" media="screen" href="screenstyles.css" />
```

## @media

`@media`是在样式表中使用的媒体查询方式

```css
@media screen { ... }
```

## @import

`@import`可以根据媒体查询将其他样式表加载到当前样式表中

```css
@import url("phone.css") screen
```

# 媒体查询语法

{% note info %}
- `and`：类似于逻辑与
- `,`：类似于逻辑或
- `not`：用于对整个媒体查询取反
- `only`：指定某种特定的媒体类型，用来对那些不支持媒体特性但支持媒体类型的设备隐藏样式表
{% endnote %}

```css
/* 当设备是彩屏设备并且视口最小宽度大于等于400px时生效 */
@media screen and (min-width: 400px) { ... }


/* 当设备垂直或者视口最小宽度大于等于400px时生效 */
@media (orientation: portrait), (min-width: 400px) { ... }


/* 当视口最小宽度小于400px时生效 */
@media not all and (min-width: 400px) { ... }
/* 等价于 */
@media not (all and (min-width: 400px)) { ... }
/* 而不是 */
@media (not all) and (min-width: 400px) { ... }


@media only screen and (min-width: 400px) { ... }
```
