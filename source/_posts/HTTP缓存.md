---
title: HTTP缓存
abbrlink: 8837602f
date: 2019-07-11 09:39:10
categories:
    - 计算机网络
    - HTTP
---

{% note warning %}
缓存的存放位置是浏览器，但是要区分开HTTP缓存和[客户端存储](https://aadonkeyz.com/posts/4fd0352/)（如cookie、localStorage等）的概念。
{% endnote %}

# 缓存的优点

{%note info %}
- 减少冗余数据的传输。
- 缓解了带宽瓶颈的问题。
- 降低了对原始服务器的要求。
- 降低了距离时延。
{% endnote %}

# 基本概念

{%note info %}
- **缓存命中**：用已有的副本为某些到达缓存的请求提供服务。
- **缓存未命中**：缓存中无可用副本，需要将请求转发给原始服务器。
- **再验证**：原始服务器的内容可能会发生变化，缓存要不时对其进行检测，看看缓存中保存的副本是否仍是服务器上最新的副本，这种“新鲜度检测”被称为HTTP再验证。
- **再验证命中或缓慢命中**：缓存对副本进行再验证时，会向原始服务器发送一个小的再验证请求。如果副本仍是新鲜的，则将副本提供给客户端。这种方式比单纯的缓存命中要慢，比缓存未命中要快。
- **缓存命中率**：由缓存提供服务的请求所占的比例。，有时也被称为**文档命中率**。
- **字节命中率**：缓存提供的字节在传输的所有字节中所占的比例。
{% endnote %}

# 相关的首部字段

## Cache-Control

HTTP/1.1定义的`Cache-Control`首部字段用来区分对缓存机制的支持情况，请求头和响应头都支持这个属性。通过它提供的不同的值来定义缓存策略。指令不区分大小写，并且具有可选参数，可以用令牌或者带引号的字符串语法。多个指令以逗号分隔。

{% note info %}
请求中的`Cache-Control`：
- `max-age`：设置缓存副本存储的最大周期，超过这个时间的副本被认为过期（单位为秒，且是从发起请求的那一时刻开始计时的）。
- `max-stale`：表明客户端愿意接收一个已经过期的资源。可以设置一个可选的秒数，表示响应不能已经过时超过该给定时间。
- `max-fresh`：表明客户端希望获取一个能在指定的秒数内保持其最新状态的响应。
- `no-cache`：在发布缓存副本之前，强制要求缓存把请求提交给原始服务器进行验证。
- `no-store`：缓存不应存储有关客户端请求或服务器响应的任何内容。
- `no-transform`：不得对资源进行转换或改变。
- `only-if-cached`：如果在缓存中有可用资源，不需要进行再验证，直接使用。如果没有可用资源的话在去原始服务器获取，然后保存对应的副本。

---
响应中的`Cache-Control`：
- `must-revalidate`：一旦资源过期，在再验证之前，缓存不能使用该资源响应后续请求。
- `no-cache`：在发布缓存副本之前，强制要求缓存把请求提交给原始服务器进行验证。
- `no-store`：缓存不应存储有关客户端请求或服务器响应的任何内容。
- `no-transform`：不得对资源进行转换或改变。
- `public`：表明响应可以被任何应用（包括：发送请求的客户端、代理服务器，等等）缓存，即使是通常不可缓存的内容。
- `private`：表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）。私有缓存可以缓存响应内容。
- `proxy-revalidate`：与`must-revalidate`作用相同，但它仅适用于共享缓存，并被私有缓存忽略。
- `max-age`：设置缓存副本存储的最大周期，超过这个时间副本被认为过期（单位为秒，且是从发起请求的那一时刻开始计时的）。
- `s-maxage`：覆盖`Cache-Control`首部字段的`max-age`或者`Expires`首部字段，但是仅适用于共享缓存，私有缓存会忽略它。
{% endnote %}

## Expires

{% note info %}
- `Expires`属于响应首部字段。
- 它的值为具体的日期，如`Expires: Fri, 05 Jul 2002, 05:00:00 GMT`。
- 超过它设定的日期后，对应的副本就会被标记为过期。
- 如果在**响应**的`Cache-Control`字段设置了`max-age`或者`s-max-age`指令，那么`Expires`字段会被忽略。
- `Expires`字段依赖于计算机时钟的正确设置。
{% endnote %}

## If-Modified-Since

{% note info %}
- `If-Modified-Since`首部字段只能用于GET或HEAD的请求首部。
- `If-Modified-Since`的值为一个绝对日期，用于作为判断的给定日期。
- 如果所请求资源最后的修改日期在给定日期之前，那么服务器会向客户端返回一个不包含主体的`304 Not Modified`。
- 如果所请求资源最后的修改日期在给定日期之后，那么服务器根据实际情况对客户端进行响应。
- 如果`If-Modified-Since`和`If-Node-Match`同时出现，则`If-Modified-Since`会被忽略。
{% endnote %}

## Last-Modified

{% note info %}
- `Last-Modified`属于响应首部字段，它包含的是对应资源的最后修改日期。
- 它的日期精确到秒。
{% endnote %}

## Etag

{% note info %}
- `Etag`属于响应首部字段，它的值代表对应资源的独一无二的**实体标签**。
- 由服务器自己确定`Etag`的值。
{% endnote %}

## If-None-Match

{% note info %}
- `If-None-Match`属于请求首部字段。
- `If-None-Match`可以包含多个`Etag`。如，`If-None-Match: "v2.4", "v2.5", "v2.6"`。
- 如果请求的方法是GET或HEAD，且所请求资源与`If-None-Match`包含的`Etag`匹配，那么服务器会向客户端返回一个不包含主体的`304 Not Modified`。
- 如果请求的方法是GET或HEAD，且所请求资源与`If-None-Match`包含的`Etag`不匹配，那么服务器根据实际情况对客户端进行响应。
- 如果使用其他方法发送请求，那么只有当所请求资源与`If-None-Match`包含的`Etag`不匹配时，服务器才会进行对应的响应。
- 如果`If-Modified-Since`和`If-Node-Match`同时出现，则`If-Node-Match`优先级高。
{% endnote %}

# 缓存工作流程

人们管使用`Cache-Control`或者`Expires`字段的方式叫作**强缓存**，而使用`If-Modified-Since`或`If-None-Match`的方式叫作**协商缓存**。

{% note info %}
1. 浏览器首先查看是否有缓存。
2. 如果有缓存，接着检查是否命中强缓存（即缓存是否过期）。
3. 如果缓存过期，则通过协商缓存来请求服务器，根据服务器返回的状态码进行不同的操作。
{% endnote %}

![缓存工作流程](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP/HTTP%E7%BC%93%E5%AD%98/%E7%BC%93%E5%AD%98%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png)

{% note info %}
1. **`Cache-Control`和`Expires`的比较。**
    - `Cache-Control`的优先级高于`Expires`。
    - 由于`Cache-Control`使用的是相对时间而不是绝对时间，并且`Expires`使用的绝对时间依赖于计算机时钟的正确设置，所以更倾向于使用`Cache-Control`。
2. **相比于`Last-Modified`，`Etag`的优势。**
    - 有的文档可能会被周期性地重写，但实际包含的数据常常是一样的。尽管内容没有变化，但是修改日期会发生变化。
    - 有些文档可能被修改了，但所做修改并不重要，不需要重新缓存。
    - 有些服务器无法精确地判定最后修改日期。
    - 有些服务器提供的文档会以亚秒间隙发生变化，对这些服务器来说，以一秒为粒度的修改日期可能就不够用了。
{% endnote %}

{% note warning %}
**不同域名下，如果加载同一个 URL 的资源，可以共用一份缓存。**
{% endnote %}

# 刷新与缓存

{% note info %}
- 通过 F5 刷新（MAC 下是 command + R），浏览器发送的请求会设置`Cache-Control: max-age=0`和`If-Modified-Since`，也就是说会将本地缓存过期，然后使用协商缓存。
- 通过 Ctrl + F5 强制刷新（MAC 下是 command + shift + R），浏览器发送的请求会设置`Cache-Control: max-age=0`，也就是说会将本地缓存过期，并且不使用协商缓存。

你可以使用 Firefox 浏览器（Chrome 看不到所有请求头）试验一下，分别采用以上两种方式刷新本页面，观察请求和响应的不同。
{% endnote %}
