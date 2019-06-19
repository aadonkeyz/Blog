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

只要`readyState`属性的值发生改变，就会触发readystatechange事件。通常只对`readyState`值为`4`的阶段感兴趣，因为这时所有数据都已经就绪。不过，**必须在调用`open()`方法之前注册`onreadystatechange`事件处理程序才能确保跨浏览器兼容性，并且在这个事件处理程序中最好不要使用`this`对象，否则可能会出问题。**

---
另外，在接收到响应之前还可以调用`abort()`方法来取消异步请求。调用这个方法后，XHR对象会停止触发事件，而且也不再允许访问与响应有关的对象属性，并且它会将`readyState`值设置为`0`。
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

# CORS跨域

## CORS

通过XHR实现Ajax通信的一个主要限制，来源于跨域安全策略（同源策略）。默认情况下，使用XHR对象发送请求的URL域名必须和当前页面完全一致。完全一致的意思是，域名要相同（`www.example.com`和`example.com`不同），协议要相同（`http`和`https`不同），端口号也要相同。有的浏览器比较宽松，允许端口不同，大多数浏览器都会严格遵守这个限制。这种安全策略可以预防某些恶意行为。但是，实现合理的跨域请求对开发某些浏览器应用程序也是至关重要的。

CORS（Cross-Origin Resource Sharing，跨域资源共享）是W3C的一个工作草案，定义了在必须访问跨域资源时，浏览器与服务器应该如何沟通。CORS背后的基本思想，就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或相应是应该成功，还是应该失败。

比如一个简单的使用GET或POST发送的请求，它没有自定义的头部，而主体内容是`text/plain`。在发送该请求时，需要给它附加一个额外的`Origin`头部，其中包含请求页面的源信息（协议、域名和端口），以便服务器根据这个头部信息来决定是否给予响应。下面是`Origin`头部的一个示例：

```md
Origin: http://www.nczonline.net
```

如果服务器认为这个请求可以接收，就在响应的`Access-Control-Allow-Origin`头部中回发相同的源信息（如果是公共资源，可以回发`"*"`）。例如：

