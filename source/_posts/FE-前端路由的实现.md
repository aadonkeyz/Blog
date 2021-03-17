---
title: 前端路由的实现
categories:
    - FE
abbrlink: eb815672
date: 2019-07-12 08:34:26
---

前端路由的实现有两种方式：hash 和 history。

# hash

{% note info %}
1. 此时浏览器显示的 URL 为 `https://aadonkeyz.com/posts/eb815672/`。
2. 执行 `location.hash = '#hash'` 后，生成一条新的浏览记录，其 URL 为 `https://aadonkeyz.com/posts/eb815672/#hash`，并将这条新的记录加入到浏览器历史记录的栈顶。与此同时浏览器的浏览状态会跳转到这个最新的记录上。
3. `location.hash` 改变时，触发了 `window` 对象上注册的 `onhashchange` 事件处理程序。
4. 页面内容进行对应的更新。
5. 当使用前进或者后退时，会再次触发 `onhashchange` 事件处理程序，页面内容会再次进行相应的更新。
6. 前端路由实现。
{% endnote %}

{% note warning %}
**在 URL 中，`#` 以及它之后的内容是用于在页面内定位的锚点，它们的改变不会引起浏览器发送网络请求**。
{% endnote %}

# history

{% note info %}
1. 此时浏览器显示的 URL 为 `https://aadonkeyz.com`。
2. 更新页面内容，同时通过 `history.pushState()` 或者 `history.replaceState()` 新增/替换浏览器历史记录的栈顶记录，浏览器的浏览状态跳转到栈顶记录上，此时 URL 为 `https://aadonkeyz.com/posts/eb815672`。**通过这两个方法，不会触发任何事件，也不会引起浏览器的加载行为。**
3. 当浏览器前进或后退到某条由 `history.pushState()` 或者 `history.replaceState()` 生成的浏览记录时，会触发 `window` 对象上注册的 `onpopstate` 事件处理程序，并且在事件处理程序的内部，`event.state` 中保存着该浏览记录的状态对象的拷贝。因此可以根据这个 `event.state` 更新页面。
4. 前端路由实现。
{% endnote %}

{% note warning %}
**采用这种方式时，为了防止浏览器真的去加载对应的 URL，从而返回 `404 Not Found` 的尴尬局面，需要服务器的支持，如果 URL 匹配不到任何静态资源，则应该返回根页面 HTML**。
{% endnote %}

# 更多

[「前端进阶」彻底弄懂前端路由](https://juejin.cn/post/6844903890278694919#heading-10) 中对前端路由进行了简单的实现，感兴趣的可以阅读一下。如果想更细致的了解前端路由的实现方式，可以阅读 react-router 或者 vue-router 的源码。

# 参考文献

- [「前端进阶」彻底弄懂前端路由](https://juejin.cn/post/6844903890278694919#heading-10) 
