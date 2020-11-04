---
title: Promise
categories:
  - JavaScript
abbrlink: 9a3eeeca
date: 2019-04-12 10:26:31
---

# Promises/A+

本来想仔细描述一下 Promises 的各种特性的，但由于翻译水平有限，最后觉得还是直接查看 Promises/A+ 的规范更精确。

所以，请先移步 [Promises/A+](https://promisesaplus.com/#point-49)。

{% note warning %}
- 规范中提到“then may be called multiple times on the same promise”，它的意思是同一个 `Promise` 实例上可以挂载多个 `then`，而不是同一个 `then` 被调用多次。
- 规范中说的 object 是指 `Object.prototype.toString.call(value)` 返回值为 `"[object Object]"` 的数据，而不是广义上的 Object 的实例。
- 与 `resolve()` 的复杂规则不同。调用 `reject(value)` 时，不论传给它的值是什么，它都会将 `Promise` 实例的状态置为 `rejected`，将 `Promise` 实例的值设置为 `value`。
{% endnote %}

# Promises/A+ 的补充说明

{% note info %}
在根据 Promises/A+ 实现 Promise 时，各种引擎又添加了如下规则：
- 如果在 `Promise` 构造器中抛出了一个错误，则会 `reject(err)`。
- 规范中的 `resolve` 和 `reject` 函数，既可以在 `Promise` 构造器中的使用，也可以单独使用，如 `Promise.resolve()` 和 `Promise.reject()`。这样做的相当于直接创建了一个 `fulfilled` 或 `rejected` 状态的 `Promise` 实例，跳过了前序的 `pending` 状态。这样做的好处是可以很方便的将 thenable 类型数据转换为标准的 `Promise` 实例。
- 可能是为了实现方便，如果传递给 `resolve` 的是一个 `Promise` 实例，它会原封不动的将该 `Promise` 实例返回。
- `Promise.prototype.catch(onRejected)` 等价于 `Promise.prototype.then(null, onRejected)`。
- `Promise.prototype.finally(onFinally)` 中的 `onFinally()` 不接受任何参数，它会在 `Promise` 实例被决议且 execution context stack 为空时被调用，但它会原封不动的将调用它的那个 `Promise` 实例作为返回值。**它是 ES8 中引入中，所以可能会存在一些兼容性问题。**
- 与 `Promise.prototype.then` 一样，同一个 `Promise` 实例上也可以挂在多个 `Promise.prototype.catch` 和 `Promise.prototype.finally`。
{% endnote %}

基础用法如下，详细的可以参考 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

```js
const promise = new Promise(function (resolve, reject) {
  // some code

  if (/* 异步操作成功 */) {
    resolve('success');
  } else {
    reject('fail');
  }
});
```

# thenable

{% note warning %}
在按照规范实现 Promise 的同时，也实现了许多额外的方法，如 `Promise.all`、`Promise.race` 等。这些方法同样兼容 thenable。
{% endnote %}

# 全局的 Promise 拒绝处理

Promise 最有争议的方面之一就是：当一个 `Promise` 实例被拒绝时若缺少拒绝处理函数，就会静默失败。有人认为这是规范中最大的缺陷，因为这是 JS 语言所有组成部分中唯一不让错误清晰可见的。

由于 Promise 的本质，判断一个 `Promise` 实例的拒绝是否已被处理并不直观。例如，研究以下示例：

```js
let rejected = Promise.reject(42)

// 在此刻 rejected 不会被处理

// 一段时间后……
rejected.catch(function (value) {
  // 现在 rejected 已经被处理了
  console.log(value)
})
```

无论 `Promise` 实例是否已被决议，你都可以在任何时候调用 `then()` 或 `catch()` 并使它们正确工作，这导致很难准确知道一个 `Promise` 实例何时会被处理。

虽然下个版本的 ES 可能会处理此问题，不过 Node.js 与浏览器已经实施了变更来解决开发者的这个痛点。这些变更不是 ES6 规范的一部分，但却是使用 Promise 时的宝贵工具。

## Node.js 的拒绝处理

在 Node.js 中，`process` 对象上存在两个关联到 `Promise` 实例的拒绝处理的事件：

{% note info %}
- `unhandledRejection`：当一个 `Promise` 实例被拒绝、而在事件循环本轮次中没有任何拒绝处理函数被调用时，该事件就会被触发。
- `rejectionHandled`：当一个 `Promise` 实例被拒绝、并从事件循环的下一个轮次开始才有拒绝处理函数被调用时，该事件就会被触发。
{% endnote %}

这两个事件旨在共同帮助识别已被拒绝但未曾被处理的 `Promise` 实例。

`unhandledRejection` 事件处理函数接受的参数是拒绝原因（常常是一个错误对象）以及被拒绝的 Promise。以下代码展示了 `unhandledRejection`的应用：

```js
let rejected

process.on('unhandledRejection', function (reason, promise) {
  // Explosion!
  console.log(reason.message)             

  // true
  console.log(rejected === promise)       
})

rejected = Promise.reject(new Error('Explosion!'))
```

`rejectionHandled` 事件处理函数则只有一个参数，即已被拒绝的 `Promise` 实例。例如：

```js
let rejected

process.on('rejectionHandled', function (promise) {
  // true
  console.log(rejected === promise)       
})

rejected = Promise.reject(new Error('Explosion!'))

// 延迟添加拒绝处理函数
setTimeout(function() {
  rejected.catch(function (value) {
    // Explosion!
    console.log(value.message)          
  })
}, 1000)
```

此处的 `rejectionHandled` 事件在拒绝处理函数最终被调用时触发。若在 `rejected` 被创建后直接将拒绝处理函数附加到它上面，那么此事件就不会被触发。因为立即附加的拒绝处理函数在 `rejected` 被创建的事件循环的同一个轮次内就会被调用，这样 `rejectionHandled` 就不会起作用。

为了正确追踪潜在的未被处理的拒绝，使用 `unhandledRejection` 与 `rejectionHandled` 事件就能保持包含这些 `Promise` 实例的一个列表，之后等待一段时间再检查此列表。例如：

```js
let possiblyUnhandledRejections = new Map()

// 当一个拒绝未被处理，将其添加到 map
process.on('unhandledRejection', function (reason, promise) {
  possiblyUnhandledRejections.set(promise, reason)
})

process.on('rejectionHandled', function (promise) {
  possiblyUnhandledRejections.delete(promise)
})

setInterval(function () {
  possiblyUnhandledRejections.forEach(function (reason, promise) {
    console.log(reason.message ? reason.message : reason)

    // 做点事来处理这些拒绝
    handleRejection(promise, reason)
  })
  possiblyUnhandledRejections.clear()
}, 60000)
```

## 浏览器的拒绝处理

浏览器同样能触发两个事件，来帮助识别未处理的拒绝。这两个事件会被 `window` 对象触发，并完全等效于 Node.js 的相关事件：

{% note info %}
- `unhandledrejection`：当一个 `Promise` 实例被拒绝、而在事件循环的本轮次中没有任何拒绝处理函数被调用时，该事件就会被触发。
- `rejectionhandled`：当一个 `Promise` 实例被拒绝、并从事件循环的下一个轮次开始才有拒绝处理函数被调用时，该事件就会被触发。
{% endnote %}

Node.js 的实现会传递分离的参数给事件处理函数，而浏览器的两个事件处理函数则只会接收到包含下列属性的一个对象：

{% note info %}
- `type`：事件的名称（`unhandledrejection` 或 `rejectionhandled`）。
- `promise`：被拒绝的 `Promise` 实例。
- `reason`：`Promise` 实例的值。
{% endnote %}

除了事件处理函数接收的参数有所区别外，浏览器的事件处理函数用法可以仿照 Node.js。
