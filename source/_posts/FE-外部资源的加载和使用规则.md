---
title: 外部资源的加载和使用规则
categories:
  - FE
abbrlink: 7fca0ecb
date: 2019-06-27 21:41:06
---

# script 标签

{% note warning %}
带有 `src` 属性的 `<script>` 标签不应该在其 `<script>` 和 `</script>` 之间再包含额外的 JavaScript 代码。如果包含了嵌入的代码，也只会加载并执行外部 js 文件，嵌入的代码会被忽略。
{% endnote %}

在用 `<script>` 加载外部 js 文件时，只要不存在 `defer` 和 `async` 属性，浏览器会按照 `<script>` 标签在页面中出现的先后顺序依次加载并执行它们。所以如果将 `<script>` 放在 `<head>` 中，意味着必须等到全部 JavaScript 代码都被加载、解析和执行完成以后，才能开始呈现页面的内容（浏览器在遇到 `<body>` 标签时才开始呈现内容）。

{% note info %}
- 如果在使用 `<script>` 时加入 `defer` 属性，意味着要并行加载这个 js 文件，但是会延迟到整个页面都被解析完成之后，才开始执行这个这个 js 文件。
- 如果在使用 `<script>` 时加入 `async` 属性，意味着要异步加载这个 js 文件，如果在页面解析过程中完成了 js 文件的加载，浏览器会暂时停止页面的解析，转而执行该 js 文件。因为加载 js 文件的过程是异步的，所以无法保证 js 文件是按顺序加载完成的，也就无法保证 js 文件按顺序执行。
{% endnote %}

用一张图生动的展示页面解析与 js 加载执行过程。

![页面解析与 js 加载执行过程](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/FE/%E9%A1%B5%E9%9D%A2%E8%A7%A3%E6%9E%90%E4%B8%8E%20js%20%E5%8A%A0%E8%BD%BD%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

上图来源于[**深入浅出浏览器渲染原理**](https://blog.fundebug.com/2019/01/03/understand-browser-rendering/)

# link 标签

当 `<link>` 标签的 `rel` 属性值为 `stylesheet` 时，代表要加载外部的 CSS 样式表，它不会阻塞 DOM 的解析，但是会阻塞 DOM 的渲染（意思就是会阻塞 render tree 的生成，从而让浏览器暂时无法渲染对应的内容）。

但是当 `rel` 属性值为 `preload` 时，意味着要浏览器立即加载对应的外部资源，但是并不会去使用它。关于它的具体用法请看 [**MDN**](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)，下面的例子也是来源于MDN。

```html
<head>
  <meta charset="utf-8">
  <title>JS and CSS preload example</title>

  <link rel="preload" href="style.css" as="style">
  <link rel="preload" href="main.js" as="script">

  <link rel="stylesheet" href="style.css">
</head>

<body>
  <h1>bouncing balls</h1>
  <canvas></canvas>

  <script src="main.js" defer></script>
</body>
```

# @import

通过 `@import` 方式引入的样式表，会在整个页面内容都加载完成之后才开始加载、解析。
