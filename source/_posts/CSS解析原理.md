---
title: CSS解析原理
abbrlink: df6afb91
date: 2019-08-17 23:37:54
categories:
    - 前端
---

本文参考自[**探究 CSS 解析原理**](https://juejin.im/entry/5a123c55f265da432240cc90)

# CSS选择器的解析顺序

在利用 DOM 和 CSSOM 合成 render tree的时候，需要将样式表中的每一条 CSS 样式规则与对应的 DOM 元素关联起来。然而实际中，样式规则可能数量很大，但是绝大多数不会匹配到任何 DOM 元素上，所以有一个快速的方法来判断 CSS 选择器是否具有匹配的 DOM 元素是极其重要的。

以下面的例子讲解CSS 选择器的解析规则：

```html
<div>
   <div class="jartto">
      <p><span> 111 </span></p>
      <p><span> 222 </span></p>
      <p><span> 333 </span></p>
      <p><span class='yellow'> 444 </span></p>
   </div>
</div>
<style>
div > div.jartto p span.yellow{
   color: yellow;
}
</style>
```

{% note info %}
如果采用**从左至右**的规则，过程如下所示：
1. 首先找到所有`<div>`。
2. 在每个`<div>`内寻找所有`class=jartto`的子`<div>`。
3. 在步骤二中找到的每个子`<div>`内接着按 CSS 选择器进行寻找，直到最后。

**这样的搜索过程对于一个只是匹配很少节点的选择器来说，效率是极低的，因为我们花费了大量的时间在回溯匹配不符合规则的节点。**
{% endnote %}

{% note info %}
如果采用**从右至左**的规则，过程如下所示：
1. 首先找到所有`class=yellow`的`<span>`。
2. 然后判断这些`<span>`的父元素是否为`<p>`
3. 接着判断这些`<p>`的父元素是否为`class=jartto`的`<div>`。
4. 最后判断这些`<div>`的父元素是否也是`<div>`。

**因为每一个元素都只拥有一个父元素，所以从右至左的解析 CSS 选择器可以有效减少无效匹配的次数，从而使匹配更快、性能更优。**
{% endnote %}

# computedStyle的计算

render tree 生成时，元素的 computedStyle 是经过层叠计算后得到的。在某些特定的情况下，浏览器会让不同元素之间共享它们的 computedStyle。也就是说，如果多个元素的 computedStyle 不通过计算就可以确认它们相等，那么这个 computedStyle 只会被计算一次，从而提高了性能。

只要元素之间满足以下条件，它们之间就可以共享 computedStyle。

{% note info %}
- 元素不能有`id`属性。
- 元素的标签名必须相同，即必须是同类型的元素。
- 元素的`class`属性必须相同。
- 元素之间的 mappedAttribute（一些可以影响 CSS ComputedStyle 的 HTML 属性） 必须相等。
- 元素不能有`style`属性，哪怕是这些元素的`style`属性值相同也不可以。
- 不能使用 sibling selector。例如，`first-child`、`:last-selector`、`+ selector`。
{% endnote %}

# 选择器书写建议

{% note info %}
- ID 选择器是非常高效的，且 ID 是唯一的，所以在使用的时候应该单独使用，不需要再指定标签名等。
- 避免深层次的选择器。
- 慎用子代选择器。
- 属性选择的的解析速度非常慢，慎用。
{% endnote %}