```md
Access-Control-Allow-Origin: http://www.nczonline.net
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

## Preflight request

参考资料：[**前端 | 浅谈preflight request**](https://www.jianshu.com/p/b55086cbd9af)

想象一个场景，我们发送一个POST跨域请求，服务器收到请求后对数据库进行了相应的操作并返回响应，但是由于浏览器的跨域限制，导致我们收到的是请求失败的结果。这种情况就是明明用户请求的操作成功了，但是用户不知道他成功了。

预检请求（Preflight request）就是为了解决上述问题的。某些情况下浏览器在发送跨域请求之前会先发送一个相应的预检请求，从而获知服务器是否允许该跨域请求。如果允许，就发送跨域请求。如果不允许，则阻止发送跨域请求。

{% note info %}
触发预检请求的条件：
- 如果请求方法不是GET、HEAD或POST，那么将发送预检请求；
- 如果人为设置了对CORS安全的首部字段集合之外的首部字段，那么将发送预检请求。对CORS安全的首部字段集合包含：
    1. **Accept**
    2. **Accept-Language**
    3. **Content-Language**
    4. **Content-Type**
    5. **DPR**
    6. **Downlink**
    7. **Save-Data**
    8. **Viewport-Width**
    9. **Width**
- 如果请求头的`Content-Type`的值不是`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`之一的话，那么将发送预检请求。

---
预检请求使用的是OPTIONS方法，发送下列头部：
- **Origin**：源信息；
- **Access-Control-Allow-Method**：跨域请求使用的方法；
- **Access-Control-Allow-Headers（可选）**：跨域请求会额外发送的头部字段，这些头部字段用逗号分隔。

---
预检请求的响应会包含如下头部字段：
- **Access-Control-Allow-Origin**：源信息；
- **Access-Control-Allow-Methods**：允许的方法，多个方法以逗号分隔；
- **Access-Control-Allow-Headers**：允许的头部，多个头部以逗号分隔；
- **Access-Control-Max-Age（可选）**：应该将这个预检请求缓存多长时间（以秒表示），也就是说在这个时间范围内，不会再发送相同的预检请求。
{% endnote %}

## 带凭据的请求

默认情况下，跨域请求不提供凭据（`cookie`、HTTP认证及客户端SSL证明等）。通过将`withCredentials`属性设置为`true`，可以指定某个请求应该发送凭据。如果服务器接受带凭据的请求，会用下面的HTTP头部来响应：

```md
Access-Control-Allow-Credentials: true
```

如果发送的是带凭据的请求，但服务器的响应中没有包含这个头部，那么浏览器就不会把响应交给JavaScript（于是，`responseText`中将是空字符串，`status`的值为`0`，而且会调用`onerror()`事件处理程序）。另外，服务器还可以在预检请求的响应中发送这个HTTP头部，表示允许源发送带凭据的请求。

# 其他跨域技术

## 图像Ping

图像Ping是与服务器进行简单、单向的跨域通信的一种方式。请求的数据是通过查询字符串形式发送的，而响应可以是任意内容，但通常是像素图或204响应。通过图像Ping，浏览器得不到任何具体的数据，但通过侦听load和error事件，它能知道响应是什么时候接收到的。来看下面的例子：

```js
var img = new Image()
img.onload = img.onerror = function () {
    alert('Done!')
}
img.src = 'http://www.example.com/test?name=Nicholas'
```

图像Ping最常用于跟踪用户点击页面或动态广告曝光次数。图像Ping有两个主要缺点，一是只能发送GET请求，二是无法访问服务器的响应文本。因此，图像Ping只能用于浏览器与服务器的单向通信。

## JSONP

JSONP是JSON with padding（填充式JSON或参数式JSON）的简写，是应用JSON的一种新方法，在后来的Web服务中非常流行。JSONP看起来与JSON差不多，只不过是被包含在函数调用中的JSON，就像下面这样：

```js
handleResponse({ "name": "Nicholas" })
```

JSONP由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数，回调函数一般是在请求中通过`callback`查询字段指定的。而数据就是传入回调函数中的JSON数据。下面是一个典型的JSONP请求：

```md
http://aadonkeyz.com/example.js?callback=handleResponse
```

JSONP的原理很简单，它是通过动态`<script>`元素来使用的，通过下面的例子进行介绍：

```js
function handleResponse(response) {
    // do something
}

var script = document.createElement('script')
script.src = 'http://aadonkeyz.com/example.js?callback=handleResponse'
document.body.insertBefore(script, document.body.firstChild)
```

{% note info %}
这个例子的执行顺序为：
1. 浏览器通过`<script>`标签发送请求；
2. 服务器收到请求后，发现这个是JSONP的请求，根据请求的`callback`查询字段获取回调函数的名称，然后准备要返回的JSON数据，最后以JSONP的格式结合回调函数名称和JSON数据；
3. 浏览器接收到JSONP的响应，进行函数调用，即`handleResponse({ "name": "Nicholas" })`。
{% endnote %}

与图像Ping相比，JSONP非常简单易用并且能够直接访问响应文本，支持在浏览器与服务器之间双向通信。不过它也有自己的缺点：

首先，JSONP是从其他域中加载代码执行，如果其他域不安全，很可能会在响应中夹带一些恶意代码，而此时除了完全放弃JSONP调用之外，没有办法追究。因此在使用不是你自己运维的Web服务时，一定要保证它安全可靠。

其次，要确定JSONP请求是否失败并不容易。但是HTML5给`<script>`标签新增了一个`onerror`事件处理程序，现在来说这个应该不算缺点了吧......

## Comte

Comte是Alex Russell发明的一个词儿，指的是一种更高级的Ajax技术（经常也有人称为“服务器推送”）。**Ajax是一种从页面向服务器请求数据的技术，而Comte则是一种服务器向页面推送数据的技术。Comte能够让信息近乎实时地被推送到页面上。**

有两种实现Comte的方式：**长轮询**和**流**。

{% note info %}
短轮询是浏览器定时向服务器发送请求，查看是否有更新的数据。

长轮询是在页面发起一个到服务器的请求，然后服务器一直保持连接打开，直到有数据可发送。发送完数据之后，浏览器关闭连接，随即又发起一个到服务器的新请求。这一过程在页面打开期间一直持续不断。

无论是短轮询还是长轮询，浏览器都要在接收数据之前，先发起对服务器的链接。两者最大的区别在于服务器如何发送数据。短轮询是服务器立即发送响应，无论数据是否有效，而长轮询是等待发送响应。
{% endnote %}

![21-1长短轮询时间线](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8AJavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E3%80%8B/21-1%E9%95%BF%E7%9F%AD%E8%BD%AE%E8%AF%A2%E6%97%B6%E9%97%B4%E7%BA%BF.png)

{% note info %}
第二种流行的Comet实现是HTTP流。流不同于上述两种轮询，因为它在页面的整个生命周期内只使用一个HTTP连接。具体来说，就是浏览器向服务器发送一个请求，而服务器保持连接打开，然后周期性地向浏览器发送数据。

在浏览器中，通过注册`onreadystatechange`事件处理程序及检测`readyState`的值，就可以利用XHR对象实现HTTP流。随着不断从服务器接收数据，`readyState`的值会周期性地变为`3`，而浏览器将接收的所有数据均保存在`responseText`属性中。
{% endnote %}

使用XHR对象实现HTTP流的典型代码如下所示：

```js
function createStreamingClient(url, progress, finished) {
    var xhr = new XMLHttpRequest(),
        received = 0
    
    xhr.onreadystatechange = function () {
        var result

        if (xhr.readyState === 3) {
            result = xhr.responseText.substring(received)
            received += result.length

            progress(result)
        } else if (xhr.readyState === 4) {
            finished(xhr.responseText)
        }
    }
    xhr.open('get', url)
    xhr.send(null)
    
    return xhr
}

