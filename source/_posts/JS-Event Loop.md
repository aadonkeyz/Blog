---
title: Event Loop
categories:
  - JavaScript
abbrlink: 73ea086f
date: 2019-06-22 15:53:34
---

# 正文

JavaScript 是以单线程方式运行的，也就是说同一时刻只能做一件事，后面的代码必须等待前面的代码运行完成后才能开始执行。但是 JavaScript 是一个解释型语言，它的运行依赖于宿主环境（浏览器、Node），而这些环境都为 JavaScript 提供了一个叫做 event loop 的并发模型。

先用一个简单的例子来展示 event loop 是如何工作的

```js
console.log('Hi');

setTimeout(function cb () {
  console.log('there');
}, 1000);

console.log('JS');

// Hi
// JS
// there
```

![event loop](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/event%20loop.png)

{% note info %}
1. 执行 `console.log('Hi')`。
2. 执行 `setTimeout`，它会调用专门的 api，负责在定时器完成后将 `cb` 函数推到 task queue 中。
3. 执行 `console.log('JS')`。
4. 当 execution context stack 为空且 task queue 中有 task 时，将 task 推到 stack 中去执行。
5. 执行 `console.log('there')`。
{% endnote %}

{% note warning %}
上面介绍的比较粗略，实际上除了 task，还有一个 microtask。因此，除了 task queue，还有一个 microtask queue。他们有如下区别：
- When executing tasks from the task queue, the runtime executes each task that is in the queue at the moment a new iteration of the event loop begins. Tasks added to the queue after the iteration begins will not run until the next iteration.
- Each time a task exits, and the execution context stack is empty, each microtask in the microtask queue is executed, one after another. The difference is that execution of microtasks continues until the queue is empty—even if new ones are scheduled in the interim. In other words, microtasks can enqueue new microtasks and those new microtasks will execute before the next task begins to run, and before the end of the current event loop iteration.

---
同样都是 task 或 microtask，优先级也是有差别的：
1. **microtask（优先级递减）**：
  1. `process.nextTick`
  2. `Promise.then`、`Promise.catch`、`Promise.finally`
  3. `Object.observe`
  4. `MutationObserver`
2. **task（优先级递减）**：
  1. `setTimeout`
  2. `setInterval`
  3. `setImmediate`
  4. `I/O`
  5. `UI rendering`
{% endnote %}

# 参考文献
- [What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
- [Concurrency model and the event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
- [In depth: Microtasks and the JavaScript runtime environment](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth)
- [Using microtasks in JavaScript with queueMicrotask()](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)
