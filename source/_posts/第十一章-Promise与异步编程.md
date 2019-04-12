---
title: 第十一章 Promise与异步编程
categories:
  - JavaScript
  - 《深入理解ES6》
abbrlink: 9a3eeeca
date: 2019-04-12 10:26:31
---

# 异步编程的背景

JS引擎建立在单线程事件循环的概念上。单线程意味着同一时刻只能执行一段代码，与Java或C++这种允许同时执行多端不同代码的多线程语言形成了反差。

JS引擎在同一时刻只能执行一段代码，所以引擎无须留意那些“可能”运行的代码。代码会被放置在**作业队列**中，每当一段代码准备被执行，它就会被添加到作业队列。当JS引擎结束当前代码的执行后，事件循环就会执行队列中的下一个作业。**事件循环**是JS引擎的一个内部处理线程，能监视代码的执行并管理作业队列。要记住既然是一个队列，作业就会从队列的第一个开始，依次运行到最后一个。

## 事件模型

当用户点击一个按钮或按下键盘上的一个键时，一个时间————例如`onclick`————就被触发了。该事件可能会对此交互进行响应，从而将一个新的作业添加到作业队列的尾部。这就是JS关于异步编程的最基本形式。事件处理程序代码直到时间发生后才会被执行，此时它会拥有合适的上下文。例如：

```js
let button = document.getElementById('my-btn')

button.onclick = function (event) {
    console.log('Clicked')
}
```

在此代码中，`console.log('Clicked')`直到`button`被点击后才会被执行。当`button`被点击，赋值给`onclick`的函数就被添加到作业队列的尾部，并在队列前部所有任务结束之后再执行。

事件可以很好地工作于简单的交互，但将多个分离的异步调用串联在一起却会很麻烦，因为必须追踪每个事件的事件对象（例如上例中的`button`）。此外，你还需确保所有的事件处理程序都能在事件第一次触发之前被绑定完毕。例如，若`button`在`onclick`被绑定之前就被点击，那就不会有任何事发生。因此虽然在响应用户交互或类似的低频功能时，事件很有用，但它在面对更复杂的需求时仍然不够灵活。

## 回调模式

当Node.js被创建时，它通过普及回调函数编程模式提升了异步编程模型。回调函数模式类似于事件模型，因为异步代码也会在后面的一个时间点才执行。不同之处在于需要调用的函数（即回调函数）是作为参数传入的，如下所示：

```js
readFile('example.txt', function (err, contents) {
    if (err) {
        throw err
    }

    console.log(contents)
})
console.log('Hi!')
```

此例使用了Node.js惯例，即错误优先的回调函数风格。`readFile()`函数用于读取磁盘中的文件，并在读取完毕后执行回调函数。如果存在错误，回调函数的`err`参数会是一个错误对象；否则`contents`参数就会以字符串形式包含文件内容。

使用回调函数模式，`readFile()`会立即开始执行，并在开始读取磁盘时暂停。这意味着`console.log('Hi!')`会在`readFile()`被调用后立即进行输出，要早于`console.log(contents)`的打印操作。当`readFile()`结束操作后，它会将回调函数以及相关参数作为一个新的作业添加到作业队列的尾部。在之前的作业全部结束后，该作业才会执行。

回调函数有它自己的问题，比如回调地狱：

```js
method1(function (err, result) {

    if (err) {
        throw err
    }

    method2(function (err, result) {

        if (err) {
            throw err
        }

        method3(function (err, result) {

            if (err) {
                throw err
            }

            method4(function(err, result) {

                if (err) {
                    throw err
                }
                
                method5(result)
            })
        })
    })
})
```

像本例一样嵌套多个方法调用会创建错综复杂的代码，会难以理解与调试。当想要实现更复杂的功能时，回调函数也会存在问题。要是你想让两个异步操作并行运行，并且在它们都结束后提醒你，那该怎么做？要是你想同时启动两个异步操作，但只采用首个结束的结果，那又该怎么做？

# Promise基础

Promise是为异步操作的结果所准备的占位符。函数可以返回一个Promise，而不必订阅一个事件或向函数传递一个回调参数，就像这样：