var client = createStreamingClient('http://demo.com', function (data) {
        alert('Received: ' + data)
    }, function (data) {
        alert('Done!')
    } )
```

## SSE

SSE（Server-Sent Events，服务器发送事件）是围绕只读Comet交互推出的API。SSE API用于创建到服务器的单向连接，服务器通过这个连接可以发送任意数量的数据。服务器响应的MIME类型必须是`text/event-stream`，而且是浏览器中的JavaScript API能解析格式输出。SSE支持短轮询、长轮询和HTTP流，而且能在断开连接时自动确定何时重新连接。

SSE的JavaScript API与其他传递消息的JavaScript API很相似。要预订新的事件流，首先要创建一个新的`EventSource`对象实例，并传进一个入口点：

```js
var source = new EventSource(url)
// 或者
var source = new EventSource(url, { withCredentials: true })
```

上面的`url`可以与当前网页同域，也可以跨域。跨域时，可以指定第二个参数。`withCredentials`是一个布尔值，表示是否一起发送`cookie`。

默认情况下，`EventSource`对象会保持与服务器的活动连接。如果连接断开，还会重新连接。这就意味着SSE适合长轮询和HTTP流。如果想强制立即断开连接并且不再重新连接，可以调用`close()`方法。

```js
source.close()
```

{% note info %}
`EventSource`的实例有一个`readyState`属性
- 值为`0`表示正连接到服务器；
- 值为`1`表示打开了连接；
- 值为`2`表示关闭了连接。

---
另外，还有以下三个事件：
- **open**：在建立连接时触发；
- **message**：在从服务器接收到新事件时触发；
- **error**：在无法建立连接时触发。

服务器发回的数据已字符串形式保存在`event.data`中。
{% endnote %}

所谓的服务器事件会通过一个持久的HTTP响应发送，这个响应的MIME类型为`text/event-stream`。响应数据的格式是纯文本，最简单的情况是每个数据项都带有前缀`data:`，例如：

```md
data: foo

data: bar

