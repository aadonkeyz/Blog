---
title: 第十一章 Promise与异步编程
categories:
    - JavaScript
    - 《深入理解ES6》
abbrlink: 9a3eeeca
date: 2019-04-12 10:26:31
---

# 异步编程的背景

JS引擎建立在单线程事件循环的概念上。单线程意味着同一时刻只能执行一段代码，与Java或C++这种允许同时执行多段不同代码的多线程语言形成了反差。

JS引擎在同一时刻只能执行一段代码，所以引擎无须留意那些“可能”运行的代码。代码会被放置在**作业队列**中，每当一段代码准备被执行，它就会被添加到作业队列。当JS引擎结束当前代码的执行后，事件循环就会执行队列中的下一个作业。**事件循环**是JS引擎的一个内部处理线程，能监视代码的执行并管理作业队列。要记住既然是一个队列，作业就会从队列的第一个开始，依次运行到最后一个。

## 事件模型

当用户点击一个按钮或按下键盘上的一个键时，一个事件 —— 例如`onclick` —— 就被触发了。该事件可能会对此交互进行响应，从而将一个新的作业添加到作业队列的尾部。这就是JS关于异步编程的最基本形式。事件处理程序代码直到事件发生后才会被执行，此时它会拥有合适的上下文。例如：

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

但是回调函数也有它自己的问题，比如回调地狱：

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

在这些情况下，你需要追踪多个回调函数并做清理操作，Promise能大幅度改善这种情况。

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

用这种方式实现`then()`方法的任何对象都被称为一个**thenable**。所有的Promise都是thenable，反之则未必成立。

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

{% note info %}
对于代码的同步和异步执行，你可以简单的理解为同步的优先级大于异步，同步代码被立刻加入作业队列，而异步代码则是当所有同步代码被加入作业队列后才被加入到作业队列尾部的（这个说法不够准确，但是方便理解）。
{% endnote %}

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

`Promise.resolve()`方法接受单个参数并会返回一个处于完成态的Promise。你需要向Promise添加一个或更多的完成处理函数来提取这个参数值。例如：

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

**一个比较有意思的现象是在将非Promise的thenable转换为Promise时，会导致其完成/拒绝处理函数被延后执行。**请看下例：

```js
let p1 = {
    then (resolve, reject) {
        resolve('非Promise的thenable')
    }
}

let p2 = new Promise(function (resolve, reject) {
    resolve('Promise')
})

p3 = Promise.resolve(p1)
p4 = Promise.resolve(p2)

p3.then(value => console.log(value))
p4.then(value => console.log(value))

// 依次输出
// Promise
// 非Promise的thenable
```

## 执行器错误

如果在执行器内部抛出了错误，那么Promise的拒绝处理函数就会被调用。例如：

```js
let promise = new Promise(function (resolve, reject) {
    throw new Error('Explosion!')
})

promise.catch(function (error) {
    console.log(error.message)      // Explosion!
})
```

在此代码中，执行器故意抛出了一个错误。此处在每个执行器之内并没有显式的`try-catch`，因此错误就被捕捉并传递给了拒绝处理函数。这个例子等价于：

```js
let promise = new Promise(function (resolve, reject) {
    try {
        throw new Error('Explosion!')
    } catch (ex) {
        reject(ex)
    }
})

promise.catch(function (error) {
    console.log(error.message)      // Explosion!
})
```

{% note info %}
**`抛出错误 == 拒绝Promise`**
{% endnote %}

# 全局的Promise拒绝处理

Promise最有争议的方面之一就是：当一个Promise被拒绝时若缺少拒绝处理函数，就会静默失败。有人认为这是规范中最大的缺陷，因为这是JS语言所有组成部分中唯一不让错误清晰可见的。

由于Promise的本质，判断一个Promise的拒绝是否已被处理并不直观。例如，研究以下示例：

```js
let rejected = Promise.reject(42)

// 在此刻 rejected 不会被处理

// 一段时间后……
rejected.catch(function (value) {
    // 现在 rejected 已经被处理了
    console.log(value)
})
```

无论Promise是否已被解决，你都可以在任何时候调用`then()`或`catch()`并使它们正确工作，这导致很难准确知道一个Promise何时会被处理。

虽然下个版本的ES可能会处理此问题，不过Node.js与浏览器已经实施了变更来解决开发者的这个痛点。这些变更不是ES6规范的一部分，但却是使用Promise时的宝贵工具。

## Node.js的拒绝处理

在Node.js中，`process`对象上存在两个关联到Promise的拒绝处理的事件：