```js
// readFile 承诺会在将来某个时间点完成
let promise = readFile('example.txt')
```

在此代码中，`readFile()`实际上并未立即开始读取文件，这将会在稍后发生。此函数反而会返回一个Promise对象以表示异步读取操作，因此你可以在将来再操作它。你能对结果进行操作的确切时刻，完全取决于Promise的生命周期是如何进行的。

## Promise的生命周期

每个Promise都会经历一个短暂的生命周期，初始为**挂起态（pending state）**，这表示异步操作尚未结束。一个挂起的Promise也被认为是**未决的（unsettled）**。上个例子中的Promise在`readFile()`函数返回它的时候就是处于挂起态。一旦异步操作结束，Promise就会被认为是**已决的（setteld）**，并进入两种可能状态之一：

{% note info %}
- **已完成（fulfilled）**：Promise的异步操作已经成功结束；
- **已拒绝（rejected）**：Promise的异步操作未成功结束，可能是一个错误，或由其他原因导致。
{% endnote %}

内部的`[[PromiseState]]`属性会被设置为`"pending"`、`"fulfilled"`或`"rejected"`，以反映Promise的状态。该属性并未在Promise对象上被暴露出来，因为你无法以编程方式判断Promise到底处于哪种状态。不过你可以使用`then()`方法在Promise的状态改变时执行一些特定操作。

`then()`方法在所有的Promise上都存在，并且接受两个参数。第一个参数是Promise被完成时要调用的函数，与异步操作关联的任何附加数据都会被传入这个完成函数。第二个参数则是Promise被拒绝时要调用的函数，与完成函数相似，拒绝函数会被传入与拒绝相关联的任何附加数据。

用这种方式实现`then()`方法的任何对象都被称为一个thenable。所有的Promise都是thenable，反之则未必成立。

传递给`then()`的两个参数都是可选的，因此你可以监听完成与拒绝的任意组合形式。例如：

```js
let promise = readFile('example.txt')

promise.then(function (contents) {
    // 完成
    console.log(contents)
}, function(err) {
    // 拒绝
    console.error(err.message)
})

promise.then(function (contents) {
    // 完成
    console.log(contents)
})

promise.then(null, function (err) {
    // 拒绝
    console.error(err.message)
})
```

Promise也具有一个`catch()`方法，其行为等同于只传递拒绝处理函数给`then()`。例如，以下的`catch()`与`then()`调用是功能等效的。

```js
promise.catch(function (err) {
    // 拒绝
    console.error(err.message)
})

// 等同于：
promise.then(null, function (err) {
    // 拒绝
    console.error(err.message)
})
```

若你未给Promise附加拒绝处理函数，所有的错误就会静默发生。建议始终附加一个拒绝处理函数，即使该处理程序只是用于打印错误日志。

{% note info %}
每次调用`then()`或`catch()`都会创建一个新的作业，它会在Promise已决议时被执行。但这些作业最终会进入一个完全为Promise保留的作业队列。这个独立队列的确切细节对于理解如何使用Promise是不重要的，你只需要理解作业队列通常来说是如何工作的。
{% endnote %}

## 创建未决的Promise

新的Promise使用`Promise`构造器来创建。此构造器接受单个参数：一个被称为执行器（executor）的函数，包含初始化Promise的代码。该执行器会被传递两个名为`resolve()`与`reject()`的函数作为参数。`resolve()`函数在执行器成功结束时被调用，用于示意该Promise已经准备好被决议，而`reject()`函数则表明执行器的操作已失败。

Promise的执行器是立即执行的，会早于源代码中在其之后的任何代码。
在执行器内部调用的`resolve()`或`reject()`是异步执行的，它们会添加一个作业到作业队列，以便决议这个Promise。
传递给`then()`与`catch()`的函数会被异步地执行，并且他们也被添加到了作业队列（先进队列再执行）。
此处有个例子：

```js
let promise = new Promise(function(resolve, reject) {
    console.log('Promise')
    resolve()
})

promise.then(function() {
    console.log('Resolved')
})

console.log('Hi!')

// 依次输出
// Promise
// Hi!
// Resolved
```

