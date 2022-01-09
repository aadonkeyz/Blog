---
title: 单例模式
abbrlink: f1601c3e
date: 2019-07-17 20:01:24
categories:
  - Design Pattern
---

# 基础概念

![单例模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

{% note info %}

1. 单例模式：保证一个类仅有一个实例，并提供一个访问该实例的全局访问点。
2. 适用性：
   - 当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
   - 当这个唯一实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。
3. 参与者： - **`Singleton`** - 定义了一个`Instance`操作，允许客户访问它的唯一实例。`Instance`是一个类操作（即它是一个静态成员方法）。 - 可能负责创建它自己的唯一实例。
   {% endnote %}

# demo

{% note warning %}
**基础概念中介绍的是单例模式的原理，而这个 demo 中展示的是使用代理来实现透明单例模式的例子，它们有些许不同。**
{% endnote %}

```js
class CreateSingleton {
  constructor(name) {
    this.name = name;
  }
}

var Singleton = (function () {
  var instance;
  return function (name) {
    if (!instance) {
      instance = new CreateSingleton(name);
    }
    return instance;
  };
})();

var a = new Singleton('a');
console.log(a.name); // a

var b = new Singleton('b');
console.log(b.name); // a

console.log(a === b); // true
```
