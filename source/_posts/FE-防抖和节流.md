---
title: 防抖和节流
categories:
  - FE
abbrlink: 45b9746d
date: 2019-07-07 13:55:07
---

在某些场景下（scroll、resize 事件等），函数有可能被非常频繁地调用，这样会消耗不必要的性能。为了解决这个问题，引出了防抖（debounce）和节流（throttle）的概念。

# debounce

{% note info %}
在事件被触发时，设定 n 秒后执行对应的回调函数。如果在 n 秒之内事件被重复触发，则重新计时。如果连续触发事件，对应的回调函数可能永远不会被调用。
{% endnote %}

```js
function debounce (fn, timeRange) {
  var timer

  return function (...args) {
    clearTimeout(timer)

    timer = setTimeout(() => {
      fn.call(this, ...args)
    }, timeRange)
  }
}
```

# throttle

{% note info %}
一段时间周期内，最多只执行一次回调函数。
{% endnote %}

```js
function throttle (fn, timeRange) {
  var timer

  return function (...args) {
    if (timer) {
      return
    } else {
      timer = setTimeout(() => {
        clearTimeout(timer)

        timer = null
        fn.call(this, ...args)
      }, timeRange)
    }
  }
}
```

# lodash

在实际工作中给你，往往都是直接引用 [lodash](https://github.com/lodash/lodash)，而 lodash 的 debounce 和 throttle 的规则是可以在进行适当配置的。