## 创建已决的Promise

基于Promise执行器行为的动态本质，`Promise`构造器就是创建未决的Promise的最好方式。但若你想让一个Promise代表一个已知的值，那么安排一个单纯传值给`resolve()`函数的作业并没有意义。相反，有两种方法可使用指定值来创建已决的Promise。

### 使用Promise.resolve()

`Promise.resolve()`方法接受单个参数并会返回一个出于完成态的Promise。你需要向Promise添加一个或更多的完成处理函数来提取这个参数值。例如：

```js
let promise = Promise.resolve(42)

promise.then(function (value) {
    console.log(value)       // 42
})
```

### 使用Promise.reject()

你也可以使用`Promise.reject()`方法来创建一个已拒绝的Promise。

```js
let promise = Promise.reject(42)

promise.catch(function (value) {
    console.log(value)       // 42
})
```

### Promise作为参数

书中关于这部分的总结有点问题，所以自己证明并整理了如下内容：

**传递任何状态的的Promise给`Promise.resolve()`方法，都会将该Promise原样返回**。

```js
let promise1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('unsettled to resolve')
    }, 500)
})
let promise2 = Promise.resolve(promise1)
console.log(promise1 === promise2)      // true

let promise3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject('unsettled to reject')
    }, 500)
})
let promise4 = Promise.resolve(promise3)
console.log(promise3 === promise4)      // true

let promise5 = Promise.resolve('fulfilled')
let promise6 = Promise.resolve(promise5)
console.log(promise5 === promise6)      // true

let promise7 = Promise.reject('rejected')
let promise8 = Promise.resolve(promise7)
console.log(promise7 === promise8)      // true
```

**传递任何状态的的Promise给`Promise.reject()`方法，都会将该Promise作为其`catch()`方法的参数**。

```js
let promise1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('unsettled to resolve')
    }, 500)
})
let promise2 = Promise.reject(promise1)
promise2.catch(err => {
    console.log(promise1 === err)   // true
    err.then(content => {
        console.log(content)        // unsettled to resolve
    })
})

let promise3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject('unsettled to reject')
    }, 500)
})
let promise4 = Promise.reject(promise3)
promise4.catch(err => {
    console.log(promise3 === err)   // true
    err.catch(err2 => {
        console.log(err2)           // unsettled to reject
    })
})

let promise5 = Promise.resolve('fulfilled')
let promise6 = Promise.reject(promise5)
promise6.catch(err => {
    console.log(promise5 === err)   // true
    err.then(content => {
        console.log(content)        // fulfilled
    })
})

let promise7 = Promise.reject('rejected')
let promise8 = Promise.reject(promise7)
promise8.catch(err => {
    console.log(promise7 === err)   // true
    err.catch(err2 => {
        console.log(err2)           // rejected
    })
})
```

### 非Promise的Thenable

`Promise.resolve()`与`Promise.reject()`都能接受非Promise的thenable作为参数。当传入了非Promise的thenable时，这些方法会创建一个新的Promise，此Promise会在`then()`函数之后被调用。

当一个对象拥有一个能接受`resolve()`与`reject()`参数的`then()`方法，该对象就会被认为是一个非Promise的thenable，就像这样：

```js
let thenable = {
    then: function (resolve, reject) {
        resolve(42)
    }
}
```

如果将非Promise的thenable传递给`Promise.resolve()`或`Promise.reject()`方法，那么JS引擎首先会将非Promise的thenable转换为Promise，然后再将其作为参数传递给`Promise.resolve()`或`Promise.reject()`。

在Promise被引入ES6之前，许多库都使用了thenable，因此将thenable转换为正规Promise的能力就非常重要了，而`Promise.resolve()`或`Promise.reject()`正是完成此任务的最佳方法。

## 执行器错误

# 全局的Promise拒绝处理
## Node.js的拒绝处理
## 浏览器的拒绝处理

# 串联Promise
## 捕获错误
## 在Promise链中返回值
## 在Promise链中返回Promise

# 响应多个Promise
## Promise.all()
## Promise.race()

# 继承Promise

# 异步任务运行
