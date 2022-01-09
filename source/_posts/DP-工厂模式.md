---
title: 工厂模式
abbrlink: 54788f73
date: 2019-07-18 17:17:15
categories:
  - Design Pattern
---

# 基础概念

![工厂模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F/%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

{% note info %}

1. 工厂模式：定义一个用于创建对象的接口，让子类决定实例化哪一个类。
2. 适用性：
   - 当一个类不知道它所必须创建的对象的类的时候。
   - 当一个类希望由它的子类来指定它所创建的对象的时候。
   - 当类创建对象的职责委托给多个帮助子类中的某一个，并且希望将哪一个帮助子类是代理者这一信息局部化的时候。
3. 参与者： - **`Product`** - 定义工厂模式所创建的对象的接口。 - **`ConcreteProduct`** - 实现 Product 的接口 - **`Creator`** - 声明工厂模式，该方法返回一个`Product`类型的对象。`Creator`也可以定义一个工厂模式的缺省实现，它返回一个缺省的 C`oncreteProduct`对象。 - 可以调用工厂模式创建一个`Product`对象。 - **`ConcreteCreator`** - 重定义工厂模式以返回一个`ConcreteProduct`实例。
   {% endnote %}

# 注意事项

{% note info %}

1. **主要有三种不同的情况**：
   - `Creator`类是一个抽象类并且不提供它所声明的工厂模式的实现。
   - `Creator`类是一个具体的类而且为工厂模式提供一个缺省的实现。
   - `Creator`类是一个抽象的类，但为工厂模式提供一个缺省的实现。
2. **参数化工厂模式**：一个工厂模式可以创建多中类型的对象。
   {% endnote %}

# demo

```js
class Product1 {
  constructor() {
    this.type = '产品1';
  }
}

class Product2 {
  constructor() {
    this.type = '产品2';
  }
}

class Creator {
  constructor(typeNum) {
    if (typeNum === 1) {
      return new Product1();
    } else {
      return new Product2();
    }
  }
}

let p1 = new Creator(1),
  p2 = new Creator(2);

console.log(p1); // Product1 { type: '产品1' }
console.log(p2); // Product2 { type: '产品2' }
```
