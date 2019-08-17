---
title: requestAnimationFrame
categories:
    - 前端
abbrlink: 5135f46c
date: 2019-08-17 22:59:49
---

CSS 动画的优势在于浏览器知道动画什么时候开始，因此会计算出正确的时间间隔，在恰当的时候刷新 UI。而对于 JavaScript 动画，浏览器无从知晓什么时候开始。

针对这一问题的解决方案就是`window.requestAnimationFrame()`。该方法接收一个回调函数作为参数，并告诉浏览器要在下一次 repaint 之前调用该回调函数。它的使用形式通常如下所示：

```js
function draw () {
    // do something

    requestAnimationFrame(draw)
}

requestAnimationFrame(draw)
```

这样就可以在浏览器每一次 repaint 前都调用对应的方法，保证了 JavaScript 动画的刷新频率与浏览器的刷新频率相匹配，频率通常是 60Hz。
