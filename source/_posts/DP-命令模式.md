---
title: 命令模式
abbrlink: '78134e07'
date: 2019-07-21 10:37:06
categories:
  - Design Pattern
---

# 基础概念

![命令模式结构](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84.png)

![命令模式交互](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E4%BA%A4%E4%BA%92.png)

{% note info %}

1. 命令模式：将一个命令封装为一个对象，从而使你可用不同的命令对客户进行参数化。
2. 适用性：
   - 抽象出待执行的命令，以参数化某对象。
   - 在不同的时刻指定、排列和执行命令。
   - 支持取消操作。
   - 支持修改日志，这样当系统崩溃时，这些修改可以被重做一遍。
3. 参与者： - **`Command`** - 声明`Execute()`的接口。 - **`ConcreteCommand`** - 将一个接收者对象绑定于一个动作。 - 实现`Execute()`接口，在该接口中会调用接收者相应的动作。 - **`Client`** - 创建一个具体命令对象，并设定它的接收者。 - **`Invoker`** - 保存着命令队列，且由它去触发命令的执行。 - **`Receiver`** - 命令的接收者，命令是由接收者完成执行的。 - 任何类都可能作为一个接收者。
   {% endnote %}

# 注意事项

{% note info %}

1. **一个命令对象应达到何种智能程度**：命令对象的能力可大可小。存在两种极端情况
   - 命令对象仅确定一个接收者和一个执行该命令的动作，其他什么也不做。
   - 命令对象自己实现所有功能，根本不需要额外的接收者对象。
2. **支持取消和重做**：如果`Command`提供方法逆转命令的执行，就可以支持取消和重做功能。为了达到这个目的，`ConcreteCommand`类可能需要存储额外的状态信息：
   - 接收者对象。
   - 接收者上执行命令的参数。
   - 如果执行命令的动作会改变接收者对象中的某些值，那么这些值必须先存储起来。接收者还必须提供一些动作，以使该命令可将接收者恢复到它先前的状态。
3. **避免取消操作过程中的错误累积**：在实现一个可靠的、能确保原先语义的取消/重做机制时，可能会遇到滞后影响问题。由于命令重复的执行、取消执行，和重执行的过程可能会累积错误，以至一个应用的状态最终偏离初始值。这就有必要在`Command`中存入更多的信息以保证这些对象可被精确地复原成它们的初始状态。这里可使用[备忘录模式](https://aadonkeyz.com/posts/c3176455/)来让该`Command`访问这些信息而不暴露其他对象的内部信息。
   {% endnote %}

# demo

呃，懒得想具体的场景了

```js
class Command {
  constructor(receiver, whatToDo) {
    this.receiver = receiver;
    this.whatToDo = whatToDo;
  }

  execute() {
    this.receiver.action.apply(this, this.whatToDo);
  }
}

class Client {
  constructor(name) {
    this.name = name;
  }

  setCommand(receiver, whatToDo, invoker) {
    let command = new Command(receiver, whatToDo);
    invoker.addCommand(command);
  }
}

class Invoker {
  constructor(name) {
    this.name = name;
    this.queue = [];
  }

  addCommand(command) {
    this.queue.push(command);
  }

  callCommand() {
    while (this.queue.length !== 0) {
      this.queue.shift().execute;
    }
  }
}

class Receiver {
  constructor() {
    this.name = name;
  }

  action(whatToDo) {
    // do whatToDo
  }
}
```
