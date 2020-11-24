---
title: Node Event Loop
categories:
  - Node
abbrlink: 1bc69df4
date: 2020-11-21 12:37:04
---

下面两段代码，你能说出正确的打印顺序分别是什么吗？

```js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});

// 无法确定打印顺序
```

```js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});

// 先 immediate
// 后 timeout
```

{% note info %}
不是我懒惰，是我怕翻译错
so
[**Node 官方解读在这里**](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
{% endnote %}

下面是写的比较好的博客，它们是一个系列哦：
- [Event Loop and the Big Picture — NodeJS Event Loop Part 1](https://blog.insiderattack.net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb67a182810)
- [Timers, Immediates and Process.nextTick — NodeJS Event Loop Part 2](https://blog.insiderattack.net/timers-immediates-and-process-nexttick-nodejs-event-loop-part-2-2c53fd511bb3)
- [Promises, Next-Ticks, and Immediates — NodeJS Event Loop Part 3](https://blog.insiderattack.net/promises-next-ticks-and-immediates-nodejs-event-loop-part-3-9226cbe7a6aa)
- [Handling IO — NodeJS Event Loop Part 4](https://blog.insiderattack.net/handling-io-nodejs-event-loop-part-4-418062f917d1)
- [New Changes to the Timers and Microtasks in Node v11.0.0 (and above)](https://blog.insiderattack.net/new-changes-to-timers-and-microtasks-from-node-v11-0-0-and-above-68d112743eb3)
- [Event Loop Best Practices — NodeJS Event Loop Part 5](https://blog.insiderattack.net/event-loop-best-practices-nodejs-event-loop-part-5-e29b2b50bfe2)
