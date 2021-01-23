---
title: 跨域问题常见解决方案
categories:
  - FE
abbrlink: 713a5759
date: 2019-07-20 20:22:06
---

我个人将跨域问题分类为两种：**网络请求跨域**和**非网络请求跨域**。

# 网络请求跨域

{% note info %}
网络请求跨域的解决方案有：
- 图像img
- JSONP
- CORS
- Nginx反向代理、Nodejs中间件
- SSE、Web Socket（它们是服务器推送技术，无跨域问题）

[详细信息在这里](https://aadonkeyz.com/posts/5ffed448/)
{% endnote %}

# 非网络请求跨域

## document.domain

{% note info %}
如果要进行通信的窗口之间主域相同，子域不同，则可以直接将它们的 `document.domain` 修改为主域，这样就不存在跨域问题了。
{% endnote %}

```html
<iframe name="a" src="http://a.domain.com"></iframe>
<iframe name="b" src="http://b.domain.com"></iframe>
```

```html
<!-- domain 初始值为 a.domain.com -->
<script>
  document.domain = 'domain.com';
  var user = 'a';
</script>
```

```html
<!-- domain 初始值为 b.domain.com -->
<script>
  document.domain = 'domain.com';
  alert('get js data from a ---> ' + window.parent.frames.a.user);
</script>
```

## location.hash

{% note info %}
父窗口 A 修改子窗口 B 的 `location.hash` 值，会触发对应子窗口 B `window` 对象的 `onhashchange` 事件处理程序。这个过程是单向的，但是你可以通过在子窗口 B 中添加一个 `iframe` 标签，这个 `iframe` 的 `src` 域名与父窗口域名相同的，然后在这个 `iframe` 页面内调用父窗口 A 的函数，此时是不存在跨域的。
{% endnote %}

## window.name

{% note info %}
`window` 对象的 `name` 属性有一个独特之处，一个窗口只要一直存在并且没有被主动的更改 `name` 属性，那么它的 `name` 属性会一直保持不变。即窗口的 URL（或 `iframe` 的 `src`）随意变化，`name` 属性也不会变。`name` 属性内存最大可以支持 2MB，所以可以通过它来传递数据。
{% endnote %}

## window.postMessage

通过 `window.postMessage` 方法和 message 事件完成数据的传递。

# 参考

- [前端常见跨域解决方案（全）](https://segmentfault.com/a/1190000011145364)