data: foo
data bar
```

对以上响应而言，事件流中的第一个message事件返回的`event.data`值为`"foo"`，第二个message事件返回的`event.data`值为`"bar"`，第三个message事件返回的`event.data`值为`"foo\nbar"`（注意中间的换行符）。对于多个连续的以`data:`开头的数据行，将作为多段数据解析，每个值之间以一个换行符分隔。**只有在包含`data:`的数据行后面有空格时，才会触发message事件，因此在服务器上生成事件流时不能忘了多添加这一行。**

通过`id:`前缀可以给特定的事件指定一个关联的ID，这个ID行位于`data:`行前面或后面皆可：

```md
data: foo
id: 1
```

设置了ID后，`EventSource`对象会跟踪上一次触发的事件。如果连接断开，会向服务器发送一个包含名为`Last-Event-ID`的特殊HTTP头部的请求，以便服务器知道下一次该触发哪个事件。在多次连接的事件流中，这种机制可以确保浏览器以正确的顺序收到连接的数据段。

## Web Sockets

Web Sockets的目标是在一个单独的持久连接上提供全双工、双向通信。在JavaScript中创建了Web Sockets之后，会有一个HTTP请求发送到浏览器以发起连接。在取得服务器响应后，建立的连接会使用Web Sockets协议。也就是说，使用标准的HTTP服务器无法实现Web Sockets，只有支持Web Sockets协议的专门服务器才能正常工作。

由于Web Sockets使用了自定义的协议，所以URL模式也略有不同。**未加密得到连接不再是`http://`而是`ws://`；加密的连接也不是`https://`而是`wss://`。**在使用Web Sockets URL时，必须带着这个模式，因为将来还有可能支持其他模式。

要创建Web Sockets，先实例一个`WebSocket`对象并传入要连接的URL：

```js
var socket = new WebSocket('ws://www.example.com/server.php')
```

**注意，必须给`WebSocket`构造函数传入绝对URL。**同源策略对Web Sockets不适用，因此可以通过它打开到任何站点的连接。

实例化了`WebSocket`对象后，浏览器就会马上尝试创建连接。

{% note info %}
与XHR类似，`WebSocket`也有一个表示当前状态的`readyState`属性。不过，这个属性的值与XHR并不相同，而是如下所示：
- **WebSocket.OPENING(0)**：正在建立连接；
- **WebSocket.OPEN(1)**：已经建立连接；
- **WebSocket.CLOSING(2)**：正在关闭连接；
- **WebSocket.CLOSE(3)**：已经关闭连接。

**`WebSocket`没有readystatechange事件。**
{% endnote %}

要关闭Web Sockets连接，可以在任何时候调用`close()`方法。

```js
socket.close()
```

调用了`close()`方法之后，`readyState`属性的值立即变为`2`，而在关闭连接后就会变成`3`。

Web Sockets打开之后，就可以通过连接发送和接收数据。要向服务器发送数据，使用`send()`方法并传入任意字符串，例如：

```js
var socket = new WebSocket('ws://www.example.com/server.php')
socket.send('Hello World!')
```

当服务器向客户端发来消息时，`WebSocket`对象就会触发message事件。这个message事件与其他传递消息的协议类似，也是把返回的数据保存在`event.data`属性中。

```js
socket.onmessage = function () {
    console.log(event.data)
}
```

{% note info %}
**在使用Web Sockets连接发送和接收数据时，需要注意传递的数据只能是纯文本数据。**
{% endnote %}

`WebSocket`对象还有其他三个事件，在连接生命周期的不同阶段触发，分别是`open`、`error`和`close`。`WebSocket`对象不支持DOM2级事件侦听器，因此必须使用DOM0级语法分别注册每个事件处理程序。

```js
var socket = new WebSocket('ws://www.example.com/server.php')

socket.onopen = function () {
    alert('Connection established.')
}

socket.onerror = function () {
    alert('Connection error.')
}

socket.onclose = function () {
    alert('Connection closed.')
}
```

在这三个事件中，只要close事件的`event`对象有额外的信息。这个事件的事件对象有三个额外属性：`wasClean`、`code`和`reason`。其中，`wasClean`是一个布尔值，表示连接是否已经明确地关闭；`code`是服务器返回的数值状态码；而`reason`是一个字符串，包含服务器发回的消息。

## SSE与Web Sockets

面对某个具体的用例，在考虑是使用SSE还是使用Web Sockets时，可以考虑如下几个因素。首先，你是否有自由度建立和维护Web Sockets服务器？因为Web Sockets协议不同于HTTP，所以现有服务器不能用于Web Sockets通信。SSE倒是通过常规HTTP通信，因此现有服务器就可以满足需求。

第二个要考虑的问题是到底需不需要双向通信。如果用例只需读取服务器数据，那么SSE比较容易实现。如果用例必须双向通信，那么Web Sockets显然更好。别忘了，在不能选择Web Sockets的情况下，组合XHR和SSE也是能实现双向通信的。
