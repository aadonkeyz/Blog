---
title: 浏览器是如何渲染一个页面的？
categories:
    - FE
abbrlink: b5fc6f17
date: 2019-07-31 19:41:24
---

# 渲染流程

![浏览器渲染流程](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/FE/%E6%B5%8F%E8%A7%88%E5%99%A8%E6%98%AF%E5%A6%82%E4%BD%95%E6%B8%B2%E6%9F%93%E4%B8%80%E4%B8%AA%E9%A1%B5%E9%9D%A2%E7%9A%84/%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B.jpg)

{% note info %}
- JavaScript： 构建 DOM + CSSOM
- Style： 构建 Render Tree
{% endnote %}

## DOM

**Document Object Model** 通过在内存中表示文档结构，将网页连接到脚本或编程语言。 DOM 使用逻辑树来表示一个文档结构。树的每一个分之都以一个节点结尾，并且每个节点都包含对象。 DOM 方法允许以编程方式访问树，使用这些方法可以更改文档的结构、样式或内容。DOM 上的节点并不一定总是 HTML 元素，当浏览器创建 DOM 树时，它还将诸如注释、属性、文本之类的内容另存为树中的单独节点。

{% note warning %}
- 构建 DOM 的过程中遇到不携带 `async` 或 `defer` 的 `<script>` 标签，浏览器会停止 DOM 的构建，等待 JS 的下载和执行。
- 构建 DOM 的过程中携带 `async` 的 `<script>` 标签下载完毕后，浏览器会停止 DOM 的构建，等待 JS 的执行。
{% endnote %}

## CSSOM

**CSS Object Model** 允许通过 JavaScript 操纵 CSS，它非常类似于 DOM。CSSOM 和 DOM 是相互独立的。CSSOM 的构建过程类似于 DOM 的构建，不过它依赖的是样式结构，而不是文档结构。

{% note warning %}
CSSOM 的构建会阻塞浏览器下载&执行 JS。
{% endnote %}

## Render

Render Tree 是将 DOM 和 CSSOM 结合在一起而生成的树状结构。通过 Render Tree 可以知道在每个节点上应用哪些样式规则，但是无法知道这些节点的确切位置及大小信息。在 Render Tree 被构建出来之前，页面上不会呈现任何内容。

{% note warning %}
- `display: none` 的元素不会出现在 Render Tree 中。
- 伪元素会出现在 Render Tree 中。
{% endnote %}

## Layout

Layout Tree 是在 Render Tree 的基础上，为每个节点计算其布局信息，从而构建出的树状结构。

{% note warning %}
避免在通过 CSSOM 设置完样式后立刻对其进行访问，因为这样要求浏览器在你访问样式时返回最新的样式信息，相当于要求浏览器立刻进入 layout 阶段，从而造成不必要的性能损耗（本来可以在下一帧再更新样式的）。
{% endnote %}

## Paint

在这一阶段需要了解两个概念：layer 和 rasterization。Layer 让浏览器知道以何种顺序去渲染相互层叠的元素，同时帮助浏览器在网页的整个生命周期中更有效执行的 paint 操作。Rasterization 是对每一个 layer 进行栅格化，使得浏览器以更高效的方式进行 paint 操作。

## Compositing

利用计算好的 layer 和每一层 layer 内的栅格，组合出 viewpoint 内应该展示的页面。

# first paint

为了更好的用户体验，浏览器不一定非要等到 DOM 和 CSSOM 构建完，才开始页面的渲染。可以参考 [Chrome的First Paint](http://eux.baidu.com/blog/fe/Chrome%E7%9A%84First%20Paint) 的思路自己在浏览器的 performance 面板中观察下实际的效果。

最佳实践是将 `<link rel="stylesheet">` 标签放在 `<head>` 中，将 `<script>` 标签放在 `</body>` 前。当然如果你为 `<script>` 标签添加 `async` 或 `defer` 属性的话，将它们放在 `<head>` 中也是可以的。

详细的原理可以参考 [How the browser renders a web page? — DOM, CSSOM, and Rendering](https://medium.com/jspoint/how-the-browser-renders-a-web-page-dom-cssom-and-rendering-df10531c9969)。

# 优化方法

{% note info %}
- 使用 `requestAnimationFrame()` 代替 `setTimeout()` 和 `setInterval()`。
- 避免 long task 的出现。
- 使用 Web Worker。
- 减少样式选择器的复杂度。
- 避免 [**强制同步布局**](https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing#avoid_forced_synchronous_layouts)
{% endnote %}

# 参考文献

- [Critical rendering path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
- [Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work)
- [How the browser renders a web page? — DOM, CSSOM, and Rendering](https://medium.com/jspoint/how-the-browser-renders-a-web-page-dom-cssom-and-rendering-df10531c9969)
- [Inside look at modern web browser (part 3)](https://developers.google.com/web/updates/2018/09/inside-browser-part3)
- [Rendering Performance](https://developers.google.com/web/fundamentals/performance/rendering)
- [Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)
- [How CSS works: Parsing & painting CSS in the critical rendering path](https://blog.logrocket.com/how-css-works-parsing-painting-css-in-the-critical-rendering-path-b3ee290762d3/)
- [The Secrets of the CSSOM & Why You Should Care](https://glenelkins.medium.com/the-secrets-of-the-cssom-why-you-should-care-943a1d50307b)
- [Software vs. GPU Rasterization in Chromium](https://software.intel.com/content/www/us/en/develop/articles/software-vs-gpu-rasterization-in-chromium.html)
- [Accelerated Rendering in Chrome](https://www.html5rocks.com/en/tutorials/speed/layers/)
- [Chrome的First Paint](http://eux.baidu.com/blog/fe/Chrome%E7%9A%84First%20Paint)