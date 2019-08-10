---
title: HTML标签
categories:
    - HTML&CSS
abbrlink: 500cdacf
date: 2019-05-27 15:02:25
---

# HTML标签大全

太多了，整理一会之后果断放弃，[**MDN上的介绍**](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

# HTML模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="renderer" content="webkit" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <meta name="description" content="简介" />
        <meta name="keywords" content="关键字" />
        <meta name="author" content="作者" />
        <title>题目</title>
        <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
        <link rel="icon" href="/favicon.ico" />
        <link rel="stylesheet" href="style.css" type="text/css" />
        <style type="text/css">
            /* CSS内部样式表 */
        </style>
    </head>
    <body>
        <!-- 这里放内容 -->
        <script type="text/javascript" src="example.js"></script>
    </body>
</html>
```

# meta标签

{% note warning %}
- `<meta>`标签中包含那些不能被`<base>`、`<link>`、`<script>`、`<style>`和`<title>`标签所表示的内容。
- `<meta>`标签一次只干一件事，如果想利用`<meta>`标签设置多个内容，那就多写几个`<meta>`标签。
{% endnote %}

## charset属性

{% note info %}
该属性表明了页面的编码方式，通常为`UTF-8`。
{% endnote %}

## content属性

{% note info %}
该属性值对应于`http-equiv`属性或`name`属性的值。
{% endnote %}

## http-equiv属性

{% note info %}
该属性定义 HTTP 的一些首部字段，对应的首部字段值在`content`属性中。
- `content-security-policy`：定义页面的内容安全策略，内容策略主要指定允许的服务器源和脚本端点，这有助于防止 XSS 攻击。
- `refresh`：
    - 如果`content`属性值是一个正整数，则该正数代表页面重载的时间间隔（秒）。
    - 如果`content`属性值是一个正整数，并且后面跟着`;url=`和一个合法的 URL，则页面将在指定秒数后进行跳转。
{% endnote %}

## name属性

{% note info %}
该属性定义能定义如下内容，同样的，这些内容的值在`content`属性中。
- `author`：定义作者。
- `description`：定义页面的描述信息。在你收藏一个页面时，描述信息就是来自于这里的。
- `generator`：生成页面的软件的标识符。
- `keywords`：用`,`分隔的关键词。
- `referrer`：控制所有从页面发出的 HTTP 请求的`Referer`首部字段。下面所述的**页面的源包含协议、域名和端口**。
    - `no-referrer`：不发送`Referer`首部字段。
    - `origin`：发送页面的源。
    - `no-referrer-when-downgrade`
    - `origin-when-cross-origin`：对于与页面同源的请求发送页面完整的 URL，与页面不同源的请求只发送页面的源。
    - `same-origin`：对于与页面同源的请求发送页面的源，与页面不同源的请求不发送`Referer`首部字段。
    - `strict-origin`
    - `strict-origin-when-cross-origin`
    - `unsafe-URL`：发送页面完整的 URL。
- `theme-color`：建议客户端应该用来使用的主题颜色。
- `color-scheme`：指定一个或多个主题颜色。
    - `normal`
    - `[light | dark]+`
    - `only light`
- `creator`：定义网页作者，即一个组织或者机构的名称。
- `googlebot`
- `publisher`：定义网页发布者。
- `robots`
- `slurp`
- `viewport`：专门用于为移动端设备定义视口的大小。
    - `width`
    - `height`
    - `initial-scale`
    - `maximum-scale`
    - `minimum-scale`
    - `user-scalable`
{% endnote %}
