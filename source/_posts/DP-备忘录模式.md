---
title: 备忘录模式
abbrlink: c3176455
date: 2019-07-21 14:56:29
categories:
  - Design Pattern
---

# 基础概念

![备忘录模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%A4%87%E5%BF%98%E5%BD%95%E6%A8%A1%E5%BC%8F/%E5%A4%87%E5%BF%98%E5%BD%95%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

![备忘录模式交互](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%A4%87%E5%BF%98%E5%BD%95%E6%A8%A1%E5%BC%8F/%E5%A4%87%E5%BF%98%E5%BD%95%E6%A8%A1%E5%BC%8F%E4%BA%A4%E4%BA%92.png)

{% note info %}

1. 备忘录模式：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。
2. 参与者： - **`Memento`（备忘录）** - 备忘录存储原发器对象在某一时刻的内部状态。原发器根据需要决定备忘录存储原发器的哪些内部状态。 - 防止原发器以外的其它对象访问备忘录内部存储的状态。理想的情况是只允许生成本备忘录的那个原发器访问本备忘录的内部状态。 - **`Originator`（原发器）** - 原发器创建一个备忘录，用以记录当前时刻它的内部状态。 - 使用备忘录恢复内部状态到某一时刻。 - **`Caretaker`（管理者）** - 负责保存备忘录。 - 不能对备忘录的内容进行操作或检查。
   {% endnote %}

# demo

等我遇到实际需求，我再更新 demo 吧

```js
class Originator {
  constructor() {
    this.state = 0;
  }

  setMemento(memento) {
    this.state = memento.state;
  }

  createMemento() {
    return new Memento(this.state);
  }
}

class Memento {
  constructor(state) {
    this.state = state;
  }
}

class Caretaker {
  constructor(originator) {
    this.mementoList = [];
    this.originator = originator;
  }

  addMemento() {
    this.mementoList.push(this.originator.createMemento());
  }
}
```
