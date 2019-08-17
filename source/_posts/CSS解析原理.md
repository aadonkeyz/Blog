---
title: CSS解析原理
abbrlink: df6afb91
date: 2019-08-17 23:37:54
categories:
    - 前端
---

本文参考自[**探究 CSS 解析原理**](https://juejin.im/entry/5a123c55f265da432240cc90)

# render tree的生成

在利用 DOM 和 CSSOM 合成 render tree的时候，需要将每一条 CSS 样式规则与对应的 DOM 元素关联起来。然而实际中，样式规则可能数量很大，但是绝大多数不会匹配到任何 DOM 元素上，所以有一个快速的方法来判断 CSS 选择器是否具有匹配的 DOM 元素是极其重要的。


# 内联样式的解析