{% note info %}
- **`unhandledRejection`**：当一个Promise被拒绝、而在事件循环的一个轮次中没有任何拒绝处理函数被调用时，该事件就会被触发；
- **`rejectionHandled`**：当一个Promise被拒绝、并在事件循环的一个轮次之后再有拒绝处理函数被调用时，该事件就会被触发。
{% endnote %}

这两个事件旨在共同帮助识别已被拒绝但未曾被处理的Promise。

`unhandledRejection`事件处理函数接受的参数是拒绝原因（常常是一个错误对象）以及被拒绝的Promise。以下代码展示了`unhandledRejection`的应用：

```js
let rejected

process.on('unhandledRejection', function (reason, promise) {
    console.log(reason.message)             // Explosion!
    console.log(rejected === promise)       // true
})

rejected = Promise.reject(new Error('Explosion!'))
```

`rejectionHandled`事件处理函数则只有一个参数，即已被拒绝的Promise。例如：

```js
let rejected

process.on('rejectionHandled', function (promise) {
    console.log(rejected === promise)       // true
})

rejected = Promise.reject(new Error('Explosion!'))

// 延迟添加拒绝处理函数
setTimeout(function() {
    rejected.catch(function (value) {
        console.log(value.message)          // Explosion!
    })
}, 1000)
```

此处的`rejectionHandled`事件在拒绝处理函数最终被调用时触发。若在`rejected`被创建后直接将拒绝处理函数附加到它上面，那么此事件就不会被触发。因为立即附加的拒绝处理函数在`rejected`被创建的事件循环的同一个轮次内就会被调用，这样`rejectionHandled`就不会起作用。

为了正确追踪潜在的未被处理的拒绝，使用`unhandledRejection`与`rejectionHandled`事件就能保持包含这些Promise的一个列表，之后等待一段时间再检查此列表。例如：

```js
let possiblyUnhandledRejections = new Map()

// 当一个拒绝未被处理，将其添加到 map
process.on('unhandledRejection', function (reason, promise) {
    possiblyUnhandledRejections.set(promise, reason)
})

process.on('rejectionHandled', function (promise) {
    possiblyUnhandledRejections.delete(promise)
})

setInterval(function () {
    possiblyUnhandledRejections.forEach(function (reason, promise) {
        console.log(reason.message ? reason.message : reason)

        // 做点事来处理这些拒绝
        handleRejection(promise, reason)
    })
    possiblyUnhandledRejections.clear()
}, 60000)
```

## 浏览器的拒绝处理

浏览器同样能触发两个事件，来帮助识别未处理的拒绝。这两个事件会被`window`对象触发，并完全等效于Node.js的相关事件：

{% note info %}
- **`unhandledrejection`**：当一个Promise被拒绝、而在事件循环的一个轮次中没有任何拒绝处理函数被调用时，该事件就会被触发；
- **`rejectionhandled`**：当一个Promise被拒绝、并在事件循环的一个轮次之后再有拒绝处理函数被调用时，该事件就会被触发。
{% endnote %}

Node.js的实现会传递分离的参数给事件处理函数，而浏览器的两个事件处理函数则只会接收到包含下列属性的一个对象：

{% note info %}
- **`type`**：事件的名称（`unhandledrejection`或`rejectionhandled`）；
- **`promise`**：被拒绝的Promise对象；
- **`reason`**：Promise中的拒绝值（拒绝原因）。
{% endnote %}

除了事件处理函数接收的参数有所区别外，浏览器的事件处理函数用法可以仿照Node.js。

# 串联Promise

{% note info %}
- **每个`then()`和`catch()`都有自己的返回值**：
    1. 如果`then()`和`catch()`中有`return`语句，则返回`return`后的内容；
    2. 如果没有显示的使用`return`并且代码执行过程中没有抛出错误，那么默认返回`undefined`；
    3. 如果代码抛出错误，那么默认返回错误对象。



- **每次调用`then()`或`catch()`都会创建并返回一个新的Promise**：
    1. 如果`then()`或`catch()`的返回值为一个错误对象，那么这个返回的Promise等价于`Promise.reject(err)`；
    2. 如果`then()`或`catch()`的返回值不是一个错误对象，那么这个返回的Promise等价于`Promise.resolve(returnValue)`。
{% endnote %}

```js
let p1 = new Promise((resolve, reject) => {
    resolve()
})

let p2 = p1.then(() => {
})
p2.then(value => {
    console.log(value)          // undefined
})

let p3 = p1.then(() => {
    return 'return something from then()'
})
p3.then(value => {
    console.log(value)          // return something from then()
})

let p4 = p1.then(() => {
    throw new Error('something wrong in then()')
})
p4.catch(err => {
    console.log(err.message)    // something wrong in then()
})

//===============华丽的分界线==================

let p5 = new Promise((resolve, reject) => {
    reject()
})

let p6 = p5.catch(() => {
})
p6.then(value => {
    console.log(value)          // undefined
})

let p7 = p5.catch(() => {
    return 'return something from catch()'
})
p7.then(value => {
    console.log(value)          // return something from catch()
})

let p8 = p5.catch(() => {
    throw new Error('something wrong in catch()')
})
p8.catch(err => {
    console.log(err.message)    // something wrong in catch()
})
```

