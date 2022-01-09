---
title: 适配器模式
categories:
  - Design Pattern
abbrlink: f5c535ea
date: 2019-08-04 15:03:49
---

# 基础概念

![适配器模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F/%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

{% note info %}

1. 适配器模式：将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
2. 适用性：
   - 你想使用一个已经存在的类，而它的接口不符合你的需求。
   - 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类（即那些可能不一定兼容的类）协同工作。
3. 参与者： - **`Target`** - 定义`Client`使用的与特定领域相关的接口。 - **`Client`** - 与符合`Target`接口的对象协同。 - **`Adaptee`** - 定义一个已经存在的接口，这个接口需要适配。 - **`Adapter`** - 对`Adaptee`的接口与`Target`的接口进行适配。
   {% endnote %}
