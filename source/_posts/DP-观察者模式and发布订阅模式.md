---
title: 观察者模式and发布订阅模式
categories:
  - Design Pattern
abbrlink: 9a246216
date: 2019-07-12 18:03:04
---

{% note warning %}
**按照书中的理解，发布订阅模式属于观察者模式。但是现在常常将二者区分开来了，本文首先介绍观察者模式，然后在过程中介绍发布订阅模式。**
{% endnote %}

# 基础概念

![观察者模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F/%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

![观察者模式交互](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F/%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E4%BA%A4%E4%BA%92.png)

{% note info %}

1. 观察者模式：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
2. 适用性：
   - 当一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这二者封装在独立的对象中以使它们可以各自独立地改变和复用。
   - 当一个对象的改变需要同时改变其它对象，而不知道具体有多少对象待改变。
   - 当一个对象必须通知其它对象，而它又不能假定其它对象是谁。
3. 参与者（未考虑发布订阅模式）： - **`Subject`（目标）** - 目标知道它的观察者。可以有任意多个观察者观察同一个目标。 - 提供注册和删除观察者对象的接口。 - **`Observer`（观察者）** - 为那些在目标发生改变时需获得通知的对象定义一个更新接口。 - **`ConcreteSubject`（具体目标）** - 将有关状态存入各具体观察者中。 - 当它的状态发生改变时，向它的各个具体观察者发出通知。 - **`ConcreteObserver`（具体观察者）** - 存储有关状态，这些状态应与具体目标的状态保持一致。 - 如有需要，可以维护一个指向具体观察者的引用。 - 实现观察者定义的更新接口。
   {% endnote %}

# 注意事项

{% note info %}

1. **创建目标到观察者之间的映射**：
   - 可以在每个目标中创建一个列表，用于保存与之关联的观察者的引用。缺点是当目标很多而观察者较少时，存储代价比较高。
   - 也可以创建一个关联查找机制（例如一个哈希表）用于维护所有目标到观察者的映射。缺点是增加了访问观察者的开销。
2. **观察多个目标**：在某些情况下，一个观察者依赖于多个目标可能是有意义的。在这种情况下，必须扩展观察者的`update()`接口，以使观察者知道是哪一个目标送来的通知。目标对象可以简单地将自己作为`update()`的一个参数，让观察者知道应去检查哪一个目标。
3. **谁调用`notify()`**：目标和它的观察者依赖于通知机制来保持一致。但到底由谁调用`notify()`来触发更新？
   - 可以让目标对象在状态改变后自动调用`notify()`。
   - 也可以让观察者在适当时候主动调用目标的`notify()`。
4. **避免对已删除目标的悬挂引用**：删除一个目标时应注意不要在其观察者中遗留对该目标的悬挂引用。一种避免悬挂引用的方法是，当一个目标被删除时，让它通知它的观察者将对该目标的引用复位。
5. **在调用`notify()`前确保目标的状态自身是一致的**。
6. **避免特定于观察者的更新协议——推/拉模型**：观察者模式的实现经常需要让目标广播关于其改变的其他一些信息。目标将这些信息作为`update()`的参数传递出去。这些信息的量可能很小，也可能很大。
   - **推模型**：目标向观察者发送关于改变的详细信息，而不管它们需要与否。
   - **拉模型**：目标除最小通知外什么也不送出。
7. **显示地指定感兴趣的改变**：你可以扩展目标的注册接口，让各观察者注册为仅对特定事件感兴趣，以提高更新的效率。当一个事件发生时，目标仅通知那些已注册为对该事件感兴趣的观察者。
8. **封装复杂的更新语义**：当目标和观察者间的依赖关系特别复杂时，可能需要一个维护这些关系的对象。我们称这样的对象为**更改管理器（`ChangeManager`）**。它的目的是尽量减少观察者反映其目标状态变化所需的工作量。例如，如果一个操作涉及到对几个相互依赖的目标进行改动，就必须保证仅在所有的目标都已更改完毕后，才一次性地通知它们的观察者，而不是每个目标都通知观察者。`ChangeManager`有三个责任：
   - 它将一个目标映射到它的观察者并提供一个接口来维护这个映射。这就不需要由目标来维护对其观察者的引用，反之亦然。
   - 它定义一个特定的更新策略。
   - 根据一个目标的请求，它更新所有依赖于这个目标的观察者。
9. **结合目标类和观察者类**：对于不支持多重继承的语言，可以将目标类和观察者类结合，实现一个既是目标又是观察者的类。
   {% endnote %}

{% note warning %}
**上述第 8 点，就是观察者模式和发布订阅模式的区别所在了。如果不使用`ChangeManager`则是观察者模式，使用`ChangeManager`的则是发布订阅模式。**
{% endnote %}

# 观察者模式 demo

下面是多个客户关注同一个房产中介，从房产中介获取房源信息的例子。

```js
class Subject {
  constructor(name = '', stateArray = []) {
    this.observers = [];
  }

  attach(ob) {
    this.observers.push(ob);
    console.log(`${ob.name}开始关注${this.name}`);
  }

  detach(ob) {
    this.observers.splice(this.observers.indexOf(ob), 1);
    console.log(`${ob.name}取消关注${this.name}`);
  }

  notify(info) {
    this.observers.forEach((item) => {
      item.update(info);
    });
  }
}

class HouseAgent extends Subject {
  constructor(name) {
    super();
    this.name = name;
    this.houses = {};
  }

  addHouse({ name, price }) {
    this.houses[name] = { name, price };
    this.notify(`${this.name}新增${name}，价格${price}`);
  }

  deleteHouse(name) {
    delete this.houses[name];
    this.notify(`${this.name}下架${name}`);
  }
}

class Observer {
  constructor(name) {
    this.name = name;
  }

  update(info) {
    console.log(`${this.name}知道了${info}`);
  }
}

let agent = new HouseAgent('房屋中介'),
  customer1 = new Observer('小明'),
  customer2 = new Observer('小李');

agent.attach(customer1); // 小明开始关注房屋中介
agent.attach(customer2); // 小李开始关注房屋中介

agent.addHouse({ name: '房屋1', price: '100万' }); // 小明知道了房屋中介新增房屋1，价格100万
// 小李知道了房屋中介新增房屋1，价格100万

agent.addHouse({ name: '房屋2', price: '200万' }); // 小明知道了房屋中介新增房屋2，价格200万
// 小李知道了房屋中介新增房屋2，价格200万

agent.deleteHouse('房屋1'); // 小明知道了房屋中介下架房屋1
// 小李知道了房屋中介下架房屋1
```
