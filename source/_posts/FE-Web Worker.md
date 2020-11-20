---
title: Web Worker
categories:
  - FE
abbrlink: da19b401
date: 2019-08-31 12:01:51
---

# 特点

Web Worker 是让 Web 内容在后台线程中运行脚本的一种简单方法。Worker 线程在执行任务期间不会受到用户交互的干扰，除此之外，还可以使用 XMLHttpRequest 执行 I/O 操作。

{% note info %}
- Worker 线程运行环境不同于主线程，所以在 Worker 线程中无法使用 `document`、`window` 和 `parent` 这些对象，但是可以使用 `navigator` 对象和 `location` 对象。
- Worker 线程中，`this` 对象和 `self` 对象是相同的。
- Worker 线程不允许读取本地文件，即不能打开本机的文件系统 `file://`。这样说来，在使用 `new Worker()` 创建 Worker 线程时也不允许使用本机文件，在学习过程中为了方便，可以使用 `window.URL.createObjectURL(new Blob())` 来绕过这一限制。
- Web Worker 分为专用 Worker 和共享 Worker。
- 在主线程和 Worker 线程之间，通过 `postMessage()` 来发送数据，通过 message 事件来接收数据。这个过程中数据是被深拷贝的，所以不会彼此干扰。
- 在主线程中，使用 `terminate()` 中止 Worker 线程。
- 在 Worker 线程中，可以使用 `importScript()` 加载脚本。
- Worker 线程运行出错时，会触发 error 事件。该事件不会冒泡并且可以被取消。
{% endnote %}

# demo

```html
<script type="text/javascript">
  if (Worker) {
    function worker_function () {
      self.addEventListener('message', function (event) {
        console.log(event.data);
        postMessage('from work thread');
      })
    }

    const myWorker = new Worker(
      URL.createObjectURL(
        new Blob(
          ['('+worker_function.toString()+')()'],
          {type: 'text/javascript'}
        )
      )
    );

    myWorker.postMessage('from main thread');
    myWorker.addEventListener('message', function () {
      console.log(event.data);

      myWorker.terminate();
    })
  }
</script>
```