既然明白了`then()`或`catch()`的工作原理和实现的效果，那么理解如何将多个Promise串联在一起形成Promise链就很容易了，比如：

```js
let p1 = new Promise((resolve, reject) => {
    resolve('the first Promise')
})

p1.then(value => {
    console.log(value)          // the first Promise
    return 'the second Promise'
}).then(value => {
    console.log(value)          // the second Promise
})
// 等价于
let p2 = p1.then(value => {
    console.log(value)          // the first Promise
    return 'the second Promise'
})
p2.then(value => {
    console.log(value)          // the second Promise
})
```

上面的例子只是展示Promise链的原理，实际上Promise链有更强大的功能。

假设你有两个异步操作需要完成，但是第二个操作必须在第一个操作成功结束后才开始执行，你要如何实现呢？

通过Promise链可以轻松解决这个问题，请研究如下代码：

```js
let p1 = new Promise((resolve, reject) => {
    console.log('p1 start:' + Date.now())

    // 第一个异步操作
    setTimeout(() => {
        console.log('the first async task start')
        resolve('the first async task end')

        console.log('p1 fulfilled:' + Date.now())
    }, 500)
})

p1.then(value => {
    console.log('p1.then() start:' + Date.now())

    return new Promise((resolve, reject) => {
        // 第二个异步操作
        setTimeout(() => {
            console.log(value)
            console.log('the second async task start')
            resolve('the second async task end')

            console.log('p1.then() fulfilled:' + Date.now())
        }, 200)
    })
}).then(value => {
    console.log('p1.then().then() start：' + Date.now())

    console.log(value)
})

// 依次输出
// p1 start:1555150011544
// the first async task start
// p1 fulfilled:1555150012048

// p1.then() start:1555150012048
// the first async task end
// the second async task start
// p1.then() fulfilled:1555150012250

// p1.then().then() start：1555150012250
// the second async task end
```

上面例子中`p1.then()`的返回值不在是单纯的传递数据，而是一个包含异步操作的Promise。虽然第一个`setTimeout()`将延迟时间设置为`500`，第二个`setTimeout()`将延迟时间设置为`200`。但是还是第一个`setTimeout()`内的函数先执行，在其完成之后，第二个`setTimeout()`才会再200ms后将它的回调函数加入作业队列的尾部等待执行。

通过上面例子可以看出，**在Promise链上，每个Promise都会等待上一个Promise决议后，才会执行自己的完成/拒绝处理函数。如果前一个Promise被完成，则后一个Promise的`then()`方法被调用；如果前一个Promise被拒绝，则后一个Promise的`catch()`方法被调用。**

# 响应多个Promise

## Promise.all()

`Promise.all()`方法接收单个可迭代对象（如数组）作为参数，并返回一个Promise。这个可迭代对象的元素都是thenable，只有在它们都完成后，所返回的Promise才会被完成。例如：

```js
let p1 = {
    then (resolve, reject) {
        resolve(42)
    }
}

let p2 = new Promise(function (resolve, reject) {
    resolve(43)
})

let p3 = new Promise(function (resolve, reject) {
    resolve(44)
})

let p4 = Promise.all([p1, p2, p3])

p4.then(function(value) {
    console.log(Array.isArray(value))   // true
    console.log(value[0])               // 42
    console.log(value[1])               // 43
    console.log(value[2])               // 44
})
```

此处前面的每个thenable都用一个数值进行了决议，对`Promise.all()`的调用创建了新的Promise，在`p1`、`p2`和`p3`都被完成后，`p4`才会被完成。**传递给`p4`的完成处理函数的结果是一个包含每个决议值的数组，这些值的存储顺序保持了待决议的thenable的顺序（与完成的先后顺序无关）。**

若传递给`Promise.all()`的任意thenable被拒绝了，那么方法所返回的Promise就会立刻被拒绝，而不必等待其他的thenable结束。**拒绝处理函数总会接收到单个值，而不是一个数组，该值就是被拒绝的thenable所返回的拒绝值。**：

```js
let p1 = {
    then (resolve, reject) {
        resolve(42)
    }
}

let p2 = new Promise(function (resolve, reject) {
    reject(43)
})

let p3 = new Promise(function (resolve, reject) {
    resolve(44)
})

let p4 = Promise.all([p1, p2, p3])

p4.catch(function(value) {
    console.log(Array.isArray(value))   // false
    console.log(value)                  // 43
})
```

