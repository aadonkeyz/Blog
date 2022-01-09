---
title: HTML标签
categories:
    - HTML&CSS
abbrlink: 500cdacf
date: 2019-05-27 15:02:25
---

# HTML标签大全

太多了，整理一会之后果断放弃，[MDN上的介绍](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

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

# a标签

{% note info %}
`<a>`标签代表一个超链接，它有如下属性：
- `download`：这个属性用于指示浏览器去下载`href`所指向的资源而不是进行页面跳转。如果给`download`属性设置一个值，这个值将是对应资源被保存成本地文件过程中的默认名称。
- `href`：URL。
- `hreflang`：用于指定链接资源的人类语言
- `ping`：包含一个以空格分隔的 URL 列表，当跟随超链接时，将由浏览器发送带有正文的 PING 的 POST 请求，通常用于跟踪。
- `rel`：该属性执行了目标对象到链接对象的关系。该值是空格分隔的列表类型值。
- `target`：该属性指定在何处显示链接的资源。它的值包括：`_self`、`_blank`、`_parent`和`_top`。
- `type`：该属性指定在一个 MIME type 链接目标的形式的媒体查询。其仅提供建议，并没有内置的功能。

---
`<a>`标签还有四个伪类：
- `link`：具有`href`属性。
- `visited`：被访问过。
- `hover`：鼠标悬浮。
- `active`：鼠标点击瞬间。
{% endnote %}

下面展示一个使用`download`属性的例子。

```html
<a download="国旗" href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALYAAAB5CAMAAACjkCtXAAAAe1BMVEX/AAD////XsbH0RETe2dnToqLi7e3wVVX80Rb80xb81Rb82Bf/MQX/JQT8xhX/FwL8yxX+VQn/Ogb83Rf8whT9nBD+TAj9jw/9lQ/9rhL9uBP8uxT+bgv9ohDGv7/SuLj9hA3+XQj9eAn+fw39qBL+Qwf+ZQv2OzvmbW2v1N+9AAACiklEQVR4nO3ZCW+bMByHYdjpg8PcEMhKUpL2+3/CORs9osUGofFzUP6vlKpqFfTINTZOPd//9WVTPf3wfd/z/e/epvpJbGDERkZsZMRGRmxkxEb2YOxkFc3slrLzVTSzW8QOgpIlQbCWaUaL2D0XnMsqXAs13bJJEkZSOZ3dy9ixGMRhJdGslrH73uuKlUSzmmbHN36Wji9nTbKT9pb7UxO/XqdJdp+drBeIq/9umtEku8mO1guEuYv1e4qtl7q95e1VUbCi6OETZYp9YDy3bCtpoTeeAX93TrGfBVel7QIqixzsllPsvWDCNrnPUZLbJ/8qGdjp+HePW82ux+/TG8N6TL3gbtipEoypdqgqxvTk7usmV4zJAe8zZBjtQ5RxLoSU7OKWUug7L2ucboxXGed2zwT7nIgcH2iuMt+S5S7jH+isdnkq+CfbSnKM3txiZ10E8VkXwHJ0i8HhQeZmdvY4vXlkvkDsZHmxsmu9kPDLgGfm2/H+HqUCxZlQjdBfTUNaNQ1vmuquHqWOeofkZz3mnCvDQ3dYSS4cHOEt7HgnRH7hdkzTTBdQKnIwSyzskxTt30Hu9GZjeH/Xlrvn9XimLOwiG97m7CnPzrff340vcGZ2HNUfd9qptZ1x8FlG+/pGc3JAN/Zgn297idNHq8XsYYMfXSaHkvXJ2d0D1jL2KWec8Xol04yWTpJGSpezZCE7zAflYHN8byG7q7xgv7l/gvzZikKHO9CjbTduIzYyYiMjNjJiIyM2MmIjIzYyYiMjNjJiIyM2MmIjIzYyYiMjNjJiIyM2MmIjIzYyYiMjNjJiIyM2MmIjIzYyYiMjNjJiIyM2MmIje2e/fN1SryP728Ya2Vtso+zffNMqoOoYARcAAAAASUVORK5CYII=">下载图片</a>
```

# input标签

{% note info %}
`<input>`标签用于接受来自用户的数据。它有如下属性：
- `autocomplete`：一个字符串，描述输入应提供的任何类型的自动完成功能。自动完成的典型实现只是回忆在同一输入字段中输入的先前值，但可以存在更复杂的自动完成形式。例如，浏览器可以与设备的联系人列表集成，以在电子邮件输入字段中自动完成电子邮件地址。不过当`type`属性值为`button`、`file`等时，`<input>`不会返回文本或数值数据，此时`autocomplete`属性将被忽略。
- `autofocus`
- `disabled`
- `form`：该属性值为一个`<form>`的`id`，用于指示该`<input>`是属于它的。如果缺失，则该`<input>`属于最近的`<form>`或者不属于任何`<form>`。
- `list`：该属性值为一个`<datalist>`的`id`，`<datalist>`用于为该`<input>`提供建议列表。
- `name`：该`<input>`的名称，与表单数据一起提交。
- `readonly`
- `required`
- `tabindex`：该`<input>`在当前文档的 Tab 导航顺序中的位置。
- `type`：该`<input>`的类型。包括：`button`、`checkbox`、`color`、`date`、`datetime-local`、`email`、`file`、`hidden`、`image`、`month`、`number`、`password`、`radio`、`range`、`reset`、`search`、`submit`、`tel`、`text`、`time`、`url`、`week`。
- `value`：该`<input>`的当前值。
{% endnote %}

# meta标签

`<meta>`标签中包含那些不能被`<base>`、`<link>`、`<script>`、`<style>`和`<title>`标签所表示的内容。并且`<meta>`标签一次只干一件事，如果想利用`<meta>`标签设置多个内容，那就多写几个`<meta>`标签。它有如下属性：

{% note info %}
- `charset`：该属性表明了页面的编码方式，通常为`UTF-8`。
- `content`：该属性值对应于`http-equiv`属性或`name`属性的值。
- `http-equiv`：该属性定义 HTTP 的一些首部字段，对应的首部字段值在`content`属性中。
    - `content-security-policy`：定义页面的内容安全策略，内容策略主要指定允许的服务器源和脚本端点，这有助于防止 XSS 攻击。
    - `refresh`：
        - 如果`content`属性值是一个正整数，则该正数代表页面重载的时间间隔（秒）。
        - 如果`content`属性值是一个正整数，并且后面跟着`;url=`和一个合法的 URL，则页面将在指定秒数后进行跳转。
- `name`：该属性定义能定义如下内容，同样的，这些内容的值在`content`属性中。
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
