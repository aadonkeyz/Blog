---
title: 事件
categories:
  - FE
abbrlink: 9c2b83ad
date: 2019-04-25 14:31:19
---

JavaScript 与 HTML 之间的交互是通过事件实现的。事件，就是文档或浏览器窗口中发生的一些特定的交互瞬间，比如：点击按钮、拖动鼠标等。

# 事件流

## 事件冒泡

事件冒泡是指事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后逐级向上传播到较为不具体的节点（文档）。以下面的 HTML 页面为例：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Event Bubbling Example</title>
  </head>
  <body>
    <div id="myDiv">Click Me</div>
  </body>
</html>
```

如果你单击了页面中的 `<div>` 元素，那么这个 click 事件会按照如下顺序传播

![事件冒泡](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/FE/%E4%BA%8B%E4%BB%B6%E5%86%92%E6%B3%A1.png)

## 事件捕获

事件捕获的顺序与事件冒泡的顺序正好相反，以前面的HTML页面为例，它的顺序为：

![事件捕获](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/FE/%E4%BA%8B%E4%BB%B6%E6%8D%95%E8%8E%B7.png)

## 事件流

DOM2 级事件规定的事件流包括三个阶段：**事件捕获阶段**、**处于目标阶段**和**事件冒泡阶段**。首先发生的是事件捕获，为截获事件提供了机会。然后是实际的目标接收到事件。最后一个阶段是冒泡阶段，可以在这个阶段对事件做出响应。还是以前面的HTML页面为例，它的顺序如下所示：

![事件流](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/FE/%E4%BA%8B%E4%BB%B6%E6%B5%81.png)

在 DOM 事件流中，实际的目标在捕获阶段不会接收到事件。这意味着在捕获阶段，事件从 `document` 到 `<html>` 再到 `<body>` 后就停止了。下一个阶段是“处于目标”阶段，于是事件在 `<div>` 上发生，并在事件处理（后面将会讨论这个概念）中被看成冒泡阶段的一部分。然后，冒泡阶段发生，事件又传播回文档。

多数支持 DOM 事件流的浏览器都实现了一种特定的行为：即使DOM2 级事件规范明确要求捕获阶段不会涉及事件目标，但 IE9、Safari、Chrome、Firefox 和 Opera9.5 及更高版本都会在捕获阶段触发事件对象上的事件。结果就是有两个机会在目标对象上面操作事件。

# 事件处理程序

事件就是用户或浏览器自身执行的某种动作。诸如 click、load 和 mouseover，都是事件的名字。而响应某个事件的函数就叫做**事件处理程序**（或**事件侦听器**）。事件处理程序的名字就是在事件名前面加上 `"on"`，因此 click 事件的事件处理程序就是 `onclick`，load 事件的事件处理程序就是 `onload`。为事件注册处理程序的方式有好几种，下面一一进行介绍。

## HTML 事件处理程序

某个元素支持的每种事件，都拥有一个与相应事件处理程序同名的 HTML 特性，可以通过这个特性来注册事件处理程序。下面以 `onclick` 事件为例：

```js
// test.js
function showAnother (that) {
  alert(that.value)
}
```
```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
  </head>
  <body>
    <!-- 直接在HTML中指定具体动作 -->
    <input type="button" value="1" onclick="alert(this.value)" />

    <!-- 调用在<script>标签内定义的函数 -->
    <input type="button" value="2" onclick="showMessage(this, event)" />

    <!-- 调用外部js文件中定义的函数 -->
    <input type="button" value="3" onclick="showAnother(this)" />

    <script type="text/javascript" src="./test.js"></script>
    <script type="text/javascript">
      function showMessage (that, event) {
        alert(that.value)
        console.log(that)
        console.log(event)
      }
    </script>
  </body>
</html>
```

在 HTML 中注册事件处理程序，会创建一个封装着元素属性值的函数。这个函数中有一个局部变量 `event`，也就是事件对象（后面将会讨论这个概念），通过 `event` 变量可以直接访问事件对象。并且在这个函数内部，`this` 值等于事件的目标元素。所以你可以将 `event` 和 `this` 当做参数，传递给要调用的函数。关于这一点，你可以查看上面例子中 `showMessage()` 函数打印的内容来验证。

{% note info %}
HTML 事件处理程序的缺点：
- 如果用户在页面解析 `showMessage()` 和 `showAnother()` 之前就点击了对应的按钮，会抛出错误。可以使用 `onclick="try {showMessage()} catch(ex) {}"` 的形式来解决这个问题。
- 事件处理程序的作用域链在不同的浏览器中会有不同的结果。
- HTML 和 JS 代码紧密耦合。
{% endnote %}

## DOM0 级事件处理程序

{% note warning %}
- 使用 DOM0 级方法只能在元素上注册一个事件处理程序。
- 对于相同的事件，DOM0 级事件处理程序与 HTML 事件处理程序事件处理程序无法共存。
- 使用 DOM0 级方法注册的事件处理程序被认为是元素的方法。因此，这时候的事件处理程序是在元素的作用域中运行，所以此时函数中的 `this` 指向元素自身。
- 在注册事件处理程序时，不推荐使用 [箭头函数](https://aadonkeyz.com/posts/9595c646/#箭头函数)，因为它会造成 `this` 的丢失。
{% endnote %}

在添加的事件处理程序函数内部，可以直接通过 `event` 变量访问事件对象。也可以通过给程序处理函数定义参数或者使用 `arguments` 来访问事件对象，在下面有例子。

以这种方式添加的事件处理程序会在事件流的冒泡阶段被处理。你可以通过将事件处理程序属性的值设置为 `null` 来删除添加的事件处理程序。

来看一个例子：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
  </head>
  <body>
    <input id="myButton" type="button" value="1" />
    <script type="text/javascript">
      var button = document.getElementById('myButton')
      button.onclick = function (e) {
        console.log(this)
        console.log(event)
        
        // true
        console.log(event === e)    

        // true
        console.log(event === arguments[0]) 

        // 删除添加的事件处理程序
        button.onclick = null
      }
    </script>
  </body>
</html>
```

