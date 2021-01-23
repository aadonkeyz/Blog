---
title: Cookie
abbrlink: 59cc01ec
date: 2021-01-01 00:33:33
categories:
  - Network
---

# 什么是 Cookie

对于一个网站来说，如果想要做到仅允许特定用户进行访问，或者为不同用户提供不同的内容，前提条件都是它能够识别当前用户。但由于 HTTP 是无状态协议，所以需要额外使用 Cookie 来辅助完成用户的识别。

{% note info %}
Cookie 技术有四个组成部分：
- HTTP 响应头中的一行 cookie 信息
- HTTP 请求头中的一行 cookie 信息
- 保存在用户终端的 cookie 文件
- 服务端用于存储 cookie 数据库
---
Cookie 工作流程：
1. 客户端发送 HTTP 请求到服务器
2. 当服务器收到请求后，在响应头中添加一个 `Set-Cookie` 字段
3. 浏览器收到响应后将 Cookie 保存下来
4. 后续对该服务器的每一次请求，浏览器都会自动为其添加对应的 Cookie 请求头（安全政策允许的话）
{% endnote %}

# Cookie 的属性

## Name

cookie 的 `Name` 由 US-ASCII 字符组成，不允许包含：控制字符（CTLs）、空格、制表符（Tab）、`()@,;:\"/[]?={}`。

虽然 RFC 并没有规定必须使用 URL encoding 对 `Name` 进行编码，但是编码可以保证 `Name` 中不会出现不符合规定的字符。

如果 `Name` 是以 `__Secure-` 为前缀，那么必须同时设置 `Secure` 属性。

如果 `Name` 是以 `__Host-` 为前缀，那么必须同时设置 `Secure` 属性、禁止设置 `Domain` 属性、`Path` 属性的值必须为 `/`。

## Value

cookie 的 `Value` 由 US-ASCII 字符组成，不允许包含：控制字符（CTLs）、空格、双引号、逗号、分号、反斜线。

虽然 RFC 并没有规定必须使用 URL encoding 对 `Value` 进行编码，但是编码可以保证 `Value` 中不会出现不符合规定的字符。

## Domain

`Domain` 属性指定了那些域名可以接受这条 cookie。

如果不指定，那么默认值为当前文档访问地址中的域名，**不包含子域名**。

如果指定了 `Domain`，则子域名也会包含在内。

因此，指定 `Domain` 比省略它的限制要少。通常当子域名需要共享有关用户信息时，会指定 `Domain` 属性。

{% note warning %}
[**stackoverflow: Share cookie between subdomain and domain**](https://stackoverflow.com/a/23086139)
In [RFC 2109](https://tools.ietf.org/html/rfc2109), a domain without a leading dot meant that it could not be used on subdomains, and only a leading dot (`.mydomain.com`) would allow it to be used across multiple subdomains.

However, all modern browsers respect the newer specification [RFC 6265](https://tools.ietf.org/html/rfc6265), and will ignore any leading dot, meaning you can use the cookie on subdomains as well as the top-level domain.
{% endnote %}

{% note warning %}
[**stackoverflow: Domain set cookie for subdomain**](https://stackoverflow.com/a/5258477)
The user agent will accept a cookie with a Domain attribute of `example.com` or of `foo.example.com` from `foo.example.com`, but the user agent will not accept a cookie with a Domain attribute of `bar.example.com` or of `baz.foo.example.com` from `foo.example.com`.
{% endnote %}

## Path

`Path` 属性制定了一个 URL 路径片段，该路径片段必须出现在要请求的资源路径中才可以携带这条 cookie。假设 `Path=/docs`，那么 `/docs/Web` 会携带 cookie，`/test` 则不会携带 cookie。

## Expires

cookie 的最长有效时间，形式为符合 HTTP-date 规范的时间戳。

如果没有设置这个属性，那么表示这是一个 **会话期 cookie**，当客户端关闭后，会话结束，会话期 cookie 会被浏览器移除。

{% note warning %}
很多浏览器支持会话恢复功能，这个功能可以使浏览器保留所有的tab标签，然后在重新打开浏览器的时候将其还原。与此同时，会话期 cookie 也会恢复，就跟从来没有关闭浏览器一样。
{% endnote %}

## Max-Age

在 cookie 失效前需要经过的秒数。秒数为 `0` 或负数会使 cookie 直接过期。

如果同时设置了 `Expires` 和 `Max-Age`，`Max-Age` 的优先级更高。

## Secure

设置该属性后，除 localhost 外，仅使用 `HTTPS` 的请求才能使用 cookie。

HTTP 站点无法为 cookie 设置 `Secure` 属性（在 Chrome 52+ 和 Firefox 52+ 中新引入的限制）。

## HttpOnly

设置该属性后，无法通过 `document.cookie` 访问到这条 cookie。通过该属性可以防止 XSS 攻击获取 cookie 信息。

## SameSite

{% note info %}
该属性用于控制在跨站请求时，是否允许携带这条 cookie。
- `Lax`（默认值）：跨站请求时不能携带这条 cookie，但如果是由外部站点导航的请求（如一个链接），可以携带这条 cookie。
- `Strict`：跨站请求不能携带这条 cookie。
- `None`：只有同时设置了 `Secure` 时，`SameSite = None` 才会生效，此时允许跨站请求携带这条 cookie。
{% endnote %}

# 如何设置 cookie

## Set-Cookie 响应头

```txt
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Path=<path-value>; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly; SameSite=Strict
```

## Document.cookie

通过 [document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie) 来访问/修改 cookie，该方法只能访问到没有设置 `HttpOnly` 的 cookie。

# Cookie 的不足

- cookie 只能以纯文本的形式保存，任何人都有可能对其进行修改
- 不同浏览器对 cookie 的限制不同，通常 cookie 文件总大小被限制在 4kb 以内，单个域名下的 cookie 个数被限制在 20 以内
- 在跨站请求时，cookie 有可能被禁止使用，即不会在请求时在头部附加对应 cookie 信息

# 参考文献

- [MDN: Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN: Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [stackoverflow: Share cookie between subdomain and domain](https://stackoverflow.com/a/23086139)
- [stackoverflow: Domain set cookie for subdomain](https://stackoverflow.com/a/5258477)
- [RFC 2109](https://tools.ietf.org/html/rfc2109)
- [RFC 6265](https://tools.ietf.org/html/rfc6265)