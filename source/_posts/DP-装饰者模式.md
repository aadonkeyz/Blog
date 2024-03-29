---
title: 装饰者模式
categories:
  - Design Pattern
abbrlink: a708a60d
date: 2019-07-14 20:23:28
---

# 基础概念

![装饰者模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E8%A3%85%E9%A5%B0%E8%80%85%E6%A8%A1%E5%BC%8F/%E8%A3%85%E9%A5%B0%E8%80%85%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

{% note info %}

1. 装饰者模式：动态地给一个对象添加一些额外的职责。
2. 适用性：
   - 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
   - 处理那些可撤销的职责。
   - 当不能采用生成子类的方法进行扩充时。一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸式增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于生成子类。
3. 参与者： - **`Component`：`ConcreteComponent`和`Decorator`的父类** - 定义一个对象接口，可以给这些对象动态地添加职责。 - **`ConcreteComponent`** - 定义一个对象，可以给这个对象添加一些职责。 - **`Decorator`** - 维持一个指向`Component`对象的指针。 - 定义一个与`Component`接口一致的接口。 - **`ConcreteDecorator`** - 实现添加装饰的接口。
   {% endnote %}

# 注意事项

{% note info %}

1. **接口一致性**：装饰对象的接口必须与它所装饰的对象的接口是一致的。
2. **省略抽象的`Decorator`类**：当你仅需要添加一个装饰时，没有必要定义抽象`Decorator`类。
3. **保持`Component`类的简单性**：如果有`Component`类，保持这个类的简单性是很重要的，即它应集中于定义接口而不是存储数据。
4. **改变对象外壳与改变对象内核**：我们可以将`Decorator`看作一个对象的外壳，它可以改变这个对象的行为。另外一种方法是改变对象的内核（通过策略模式）。当需要改变的对象比较庞大时，推荐使用策略模式而不是装饰者模式。
   {% endnote %}

# demo

```js
class Plane {
  fire() {
    console.log('发射普通子弹');
  }
}

class EnforePlane {
  constructor(plane) {
    this.plane = plane;
  }

  fire() {
    this.plane.fire();
    console.log('发射超级导弹');
  }
}

var plane = new Plane();
plane = new EnforePlane(plane);
plane.fire(); // 发射普通子弹
// 发射超级导弹
```
