---
title: HTTP基础
abbrlink: 22f97770
date: 2019-07-15 21:26:17
categories:
    - 计算机网络
    - HTTP
---

# HTTP报文结构

{% note warning %}
- 实体是要进行传输的数据。
- 实体首部字段存在于报文首部字段之中。
- 实体主体和报文主体在起始时是相同的。
- 如果实体主体过大，需要进行分块传输，会导致报文主体中包含多个实体主体。
{% endnote %}

![请求报文格式](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/HTTP%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E9%80%9A%E7%94%A8%E6%A0%BC%E5%BC%8F.png)

![响应报文格式](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/HTTP%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87%E9%80%9A%E7%94%A8%E6%A0%BC%E5%BC%8F.png)

# HTTP方法

## GET

GET是最常用的方法。通常用于向服务器请求某个资源。

![GET](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/GET.png)

## HEAD

HEAD方法与GET方法的行为很类似，但服务器在响应中只返回首部。不会返回实体的主体部分。这就允许客户端在未获取是集资源的情况下，对资源的首部进行检查。服务器开发者必须确保返回的首部与GET请求所返回的首部完全相同。

{% note info %}
使用HEAD，可以：
- 在不获取资源的情况下了解资源的情况。比如，判断其类型。
- 通过查看响应中的状态码，看看某个对象是否存在。
- 通过查看首部，测试资源是否被修改了。
{% endnote %}

![HEAD](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/HEAD.png)

## PUT

与GET从服务器读取文档相反，PUT方法会向服务器写入文档。服务器应使用PUT请求的主体来创建/替换文档。

![PUT](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/PUT.png)

## POST

POST方法用于向服务器传输数据，由服务器根据情况判断如何使用这些数据。

![POST](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/POST.png)

## TRACE

客户端发起一个请求时，这个请求可能要穿过防火墙、代理、网关或其他一些应用程序。每个中间节点都可能会修改原始的HTTP请求。TRACE方法允许客户端在最终将请求发送给服务器时，看看它变成了什么样子。

TRACE请求会在目的服务器端发起一个“环回”诊断。行程最后一站的服务器会弹回一个TRACE响应，并在响应主体中携带它收到的原始请求报文。这样客户端就可以查看在所有中间HTTP应用程序组成的请求/响应链上，原始报文是否以及如何被毁坏或修改的。

尽管TRACE可以很方便地用于诊断，但它确实也有缺点，它假定中间应用程序对各种不同类型请求的处理是相同的。很多HTTP应用程序会根据方法的不同做出不同的事情——比如，代理可能会将POST请求直接发送给服务器，而将GET请求发送给另一个HTTP应用程序。TRACE并不提供区分这些方法的机制。通常，中间应用程序会自行决定对TRACE请求的处理方式。

TRACE请求中不能带有实体的主体部分。TRACE响应的实体主体部分包含了响应服务器收到的请求的精确副本。

![TRACE](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/TRACE.png)

## OPTIONS

OPTIONS方法请求服务器告知其支持的各种功能。可以询问服务器通常支持哪些方法，或者对某些特殊资源支持哪些方法。

![OPTIONS](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/OPTIONS.png)

## DELETE

DELETE方法所做的事情就是请服务器删除请求URL所指定的资源。但是，客户端应用程序无法保证删除操作一定会被执行，因为HTTP规范允许服务器在不通知客户端的情况下撤销请求。

![DELETE](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/DELETE.png)

## GET与POST的区别

{% note info %}
- GET 请求参数放在 URL 中，POST 请求参数放在请求的 body 中。
- GET 请求会被浏览器主动缓存，而 POST 则需要手动设置。
- 浏览器对 GET 请求在 URL 中传送的参数是有长度限制的（通常为 2K），而对 POST 请求的限制为 80K - 100K。**实际上 GET 和 POST 请求本身对大小是无限制的。**
- GET 请求产生一个 TCP 数据包，而 POST 请求产生两个数据包（Firefox 除外）。POST 请求先发送 header ，服务器响应 100 Continue 后，再接着发送 data，最后服务器响应 200 OK。
{% endnote %}

# 状态码

![状态码类别](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/%E7%8A%B6%E6%80%81%E7%A0%81%E7%B1%BB%E5%88%AB.png)

{% note info %}
- **100 Continue**：说明收到了请求的初始部分，请客户端继续。发送了这个状态码之后，服务器在收到请求之后必须进行响应。
- **101 Switching Protocols**：说明服务器正在根据客户端的指定，将协议切换成`Update`首部所列的协议。