这个例子中的按钮，只有第一次点击时会打印内容，之后就没有任何反应，因为事件处理程序在第一次触发之后，就被删除了。

## DOM2 级事件处理程序

DOM2 级事件定义了两个方法，`addEventListener()` 和 `removeEventListener()`，分别用于注册和删除事件处理程序，所有 DOM 节点都包含这两个方法。

{% note warning %}
- 使用 `addEventListener()` 可以在同一个元素上注册多个事件处理程序，触发的顺序为添加顺序。
- 关于 `this` 和 `event` 的使用规则，与 DOM0 级事件处理程序一致。
{% endnote %}

{% note info %}
首先介绍 `addEventListener()` 方法，它的参数如下：
- `type`：表示监听事件类型的字符串，**需要注意的是没有 `on` 前缀**。
- `listener`：作为事件处理程序的函数。
- `options（可选）`：一个对象。其属性如下：
  - `capture`：一个布尔值，默认为 `false`。当值为 `true` 时，`listener` 会在事件捕获阶段时被调用。
  - `once`：一个布尔值，默认为 `false`。当值为 `true` 时，`listener` 会在其被调用之后自动移除。
  - `passive`：一个布尔值，默认为 `false`。当值为 `true` 时，`listener` 内部不允许调用 `event.preventDefault()`，否则会抛出错误。
- `useCapture（可选）`：一个布尔值，默认为 `false`。当值为 `true` 时，`listener` 会在事件捕获阶段时被调用。

**对于 `options` 和 `useCapture` 参数，它们都是该方法的第三个参数，`options` 是新标准，而 `useCapture` 是老标准。**

---
接着介绍 `removeEventListener()` 方法，它的参数如下：
- `type`：表示监听事件类型的字符串，**需要注意的是没有 `on` 前缀**。
- `listener`：作为事件处理程序的函数。
- `options（可选）`：一个对象。其属性如下：
  - `capture`：一个布尔值，默认为 `false`。当值为 `true` 时，表示要移除的 `listener` 是注册在事件捕获阶段的。
- `useCapture（可选）`：一个布尔值，默认为 `false`。当值为 `true` 时，表示要移除的 `listener` 是注册在事件捕获阶段的。

**如果一个事件处理程序一共注册了两次，一次在事件捕获阶段，一次在事件冒泡阶段，那么这两次注册需要分别移除，两者不会互相干扰。**
{% endnote %}

下面的例子用于观察 `options.capture` 和 `useCapture` 的效果。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>test</title>
    <style>
      #outer, #inner {
        display: block;
        width: 500px;
        height: 500px;
        text-decoration: none;
      }
      #outer{
        border: 1px solid red;
        color: red;
      }
      #inner{
        border: 1px solid green;
        color: green;
        width: 250px;
        height: 250px;
        margin: 125px auto;
      }
    </style>
  </head>
  <body>
    <div id="outer">
      outer, capture & none-capture
      <div id="inner">
        inner
      </div>
    </div>
    <script type="text/javascript">
      var outer = document.getElementById('outer')
      var inner = document.getElementById('inner')

      function captureListener1 () {
        console.log('outer, capture1')
        outer.removeEventListener('click', captureListener1, true)
      }
      function captureListener2 () {
        console.log('outer, capture2')
      }
      function noneCaptureListener () {
        console.log('outer, none-capture')
      }
      function innerListener () {
        console.log('inner')
      }

      outer.addEventListener('click', captureListener1, { capture: true })
      outer.addEventListener('click', captureListener2, true)
      outer.addEventListener('click', noneCaptureListener)
      inner.addEventListener('click', innerListener)
    </script>
  </body>
