---
title: 选择器
categories:
    - HTML&CSS
abbrlink: 7c8269b4
date: 2019-05-07 19:07:54
---

# 选择器种类

## 标签选择器

```html
<div>
    <span>选择器</span>
</div>

<style type="text/css">
span {
    color: red;
}
</style>
```

## 类选择器

```html
<div>
    <span class="my-span">选择器</span>
</div>

<style type="text/css">
.my-span {
    color: red;
}
</style>
```

## id选择器

```html
<div>
    <span id="my-span">选择器</span>
</div>

<style type="text/css">
#my-span {
    color: red;
}
</style>
```

## 通用选择器

```html
<div>
    <h1>h1</h1>
    <p>p</p>
    <span>span</span>
</div>

<style type="text/css">
* {
    color: red;
}
</style>
```

## 属性选择器

{% note info %}
假设元素内设置了`data-attr="value1 value2 value3"`，下面介绍对它使用属性选择器的几种方式：
- `[data-attr]`：根据属性名称；
- `[data-attr="value1 value2 value3"]`：根据属性名和属性值；
- `[data-attr^="val"]`：以......开头；
- `[data-attr*="value2"]`：包含......；
- `[data-attr$="3"]`：以......结尾。
{% endnote %}

```html
<div>
    <span data-attr="value1 value2 value3">选择器</span>
</div>

<style type="text/css">
[data-attr] {
    color: red;
}

[data-attr="value1 value2 value3"] {
    color: red;
}

[data-attr^="val"] {
    color: red;
}

[data-attr*="value2"] {
    color: red;
}

[data-attr$="3"] {
    color: red;
}
</style>
```

# 选择器使用规则

{% note info %}
选择器的组合使用：
- `div, span`：同时选择`<div>`和`<span>`元素；
- `div.classValue`：选择`class`属性值包含`classValue`的`<div>`元素；
- `div#idValue`：选择`id`属性值为`idValue`的`<div>`元素；
- `div[data-attr]`：选择具有`data-attr`属性的`<div>`元素；
- `div.class1.class2`：选择`class`属性值同时包含`class1`和`class2`的`<div>`元素。

---
根据结构来使用选择器：
- `selector1 selector2`：当`selector2`是`selector1`的后代元素时，选择`selector2`；
- `selector1 > selector2`：当`selector2`是`selector1`的子元素时，选择`selector2`；
- `selector1 + selector2`：当`selector2`是`selector1`的同胞元素，且`selector2`紧跟着`selector1`时，选择`selector2`；
- `selector1 ~ selector2`：当`selector2`是`selector1`的同胞元素，且`selector2`位于`selector1`后面时，选择`selector2`；
- `selector1 * selector2`：当`selector2`是`selector1`的后代元素，且`selector2`不是`selector1`的子元素时，选择`selector2`。
{% endnote %}

# 伪类和伪元素

[MDN上伪类和伪元素的总结](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Pseudo-classes_and_pseudo-elements)

{% note info %}
- 伪类用于给已经存在于文档树中的元素添加不存在的类，并为其添加样式；
- 伪元素用于创建文档树中不存在的元素，并为其添加样式。
{% endnote %}

# 样式层叠

## 样式来源

{% note info %}
以下就是浏览器层叠各个来源样式的顺序：
- 浏览器默认样式表
- 用户样式表
- 作者链接样式表（按照它们链接到页面的先后顺序）
- 作者嵌入样式
- 作者行内样式
{% endnote %}

## 特指度

{% note info %}
1. `100`：ID选择器。
2. `10`：类选择器、伪类选择器和属性选择器。
1. `1`：标签选择器和伪元素。
{% endnote %}

![特指度](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/CSS%E9%80%89%E6%8B%A9%E5%99%A8/%E7%89%B9%E6%8C%87%E5%BA%A6.png)

## 层叠规则

{% note info %}
1. 带有`!import`的优先级最高（开挂一样的存在）。
2. 行内样式优先级仅次于`!import`。
3. 抛开`!import`和行内样式，其他情况下，特指度大的优先级高。
4. 当特指度相同时，嵌入样式表 > 链接样式表 > 用户样式表 > 浏览器默认样式表。
5. 当特指度相同时，则后出现的优先级高。
{% endnote %}