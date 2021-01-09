---
title: 浏览器的渲染原理
categories:
    - 待整理
abbrlink: b5fc6f17
date: 2019-07-31 19:41:24
---

# 浏览器的进程和线程

{% note info %}
浏览器的主要进程：
1. **Browser进程**：浏览器的主进程，只有一个。作用为：
    - 负责浏览器界面显示、与用户的交互。
    - 负责各个标签页的管理、创建和关闭。
    - 将Renderer进程得到的内存中的Bitmap，绘制到用户界面上
    - 网络资源的管理、下载等。
2. **插件进程**：每种类型的插件对应一个插件进程，仅当使用该插件时才创建对应的插件进程。
3. **GPU进程**：最多一个，用于3D绘制等。
4. **Renderer进程**：又叫作**浏览器内核**，内部有多个线程。默认每个标签页有一个Renderer进程，互不影响。浏览器的优化机制在某些情况下会将多个标签页的Renderer进程合并为一个。

---
Renderer进程的主要线程：
1. **GUI渲染线程**：负责解析HTML和CSS。
2. **JS引擎线程**：负责执行JavaScript代码，**JS引擎线程与GUI渲染线程互斥，且JS引擎线程的优先级高。**
3. **事件触发线程**：任务队列和交互事件都是它说了算。
4. **定时触发器线程**：管理`setTimeout`和`setInterval`。
5. **异步http请求线程**：异步http请求收到响应后，就是通过它将对应回调函数放在任务队列中，然后再交给js引擎线程去执行。
{% endnote %}

# 输入一个url，浏览器都做了什么

{% note info %}
1. Browser进程收到url，加载资源，然后将资源传递给该标签页对应的Renderer进程。
2. Renderer进程开始分工：
    - GUI渲染线程解析HTML和CSS，分别生成DOM和CSSOM，然后合并为Rendering Tree。
    - GUI渲染线程根据Rendering Tree计算布局，然后渲染页面。
    - JS引擎线程执行JavaScript代码。因为这一过程可能会改变DOM和CSSOM，所以JS引擎线程和GUI渲染线程被设计为互斥的，且JS引擎线程优先级高。
3. Renderer进程渲染页面的过程中，可能需要GPU进程来完成3D绘制。
{% endnote %}

# 注意事项

{% note info %}
- 为了更好的用户体验，GUI渲染线程将会尽可能早的将内容呈现到屏幕上，并不会等到所有的HTML和CSS都解析完成之后再去构建Rendering Tree和计算布局。它是解析完一部分内容就显示一部分内容，同时，可能还在通过网络下载其余内容。
- reflow一定repaint，repaint不一定reflow。
{% endnote %}

# 参考文献

- [**从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理**](https://zhuanlan.zhihu.com/p/33230026)
- [**浏览器渲染原理及流程**](https://www.cnblogs.com/slly/p/6640761.html)


<!-- aadonkeyz -->