</html>
```

上例中 `captureListener1` 和 `captureListener2` 都是注册在 `outer` 的捕获阶段，而 `noneCaptureListener` 和 `innerListener` 分别注册在 `outer` 和 `inner` 的冒泡阶段。并且 `captureListener1` 会在第一次调用后被移除。请多点击几次 inner 框，查看打印的结果。

# 事件对象

在触发 DOM 上的某个事件时，会产生一个事件对象 `event`，这个对象中包含着所有与事件有关的信息。包括导致事件的元素、事件的类型以及其他与特定事件相关的信息。例如，鼠标操作导致的事件对象中，会包含鼠标位置信息，而键盘操作导致的事件对象中，会包含与按下的键有关的信息。这里只介绍 DOM 中的事件对象，忽略 IE 的。

无论注册事件处理程序时使用的是 DOM0 级还是 DOM2 级方法，兼容 DOM 的浏览器都会将一个 `event` 对象传入到事件处理程序中，这样就可以直接在函数内部访问到 `event` 对象了。

`event` 对象包含与创建它的特定事件有关的属性和方法。触发的事件类型不一样，可用的属性和方法也不一样。下面简单的对事件对象的属性和方法进行了介绍，如果想查看详细信息，去移步 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Event)。

{% note info %}
属性：
- `bubbles`（只读）：表明事件是否会冒泡。
- `cancelBubble`：通过将该属性设置为 `true` 可阻止事件继续冒泡。
- `cancelable`（只读）：表明是否可以取消事件的默认行为。
- `currentTarget`（只读）：事件处理程序注册在哪个元素上，`currentTarget` 就指向哪个元素。
- `defaultPrevented`（只读）：表明是否已经调用了 `preventDefault()` 方法。
- `eventPhase`（只读）：表明处于事件流的哪个阶段。`1` 表示捕获阶段，`2` 表示处于目标，`3` 表示冒泡阶段。
- `target`（只读）：触发事件的那个元素，也就是事件流在“处于目标”阶段时的那个目标元素。
- `timeStamp`（只读）：表明事件对象的创建时间。
- `type`（只读）：表明事件对象的类型。
- `isTrusted`（只读）：当事件是由用户触发的时（比如点击鼠标），该属性值为 `true`。当事件是由脚本触发时，该属性值为 `false`。

---
方法：
- `preventDefault()`：取消事件的默认行为。该方法只有在 `cancelable` 属性为 `true` 时才会起作用。
- `stopImmediatePropagation()`：取消事件的进一步捕获或冒泡，同时阻止其后的所有事件处理程序被调用。
- `stopPropagation()`：取消事件的进一步捕获或冒泡，但是不会阻止注册在当前 `currentTarget` 上的事件处理程序被调用。
{% endnote %}

# 事件类型
{% note info %}
下面我只简单的介绍一下我认为比较常用的事件，如果你想比较全面的了解这里，点击下面的链接！

[MDN，只有你想不到，没有找不到](https://developer.mozilla.org/en-US/docs/Web/Events)
{% endnote %}

# 内存和性能

在 JavaScript 中，添加到页面上的事件处理程序数量将直接关系到页面的整体运行性能。导致这一问题的原因是多方面的。首先，每个函数都是对象，都会占用内存。内存中的对象越多，性能就越差。其次，必须事先指定所有事件处理程序而导致的 DOM 访问次数，会延迟整个页面的交互就绪时间。

## 事件委托

对“事件处理程序过多”问题的解决方案就是**事件委托**。事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。例如，click 事件会一直冒泡到 `document` 。也就是说，我们可以为整个页面指定一个 `onclick` 事件处理程序，而不必给每个可单击的元素分别添加事件处理程序。

## 移除事件处理程序

每当将事件处理程序指定给元素时，运行中的浏览器代码与支持页面交互的 JavaScript 代码之间就会建立一个连接。这种连接越多，页面执行起来就越慢。如前所述，可以采用事件委托技术，限制建立的连接数量。另外，在不需要的时候移除事件处理程序，也是解决这个问题的一种方案。内存中留有那些过时不用的“空事件处理程序”，也是造成 Web 应用程序内存与性能问题的主要原因。

在两种情况下，可能会造成上述问题。第一种情况就是从文档中移除带有事件处理程序的元素时。这可能是通过纯粹的 DOM 操作，例如使用 `removeChild()` 方法，但更多地是发生在使用 `innerHTML` 替换页面中某一部分的时候。如果带有事件处理程序的元素被 `innerHTML` 删除了，那么原来添加到元素中的事件处理程序极有可能无法被当作垃圾回收。**所以如果你知道某个元素即将被移除，那么最好在此之前手工移除事件处理程序。**

另一种情况，就是卸载页面的时候。如果在页面被卸载之前没有清理干净事件处理程序，那它们就会滞留在内存中。每次加载完页面在卸载页面时（可能是在两个页面间来回切换，也可能是单击了“刷新”按钮），内存中滞留的对象数目就会增加，因为事件处理程序占用的内存并没有被释放。**因此最好的做法就是在页面卸载之前，先通过 `onunload` 事件处理程序移除所有事件处理程序。**
