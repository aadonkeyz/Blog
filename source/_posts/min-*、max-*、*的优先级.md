---
title: min-*、max-*、*的优先级
categories:
    - HTML&CSS
abbrlink: df94ca98
date: 2019-07-22 14:13:38
---

本文参考链接[**理解css中min-width和max-width，width与它们之间的区别联系**](http://www.fly63.com/article/detial/561)

{% note warning %}
`width`与`height`的情况是一致的，下面以`width`为例进行说明。
{% endnote %}

# min-width比width大时

{% note info %}
`min-width`的优先级高于`width`，即使有`!important`也是如此。
{% endnote %}

```html
<div></div>
<style>
    div {
        height: 200px;
        min-width: 400px;
        width: 200px !important;
        background: black;
    }
</style>
```

# min-width比max-width大时

{% note info %}
`min-width`的优先级高于`max-width`，即使有`!important`也是如此。
{% endnote %}

```html
<div></div>
<style>
    div {
        height: 200px;
        min-width: 400px;
        max-width: 200px !important;
        background: black;
    }
</style>
```

# max-width比width小时

{% note info %}
`max-width`的优先级高于`width`，即使有`!important`也是如此。
{% endnote %}

```html
<div></div>
<style>
    div {
        height: 200px;
        max-width: 200px;
        width: 400px !important;
        background: black;
    }
</style>
```