## Promise.race()

`Promise.race()`方法接受也接受单个可迭代对象作为参数，并返回一个Promise。这个可迭代对象的元素都是thenable，只要这些thenable中有一个被完成，所返回的Promise就会立刻被完成。例如：

```js
let p1 = Promise.resolve(42)

let p2 = {
    then (resolve, reject) {
        resolve(43)
    }
}

let p3 = new Promise(function (resolve, reject) {
    resolve(44)
})

let p4 = Promise.race([p1, p2, p3])

p4.then(function (value) {
    console.log(value)          // 42
})
```

传递给`Promise.race()`的thenable确实在进行赛跑，看哪一个首先被完成。若胜出的thenable是被完成的，则返回的新Promise也会被完成；若胜出的thenable是被拒绝的，则新Promise也会被拒绝。此处有个使用拒绝的范例：

```js
let p1 = new Promise(function (resolve, reject) {
    setTimeout(() => {
        resolve(42)
    }, 0)
})

let p2 = {
    then (resolve, reject) {
        reject(43)
    }
}

let p3 = new Promise(function (resolve, reject) {
    setTimeout(() => {
        resolve(44)
    }, 0)
})

let p4 = Promise.race([p1, p2, p3])

p4.catch(function (value) {
    console.log(value)          // 43
})
```

# 继承Promise

正像其他内置类型，你可将一个Promise用作派生类的基类。这允许你自定义变异的Promise，在内置Promise的基础上扩展功能。例如，假设你想创建一个可以使用`success()`与`failure()`方法的Promise，对常规的`then()`与`catch()`方法进行扩展，可以像下面这样创建该Promise类型：

```js
class MyPromise extends Promise {
    // 使用默认构造器
    success (resolve, reject) {
        return this.then(resolve, reject)
    }

    failure (reject) {
        return this.catch(reject)
    }
}

let promise = new MyPromise(function (resolve, reject) {
    resolve(42)
})

promise.success(function (value) {
    console.log(value)          // 42
}).failure(function (value) {
    console.log(value)
})
```

在此例中，`MyPromise`从`Promise`上派生出来，并拥有两个附加方法。`success()`方法模拟了`resolve()`，`failure()`方法则模拟了`reject()`。

每个附加方法都使用了`this`来调用它所模拟的方法。派生的Promise函数与内置的Promise几乎一样，除了可以随你需要调用`success()`与`failure()`。

由于静态方法被继承了，`MyPromise.resolve()`方法、`MyPromise.reject()`方法、`MyPromise.race()`方法与`MyPromise.all()`方法在派生的Promise上都可用。后两个方法的行为等同于内置的方法，但前两个方法则有轻微的不同。

`MyPromise.resolve()`与`MyPromise.reject()`方法都会返回`MyPromise`的一个实例，无视传递进来的值的类型，这是由于这两个方法使用了`Symbol.species`属性来决定需要返回的Promise的类型。若传递内置Promise给这两个方法，将会被决议或被拒绝，并且会返回一个新的`MyPromise`，以便绑定完成或拒绝处理函数。例如：

```js
class MyPromise extends Promise {
    // 使用默认构造器
    success (resolve, reject) {
        return this.then(resolve, reject)
    }

    failure (reject) {
        return this.catch(reject)
    }
}

let p1 = new Promise(function (resolve, reject) {
    resolve(42)
})

let p2 = MyPromise.resolve(p1)

p2.success(function(value) {
    console.log(value)                  // 42
})

console.log(p2 instanceof MyPromise)    // true
```

# 异步任务运行

对照[生成器实现的异步运行器](https://aadonkeyz.com/posts/990a6acc/#异步任务运行器)，我们借助Promise对其进行进一步的完善。

```js
let fs = require('fs')

function readFile (filename) {
    return new Promise(function (resolve, reject) {
        fs.readFile(filename, function (err, contents) {
            if (err) {
                reject(err)
            } else {
                resolve(contents)
            }
        })
    })
}

function run (taskDef) {

    // 创建迭代器，让它在别处可用
    let task = taskDef()

    // 启动任务
    let result = task.next()

    // 递归使用函数来保持对 next() 的调用
    function step () {
        // 如果还有更多要做的
        if (!result.done) {

            // 决议一个Promise，让人物处理变简单
            let promise = Promise.resolve(result.value)
            promise.then(function (value) {
                result = task.next(value)
                step()
            }).catch(function (err) {
                result = task.throw(err)
                step()
            })
        }
    }

    // 开始处理过程
    step()
}

run (function * () {
    let contents = yield readFile('config.json')
    doSomethingWith(contents)
    console.log('Done')
})
```