---
- **200 OK**：请求没问题，实体的主体部分包含了所请求的资源。
- **204 No Content**：响应报文中包含若干首部和一个状态行，但没有实体的主体部分。主要用于在浏览器不专为显示新文档的情况下，对其进行更新。
- **206 Partial Content**：客户端进行了范围请求，而服务器成功执行了这部分的GET请求。响应报文中包含由`Content-Range`指定范围的实体内容。

--- 
- **301 Moved Permanently**：在请求的URL已被移除时使用。响应的`Location`首部中应包含资源现在所处的URL。
- **302 Found**：与301状态码类似，但是客户端应该使用`Location`首部给出的URL来临时定位资源。将来的请求仍应使用老的URL。
- **303 See Other**：与`302`类似，但是`303`明确表示客户端应当采用GET方法获取资源，这点与`302`有区别。
- **304 Not Modified**：告知客户端它的它所请求的资源不同重新缓存，使用以前的就好。它虽然被划分在`3XX`类别中，但是和重定向没有关系。
- **307 Temporary Redirect**：与`302`有着相同的含义。尽管`302`禁止POST变换为GET，但实际使用时大家并不遵守。`307`会遵照浏览器标准，不会从POST变换为GET。

---
- **400 Bad Request**：请求报文中存在语法错误。
- **401 Unauthorized**：发送的请求需要有通过HTTP认证的认证信息，若之前已进行过一次请求，则表示用户认证失败。
- **403 Forbidden**：对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。
- **404 Not Found**：服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用。

---
- **500 Internal Server Error**：服务器端在执行请求时发生了错误。
- **502 Bad Gateway**：服务器作为网关或代理，从上游服务器收到无效响应。
- **503 Service Unavailable**：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。如果事先得知解除以上状况需要的时间，最好写入`RetryAfter`首部字段再返回给客户端。
{% endnote %}

# 版本比较

## HTTP 0.9

历史上第一个有记载的 HTTP 版本是 0.9，它诞生在 1991 年。这个协议被设计用于从服务器获取 HTML 文档。

```text
telent example.com 80
GET /
```

整个协议的请求只有 1 行，只支持 GET 请求，当从服务器返回文档内容后，关闭 TCP 连接。

## HTTP 1.0

{% note info %}
- 支持请求/响应头
- 支持响应状态码
- 支持 HEAD、POST 方法
{% endnote %}

## HTTP 1.1

{% note info %}
- 支持持久连接
- 支持并行连接
- 支持管道化连接
- 在 HTTP 1.0 的基础上，对 HTTP 缓存进行优化（加入`Etag`实体标签）
- 支持 OPTIONS、PUT、DELETE、TRACE、CONNECT 方法
{% endnote %}

## HTTP 2.0

{% note info %}
- 所有数据以二进制格式传输
- 支持信道复用，一个 TCP 连接里可以发送多个请求，不再需要按着顺序来
- 头信息压缩、减少带宽
- 支持服务器推送
{% endnote %}

# HTTP连接方式

{% note info %}
- **串行连接**：一个 TCP 连接一次只能发送一次 HTTP 请求。
- **并行连接**：通过多条 TCP 连接发起并发的 HTTP 请求。
- **持久连接**：重用 TCP 连接，以消除连接及关闭时延。
- **管道化连接**：通过共享的 TCP 连接发起并发的 HTTP 请求。
{% endnote %}

## 串行连接

串行连接中 HTTP 请求只能一个一个发送，且每次发起 HTTP 请求都要进行 TCP 连接和断开操作，时延较长且耗费性能。

![串行连接](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/%E4%B8%B2%E8%A1%8C%E8%BF%9E%E6%8E%A5.png)

## 并行连接

并行连接中 HTTP 允许客户端打开多条连接，并行地执行多个 HTTP 事务。相比于串行连接，每个 TCP 连接的时延会重叠，所以总的网络时延会较小。但是想象一个场景，如果一百个用户同时发出申请，每个用户打开 100 个连接，服务器就要负责处理 10000 个连接，这会造成服务器性能的严重下降。

{% note warning %}
总结起来，并行连接有如下缺点：
- 每个事务都会打开/关闭一条新的 TCP 连接，会耗费时间和带宽。
- 由于 TCP 拥塞控制的存在，每条新连接的性能都会有所降低。
- 可打开的并行连接数量实际上是有限的。
{% endnote %}

