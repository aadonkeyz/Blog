---
title: Object
abbrlink: a7625ff7
date: 2020-10-24 23:25:59
categories:
  - JavaScript
---

**我们看到的大多数引用类型值都是 Object 类型的实例！对，连数组、函数等都是 Object 类型的实例！**

我所知道的例外有 `Object.create(null)` 返回的空对象。

{% note info %}
- 创建 Object 实例方式有：`new Object()` 和**对象字面量表示法**。
- 对象属性名一定是**字符串**或**符号**，如果不是，会自动转换为字符串。
- 访问对象属性的方式：**点表示法**和**方括号表示法**。
{% endnote %}