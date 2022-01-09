---
title: 盒模型
categories:
    - HTML&CSS
abbrlink: 165da8b7
date: 2019-06-15 20:44:03
---

# 理解盒模型

{% note info %}
- 盒模型由四个部分组成：**内容区**、**内边距**、**边框**、**外边距**
- 盒模型中有四种盒子：`content-box`、`padding-box`、`border-box`、`margin-box`
{% endnote %}

![理解盒模型](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/%E7%9B%92%E6%A8%A1%E5%9E%8B.png)

# 盒模型和元素的大小

{% note info %}
1. 盒模型的大小和元素的大小不是一回事
2. 盒模型的大小是`外边距+边框+内边距+内容区`
3. 元素的大小取决于`box-sizing`属性，它有两个值：
    - `content-box`（默认）：此时元素的`width`和`height`属性设置的是`内容区`的宽度和高度
    - `border-box`：此时元素的`width`和`height`属性设置的是`边框+内边距+内容区`的宽度和高度
4. 块级元素在没有设置`width`时，它的盒模型会始终填满其父元素的宽度。
{% endnote %}

# 盒模型种类

{% note info %}
根据盒模型的不同，它们参与不同的[格式化上下文](https://aadonkeyz.com/posts/451e216f/)
- **block-level box**：`display`属性为`block`、`list-item`、`table`
- **inline-level box**：`display`属性为`inline`、`inline-block`、`inline-table`
- **run-in box**：`display`属性为`run-in`
{% endnote %}

# margin和padding的注意事项

{% note warning %}
- `margin`的`top`和`bottom`对非替换内联元素无效，例如：`<span>`和`<code>`
- `margin`和`padding`的属性值如果用百分比设置，那么都是相对于该元素包含块的**宽度**
- `margin`的属性值如果是`auto`，那么对应的外边距会自动占据包含块空间的所有可用空间
{% endnote %}

# border-radius

`border-radius`允许你设置元素的外边框圆角。当使用一个半径时确定一个圆形，当使用两个半径时确定一个椭圆

![border-radius](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius.png)

该属性是一个简写属性，是为了将`border-top-left-radius`、`border-top-right-radius`、`border-bottom-right-radius`和`border-bottom-left-radius`简写为一个属性

`border-radius`在设置椭圆形时，使用（`/`）分隔水平半轴和垂直半轴

```css
border-radius: 1em / 5em;
/* 等价于： */
border-top-left-radius:     1em 5em;
border-top-right-radius:    1em 5em;
border-bottom-right-radius: 1em 5em;
border-bottom-left-radius:  1em 5em;

/* ================================ */

border-radius: 4px 3px 6px / 2px 4px;
/* 等价于： */
border-top-left-radius:     4px 2px;
border-top-right-radius:    3px 4px;
border-bottom-right-radius: 6px 2px;
border-bottom-left-radius:  3px 4px;
```

![border-radius1](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius1.png)
![border-radius2](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius2.png)
![border-radius3](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius3.png)
![border-radius4](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius4.png)
![border-radius5](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius5.png)
![border-radius6](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E5%B8%83%E5%B1%80/border-radius6.png)

# box-shadow

{% note info %}
1. `box-shadow: inset <offset-x> <offset-y> <blur-radius> <spread-radius> <color>`，需要特别注意的是，**这些长度值需要指明单位！！！**
    - `inset`（可选）：不使用`inset`则阴影在边框外，使用`inset`则阴影在边框内
    - `<offset-x>`（必选）：水平偏移量
    - `<offset-y>`（必选）：垂直偏移量
    - `<blur-radius>`（可选）：该值越大，阴影面积越大，阴影就越大越淡。不能为负值。默认为`0`，此时阴影边缘锐利
    - `<spread-radius>`（可选）：取正值时，阴影扩大。取负值时，阴影缩小。默认为`0`，此时阴影与元素同样大
    - `<color>`（可选）：颜色。默认颜色由浏览器决定
2. 如果给出了第三个长度值，则它被解释为`<blur-radius>`。如果给出了第四个长度值，则它被解释为`<spread-radius>`
3. 如果想设置多个阴影，则不同阴影规则之间需要使用逗号分隔。越靠前的阴影规则，对应的阴影越接近显示的表层
4. 如果元素同时设置了`border-radius`，那么阴影也会有圆角效果
{% endnote %}