---
title: XSS和CSRF
abbrlink: 5c76f87e
date: 2019-07-20 16:21:12
categories:
    - 前端
---

# XSS

XSS是跨站脚本攻击（Cross Site Script）的简写。之所以是 X 开头而不是 C 是为了与CSS进行区分。

{% note info %}
**之所以存在XSS攻击，是因为`<script>alert('XSS')</script>`可以内嵌到大部分HTML标签（`<script>`、`<style>`除外）内部，且正常工作（即可以弹出提示框）。**
{% endnote %}

具体的XSS攻击方式就不在这里展示了，不过[**这里挺详细的**](https://github.com/dwqs/blog/issues/68)

至于防范XSS攻击的方式，主要为对`<`、`>`、`script`进行转义或过滤。

# CSRF

CSRF是跨站请求伪造（Cross Site Request Forgery）的简写。

{% note info %}
**之所以存在CSRF攻击，是因为如果浏览器中保存有某一个域名下的Cookie的话，那么每次向该域名下的服务器发送请求都会自动携带上对应的Cookie。而Cookie中通常包含用户的认证信息，所以攻击者可以假冒它是用户来向服务器发起CSRF攻击。**

---
防范CSRF攻击的方式有：
- 使用验证码验证身份。
- 使用HTTP请求的`Referer`首字段检查发送请求的来源地址。`Referer`首字段的另一个用途是“防止图片盗链”。
- 使用token验证Cookie中不包含的信息。
{% endnote %}
