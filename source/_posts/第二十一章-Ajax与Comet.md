---
title: 第二十一章 Ajax与Comet
categories:
  - JavaScript
  - 《JavaScript高级程序设计》
abbrlink: 5ffed448
date: 2019-04-27 12:43:38
---

Ajax是Asynchronous JavaScript + XML的简写，这一技术能够向服务器请求额外的数据而无须卸载页面，会带来更好的用户体验。

Ajax技术的核心是XMLHttpRequest对象（简称XHR），XHR为向服务器发送请求和解析服务器响应提供了流畅的接口。

# XMLHttpRequest对象

## XHR的用法

老规矩，上[**MDN**](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

{% note info %}
XHR的使用流程如下：
1. 首先创建一个XHR对象实例；
2. 使用`open`方法初始化一个请求；
3. 使用`send`方法发送请求。

---
`open(method, url, async, user, password)`方法的参数：
- **method**：一个字符串，用于表示要发送的请求类型，比如：`"get"`、`"post"`；
- **url**：请求的URL；
- **async（可选）**：一个布尔值，用于表示是否发送是异步请求，默认为`true`；
- **user（可选）**：用户名，默认为`null`；
- **password（可选）**：密码，默认为`null`。

---
`send(body)`方法的`body`参数是请求主体要发送的数据，如果不需要通过请求主体发送数据，则传入`null`。

---
在收到响应后，响应的数据会自动填充XHR对象的属性，相关的属性如下：
- **responseText**：作为响应主体被返回的文本；
- **responseXML**：如果响应的内容类型是`"text/xml"`或`"application/xml"`，这个属性中将保存包含着响应数据的XML DOM文档；
- **status**：响应的HTTP状态；
- **statusText**：HTTP状态的说明。

---
如果发送的是异步请求，可以通过检测XHR对象的`readyState`属性来确定请求/响应过程的当前活动阶段。这个属性可取的值如下：
- **0**：未初始化；
- **1**：已初始化但未发送；
- **2**：已发送但未收到响应；
- **3**：已经接收到部分响应数据；
- **4**：已经接收到全部响应数据。

只要`readyState`属性的值发生改变，就会触发readystatechange事件。通常只对`readyState`值为`4`的阶段感兴趣，因为这时所有数据都已经就绪。不过，**必须在调用`open()`方法之前注册`onreadystatechange`事件处理程序，并且在这个事件处理程序中最好不要使用`this`对象，否则可能会出问题。**

---
另外，在接收到响应之前还可以调用`abort()`方法来取消异步请求。调用这个方法后，XHR对象会停止触发事件，而且也不再允许访问与响应有关的对象属性，并且它会将`readyState`值设置为0。
{% endnote %}

说了那么多，上例子：

```js
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // do something
}
xhr.open('get', 'example.txt')
xhr.send(null)
```

## HTTP头部信息

{% note info %}
每个HTTP请求和响应都会带有相应的头部信息，XHR对象提供了操作者两种头部信息的方法。默认情况下，在发送XHR请求的同时，还会发送下列头部信息：
- **Accept**：浏览器能够处理的内容类型；
- **Accept-Charset**：浏览器能够显示的字符集；
- **Accept-Encoding**：浏览器能够处理的压缩编码；
- **Accept-Language**：浏览器当前设置的语言；
- **Connection**：浏览器与服务器之间连接的类型；
- **Cookie**：当前页面设置的任何`cookie`；
- **Host**：发出请求的页面所在域；
- **Referer**：发出请求的页面的URI。注意，HTTP规范将这个头部字段拼写错了，而为保证与规范一致，也只能将错就错了（正确的拼法应该是referrer）；
- **User-Agent**：浏览器的用户代理字符串。

---
使用`setRequestHeader()`方法可以设置自定义的请求头部信息。这个方法接受两个参数：头部字段的名称和头部字段的值。**要成功发送请求头部信息，必须在调用`open()`方法之后且调用`send()`方法之前调用`setRequestHeader()`。**
{% endnote %}

```js
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // do something
}
xhr.open('get', 'example.txt')
xhr.setRequestHeader('MyHeader', 'MyValue')
xhr.send(null)
```

服务器在接收到这种自定义的头部信息之后，可以执行相应的后续操作。我们建议使用自定义的头部字段名称，不要使用浏览器正常发送的字段名称，否则有可能会影响服务器的相应。有的浏览器允许开发人员重写默认的头部信息，但有的浏览器则不允许这样做。

调用XHR对象的`getResponseHeader()`方法并传入头部字段名称，可以取得相应的响应头部信息。而调用`getAllResponseHeaders()`方法则可以取得一个包含所有头部信息的长字符串。

## GET请求

GET请求是最常见的请求类型，最常用于向服务器查询某些信息。必要时，可以将查询字符串参数追加到URL的末尾，以便将信息发送给服务器。对XHR而言，位于传入`open()`方法的URL末尾的查询字符串必须经过正确的编码才行。

使用GET请求经常会发生的一个错误，就是查询字符串的格式有问题。查询字符串中每个参数的名称和值都必须使用`encodeURIComponent()`进行编码，然后才能放到URL的末尾；而且所有键值对儿都必须由和号（`&`）分隔，如下面的例子所示：

```js
xhr.open('get', 'example.php?name1=value1&name2=value2')
```

下面这个函数可以辅助向现有URL的末尾添加查询字符串参数：

```js
function addURLParam(url, name, value) {
    url += (url.indexOf('?') === -1 ? '?' : '&')
    url += encodeURIComponent(name) + '=' + encodeURIComponent(value)
    return url
}
```

## POST请求

POST请求的主体可以包含非常多的数据，而且格式不限。在`open()`方法第一个参数的位置传入`"post"`，就可以初始化一个POST请求。

发送POST请求的第二步就是向`send()`方法中传入某些数据。

# XMLHttpRequest2级

## FormData

现代Web应用中频繁使用的一项功能就是表单数据的序列化，XMLHttpRequest2级为此定义了FormData类型。FormData为序列化表单以及创建与表单格式相同的数据提供了便利。下面的代码创建了一个FormData对象，并向其中添加了一些数据。

```js
var data = new FormData()
data.append('name', 'Nicholas')
```

`append()`方法接收两个参数：键和值，分别对应表单字段的名字和字段中包含的值。可以像这样添加任意多个键值对儿。而通过向FormData构造函数中传入表单元素，也可以用表单元素的数据预先向其中填入键值对儿：

```js
var data = new FormData(document.forms[0])
```

创建了FormData的实例后，可以将它直接传给XHR的`send()`方法，如下所示：

```js
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // do something
}
xhr.open('post', 'postexample.php')
var form = document.getElementById('user-info')
xhr.send(new FormData(form))
```

使用FormData的方便之处体现在不必明确地在XHR对象上设置请求头部。XHR对象能够识别传入的数据类型是FormData的实例，并配置适当的头部信息。

## 超时设定

XHR对象有一个`timeout()`属性，表示请求在等待响应多少毫秒之后就终止。在给`timeout()`设置一个数值后，如果在规定的时间内浏览器还没有接收到响应，那么就会触发timeout事件，进而会调用`ontimeout`事件处理程序。**必须在调用`open()`方法之后且调用`send()`方法之前注册`ontimeout`事件处理程序。**

```js
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // do something
}
xhr.open('get', 'timeout.php')
xhr.timeout = 1000
xhr.ontimeout = function () {
    alert('Request did not return in a second.')
}
xhr.send(null)
```

这个例子示范了如何使用timeout事件，将这个属性设置为1000毫秒，意味着如果请求在1秒钟内还没有返回，就会自动终止。请求终止时，会调用ontimeout事件处理程序。但此时`readyState`可能已经改为`4`了，这意味着会调用`onreadystatechange`事件处理程序。可是，如果在超时终止请求之后再访问`status`属性，就会导致错误。为避免浏览器报告错误，可以将检查`status`属性的语句封装在一个`try-catch`语句中。

## overrideMimeType( )方法

该方法用于重写XHR响应的MIME类型。

```js
var xhr = new XMLHttpRequest()
xhr.open('get', 'text.php')
xhr.overrideMimeType('text/xml')
xhr.send(null)
```

这个例子强迫XHR对象将响应当作XML而非纯文本来处理。**调用`overrideMimeType()`必须在`send()`方法之前，才能保证重写响应的MIME类型。**

# 进度事件

{% note info %}
**以下事件，都需要在调用`open()`方法之前注册相应的事件处理程序。**
- **loadstart**：在接收到响应数据的第一个字节时触发；
- **progress**：在接收响应期间持续不断地触发；
- **error**：在请求发生错误时触发；
- **abort**：在因为调用`abort()`方法而终止连接时触发；
- **load**：在接收到完整的响应数据时触发；
- **loadend**：在通信完成或者触发error、abort或load事件后触发。
{% endnote %}

# CORS

通过XHR实现Ajax通信的一个主要限制，来源于跨域安全策略。默认情况下，XHR对象只能访问与包含它的页面位于同一个域中的资源。这种安全策略可以预防某些恶意行为。但是，实现合理的跨域请求对开发某些浏览器应用程序也是至关重要的。

CORS（Cross-Origin Resource Sharing，跨域资源共享）是W3C的一个工作草案，定义了在必须访问跨域资源时，浏览器与服务器应该如何沟通。CORS背后的基本思想，就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或相应是应该成功，还是应该失败。

比如一个简单的使用GET或POST发送的请求，它没有自定义的头部，而主体内容是`text/plain`。在发送该请求时，需要给它附加一个额外的`Origin`头部，其中包含请求页面的源信息（协议、域名和端口），以便服务器根据这个头部信息来决定是否给予响应。下面是`Origin`头部的一个示例：

```md
Origin: http://www.nczonline.net
```

如果服务器认为这个请求可以接收，就在响应的`Access-Control-Origin`头部中回发相同的源信息（如果是公共资源，可以回发`"*"`）。例如：

```md
Access-Control-Origin: http://www.nczonline.net
```

如果响应没有这个头部，或者有这个头部但源信息不匹配，浏览器就会驳回请求。正常情况下，浏览器会处理请求。注意，请求和响应都不包含`cookie`信息。

现代浏览器都通过XMLHttpRequest对象对象实现了对CORS的源生支持，在尝试打开不同来源的资源时，无需额外编写代码就可以触发这个行为。

{% note info %}
跨域XHR对象也有一些限制，但为了安全这些限制是必需的。以下就是这些限制：
- 不能使用`setRequestHeader()`设置自定义头部；
- 不能发送和接收`cookie`；
- 调用`getAllResponseHeaders()`方法总会返回空字符串。

由于无论同源请求还是跨源请求都使用相同的接口，因此对于本地资源，最好使用相对URL，在访问远程资源时再使用绝对URL。这样做能消除歧义，避免出现限制访问头部或本地`cookie`信息等问题。
{% endnote %}

# Preflight request