![并行连接](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/%E5%B9%B6%E8%A1%8C%E8%BF%9E%E6%8E%A5.png)

## 持久连接

![持久连接](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/%E6%8C%81%E4%B9%85%E8%BF%9E%E6%8E%A5.png)

## 管道化连接

HTTP/1.1 允许在持久连接上可选地使用**请求管道**。在响应到达之前，可以将多条请求放入队列，当第一条请求通过网络 流向地球另一端的服务器时，第二条和第三条请求也可以开始发送了。

{% note warning %}
对管道化连接有几条限制：
- 如果 HTTP 客户端无法确认连接是持久的，就不应该使用管道。
- 必须使用与请求相同的顺序回送 HTTP 响应。HTTP 报文中没有序列号标签，因此如果收到的响应失序了，就没办法将其与请求匹配起来了。
- HTTP 客户端必须做好连接会在任意时刻关闭的准备，还要准备好重发所有未完成的管道化请求。如果客户端打开了一条持久连接，并立即发出了 10 条请求，服务器可能在只处理了 5 条请求之后关闭连接，剩下的 5 条请求会失败，客户端必须能够应付这些过早关闭连接的情况，重新发出这些请求。
- HTTP 客户端不应该使用管道化的方式发送会产生副作用的请求（比如 POST）。总之，出错的时候，管道化方式会阻碍客户端了解服务器执行的是一系列管道化请求中的哪一些。由于无法安全地重试 POST 这样的非幂等请求，所以出错时，就存在某些方法永远不会被执行的风险。
{% endnote %}

![管道化连接](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E5%9F%BA%E7%A1%80/%E7%AE%A1%E9%81%93%E5%8C%96%E8%BF%9E%E6%8E%A5.png)

# 首部

## 通用首部

{% note info %}
- `Connection`：允许客户端和服务器指定与请求/响应连接有关的选项。
- `Cache-Control`：强缓存使用的首部字段。
- `Date`：创建报文的日期。
- `MIME-Version`：给出了发送端使用的 MIME 版本。
- `Trailer`：如果报文采用了分块传输编码，就可以用这个首部列出位于报文拖挂部分的首部集合。
- `Transfer-Encoding`：告知接收端为了保证报文的可靠传输，对报文采用了什么编码方式。
- `Update`：给出了发送端可能想要“升级”使用的新版本或协议。
- `Via`：显示了报文经过的中间节点（代理、网关）。
- `Warning`：错误通知。
{% endnote %}

## 请求首部

{% note info %}
- `Accept`：告诉服务器能够发送哪些媒体类型。
- `Accept-Charset`：告诉服务器能够发送哪些字符集。
- `Accept-Encoding`：告诉服务器能够发送哪些内容编码。
- `Accept-Language`：告诉服务器能够发送哪些自然语言。
- `Authorization`：Web 认证信息。
- `Except`：期待服务器的特定行为。
- `From`：用户的电子邮箱地址。
- `Host`：请求资源所在服务器。
- `If-Match`：协商缓存使用的首部字段。
- `If-Modified-Since`：协商缓存使用的首部字段。
- `If-Range`：允许对文档的某个范围进行条件请求。
- `If-Unmodified-Since`：协商缓存使用的首部字段。
{% endnote %}

## 响应首部

{% note info %}
- `Accept-Ranges`：是否接受字节范围请求。
- `Age`：推算资源创建经过时间。
- `Etag`：协商缓存使用的首部字段。
- `Location`：令客户端重定向至指定 URI。
- `Proxy-Authenticate`：代理服务器对客户端的认证信息。
- `Retry-After`：对再次发起请求的时机要求。
- `Server`：HTTP 服务器的安装信息。
- `Vary`：代理服务器缓存的管理信息。
- `WWW-Authenticate`：服务器对客户端的认证信息。
{% endnote %}

## 实体首部

{% note info %}
- `Allow`：资源可支持的 HTTP 方法。
- `Content-Encoding`：实体主体使用的编码方式。
- `Content-Language`：实体主体的自然语言。
- `Content-Length`：实体主体的大小
- `Content-Location`：替代对应资源的 URI。
- `Content-MD5`：实体主体的报文摘要。
- `Content-Range`：实体主题的位置范围。
- `Content-Type`：实体主题的媒体类型。
- `Expires`：实体主体的日期时间。
- `Last-Modified`：资源的最后修改日期时间。
{% endnote %}
