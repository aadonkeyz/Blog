---
title: Promise
categories:
  - JavaScript
abbrlink: 9a3eeeca
date: 2019-04-12 10:26:31
---

# Promise 基础

Promise 是为异步操作的结果所准备的占位符。函数可以返回一个 Promise，而不必订阅一个事件或向函数传递一个回调参数，就像这样：

```js
// readFile 承诺会在将来某个时间点完成
let promise = readFile('example.txt')
```

在此代码中，`readFile()` 实际上并未立即开始读取文件，这将会在稍后发生。此函数反而会返回一个 Promise 对象以表示异步读取操作，因此你可以在将来再操作它。你能对结果进行操作的确切时刻，完全取决于 Promise 的生命周期是如何进行的。

## Promise 的生命周期

每个 Promise 都会经历一个短暂的生命周期，初始为**挂起态（pending state）**，这表示异步操作尚未结束。一个挂起的 Promise 也被认为是**未决的（unsettled）**。上个例子中的 Promise 在 `readFile()` 函数返回它的时候就是处于挂起态。一旦异步操作结束，Promise 就会被认为是**已决的（setteld）**，并进入两种可能状态之一：

{% note info %}
- **已完成（fulfilled）**：Promise 的异步操作已经成功结束。
- **已拒绝（rejected）**：Promise 的异步操作未成功结束，可能是一个错误，或由其他原因导致。
{% endnote %}

内部的 `[[PromiseState]]` 属性会被设置为 `"pending"`、`"fulfilled"` 或 `"rejected"`，以反映 Promise 的状态。该属性并未在 Promise 对象上被暴露出来，因为你无法以编程方式判断 Promise 到底处于哪种状态。不过你可以使用 `then()` 方法在Promise的状态改变时执行一些特定操作。

`then()` 方法在所有的 Promise 上都存在，并且接受两个参数。第一个参数是 Promise 被完成时要调用的函数，与异步操作关联的任何附加数据都会被传入这个完成函数。第二个参数则是 Promise 被拒绝时要调用的函数，与完成函数相似，拒绝函数会被传入与拒绝相关联的任何附加数据。

用这种方式实现 `then()` 方法的任何对象都被称为一个 **thenable**。所有的 Promise 都是 thenable，反之则未必成立。

传递给 `then()` 的两个参数都是可选的，因此你可以监听完成与拒绝的任意组合形式。例如：

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

Promise 也具有一个 `catch()` 方法，其行为等同于只传递拒绝处理函数给 `then()`。例如，以下的 `catch()` 与 `then()` 调用是功能等效的。

```js
promise.catch(function (err) {
  // 拒绝
  console.error(err.message)
})

// 等价于
promise.then(null, function (err) {
  // 拒绝
  console.error(err.message)
})
```

若你未给 Promise 附加拒绝处理函数，所有的错误就会静默发生。建议始终附加一个拒绝处理函数，即使该处理程序只是用于打印错误日志。

## 创建未决的 Promise

新的 Promise 使用 `Promise` 构造器来创建。此构造器接受单个参数：一个被称为执行器（executor）的函数，包含初始化 Promise 的代码。该执行器会被传递两个名为 `resolve()` 与 `reject()` 的函数作为参数。`resolve()` 函数在执行器成功结束时被调用，用于示意该 Promise 已经准备好被决议，而 `reject()` 函数则表明执行器的操作已失败。

Promise 的执行器是立即执行的，会早于源代码中在其之后的任何代码。在执行器内部调用的 `resolve()` 或 `reject()` 是异步执行的。传递给 `then()` 与 `catch()` 的函数也是异步执行的。

```js
let promise = new Promise(function (resolve, reject) {
  console.log('Promise')
  resolve()
})

promise.then(function () {
  console.log('Resolved')
})

console.log('Hi!')

// 依次输出
// Promise
// Hi!
// Resolved
```

## 创建已决的 Promise

基于 Promise 执行器行为的动态本质，`Promise` 构造器就是创建未决的 Promise 的最好方式。但若你想让一个 Promise 代表一个已知的值，那么安排一个单纯传值给 `resolve()` 函数的作业并没有意义。相反，有两种方法可使用指定值来创建已决的 Promise。

### 使用 Promise.resolve

`Promise.resolve()` 方法接受单个参数并会返回一个处于完成态的 Promise。你需要向 Promise 添加一个或更多的完成处理函数来提取这个参数值。例如：

```js
let promise = Promise.resolve(42)

promise.then(function (value) {
  // 42
  console.log(value)       
})
```

### 使用Promise.reject

你也可以使用 `Promise.reject()` 方法来创建一个已拒绝的Promise。

```js
let promise = Promise.reject(42)

promise.catch(function (value) {
  // 42
  console.log(value)       
})
```

### Promise作为参数

{% note warning %}
**传递任何状态的的 Promise 给 `Promise.resolve()` 方法，都会将该 Promise 原样返回**。
{% endnote %}

```js
let promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('unsettled to resolve')
  }, 500)
})
let promise2 = Promise.resolve(promise1)

// true
console.log(promise1 === promise2)      

let promise3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('unsettled to reject')
  }, 500)
})
let promise4 = Promise.resolve(promise3)

// true
console.log(promise3 === promise4)      

let promise5 = Promise.resolve('fulfilled')
let promise6 = Promise.resolve(promise5)

// true
console.log(promise5 === promise6)      

let promise7 = Promise.reject('rejected')
let promise8 = Promise.resolve(promise7)

// true
console.log(promise7 === promise8)      
```

{% note warning %}
**传递任何状态的的 Promise 给 `Promise.reject()` 方法，都会将该 Promise 作为其 `catch()` 方法的参数**。
{% endnote %}

```js
let promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('unsettled to resolve')
  }, 500)
})
let promise2 = Promise.reject(promise1)
promise2.catch(err => {
  // true
  console.log(promise1 === err)   
  err.then(content => {
    // unsettled to resolve
    console.log(content)        
  })
})

let promise3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('unsettled to reject')
  }, 500)
})
let promise4 = Promise.reject(promise3)
promise4.catch(err => {
  // true
  console.log(promise3 === err)   
  err.catch(err2 => {
    // unsettled to reject
    console.log(err2)           
  })
})

let promise5 = Promise.resolve('fulfilled')
let promise6 = Promise.reject(promise5)
promise6.catch(err => {
  // true
  console.log(promise5 === err)   
  err.then(content => {
    // fulfilled
    console.log(content)        
  })
})

let promise7 = Promise.reject('rejected')
let promise8 = Promise.reject(promise7)
promise8.catch(err => {
  // true
  console.log(promise7 === err)   
  err.catch(err2 => {
    // rejected
    console.log(err2)           
  })
})
```

### 非 Promise 的 thenable

当一个对象拥有一个能接受 `resolve()` 与 `reject()` 参数的 `then()` 方法，该对象就会被认为是一个非 Promise 的 thenable，就像这样：

```js
let thenable = {
  then: function (resolve, reject) {
    resolve(42)
  }
}
```

如果将非 Promise 的 thenable 传递给 `Promise.resolve()` 或 `Promise.reject()` 方法，那么 JS 引擎首先会将非 Promise 的 thenable 转换为 Promise，然后再将其作为参数传递给 `Promise.resolve()` 或 `Promise.reject()`。

在 Promise 被引入 ES6 之前，许多库都使用了 thenable，因此将 thenable 转换为正规 Promise 的能力就非常重要了，而 `Promise.resolve()` 或 `Promise.reject()` 正是完成此任务的最佳方法。

{% note warning %}
**一个比较有意思的现象是在将非 Promise 的 thenable 转换为 Promise 时，会导致其完成/拒绝处理函数被延后执行。**
{% endnote %}

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

如果在执行器内部抛出了错误，那么 Promise 的拒绝处理函数就会被调用。例如：

```js
let promise = new Promise(function (resolve, reject) {
  throw new Error('Explosion!')
})

promise.catch(function (error) {
  // Explosion!
  console.log(error.message)      
})
```

在此代码中，执行器故意抛出了一个错误。此处在每个执行器之内并没有显式的 `try-catch`，因此错误就被捕捉并传递给了拒绝处理函数。这个例子等价于：

```js
let promise = new Promise(function (resolve, reject) {
  try {
    throw new Error('Explosion!')
  } catch (ex) {
    reject(ex)
  }
})

promise.catch(function (error) {
  // Explosion!
  console.log(error.message)      
})
```

{% note info %}
**`抛出错误 == 拒绝 Promise`**
{% endnote %}

## finially

{% note info %}
不同于 `then()` 和 `catch()`，`finally()` 是无论如何都会被调用的。它有如下特点：
- `finally()` 的回调函数不接受任何参数。
- 返回调用它的那个 Promise 实例对象，请看下面两个示例。
  - `Promise.resolve(2).finally(() => {})` 将返回一个状态为 fulfilled、值为 2 的 Promise。
  - `Promise.reject(3).then(() => {}, () => {})` 将返回一个状态为 rejected、值为 3 的 Promise。
{% endnote %}

# 全局的 Promise 拒绝处理

Promise 最有争议的方面之一就是：当一个 Promise 被拒绝时若缺少拒绝处理函数，就会静默失败。有人认为这是规范中最大的缺陷，因为这是 JS 语言所有组成部分中唯一不让错误清晰可见的。

由于 Promise 的本质，判断一个 Promise 的拒绝是否已被处理并不直观。例如，研究以下示例：

```js
let rejected = Promise.reject(42)

// 在此刻 rejected 不会被处理

// 一段时间后……
rejected.catch(function (value) {
  // 现在 rejected 已经被处理了
  console.log(value)
})
```

无论 Promise 是否已被解决，你都可以在任何时候调用 `then()` 或 `catch()` 并使它们正确工作，这导致很难准确知道一个 Promise 何时会被处理。

虽然下个版本的 ES 可能会处理此问题，不过 Node.js 与浏览器已经实施了变更来解决开发者的这个痛点。这些变更不是 ES6 规范的一部分，但却是使用 Promise 时的宝贵工具。

## Node.js 的拒绝处理

在 Node.js 中，`process` 对象上存在两个关联到 Promise 的拒绝处理的事件：

{% note info %}
- `unhandledRejection`：当一个 Promise 被拒绝、而在事件循环本轮次中没有任何拒绝处理函数被调用时，该事件就会被触发。
- `rejectionHandled`：当一个 Promise 被拒绝、并从事件循环的下一个轮次开始才有拒绝处理函数被调用时，该事件就会被触发。
{% endnote %}

这两个事件旨在共同帮助识别已被拒绝但未曾被处理的 Promise。

`unhandledRejection` 事件处理函数接受的参数是拒绝原因（常常是一个错误对象）以及被拒绝的 Promise。以下代码展示了 `unhandledRejection`的应用：

```js
let rejected

process.on('unhandledRejection', function (reason, promise) {
  // Explosion!
  console.log(reason.message)             

  // true
  console.log(rejected === promise)       
})

rejected = Promise.reject(new Error('Explosion!'))
```

`rejectionHandled` 事件处理函数则只有一个参数，即已被拒绝的 Promise。例如：

```js
let rejected

process.on('rejectionHandled', function (promise) {
  // true
  console.log(rejected === promise)       
})

rejected = Promise.reject(new Error('Explosion!'))

// 延迟添加拒绝处理函数
setTimeout(function() {
  rejected.catch(function (value) {
    // Explosion!
    console.log(value.message)          
  })
}, 1000)
```

此处的 `rejectionHandled` 事件在拒绝处理函数最终被调用时触发。若在 `rejected` 被创建后直接将拒绝处理函数附加到它上面，那么此事件就不会被触发。因为立即附加的拒绝处理函数在 `rejected` 被创建的事件循环的同一个轮次内就会被调用，这样 `rejectionHandled` 就不会起作用。

为了正确追踪潜在的未被处理的拒绝，使用 `unhandledRejection` 与 `rejectionHandled` 事件就能保持包含这些 Promise 的一个列表，之后等待一段时间再检查此列表。例如：

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

浏览器同样能触发两个事件，来帮助识别未处理的拒绝。这两个事件会被 `window` 对象触发，并完全等效于 Node.js 的相关事件：

{% note info %}
- `unhandledrejection`：当一个 Promise 被拒绝、而在事件循环的本轮次中没有任何拒绝处理函数被调用时，该事件就会被触发。
- `rejectionhandled`：当一个 Promise 被拒绝、并从事件循环的下一个轮次开始才有拒绝处理函数被调用时，该事件就会被触发。
{% endnote %}

Node.js 的实现会传递分离的参数给事件处理函数，而浏览器的两个事件处理函数则只会接收到包含下列属性的一个对象：

{% note info %}
- `type`：事件的名称（`unhandledrejection` 或 `rejectionhandled`）。
- `promise`：被拒绝的 Promise 对象。
- `reason`：Promise 中的拒绝值（拒绝原因）。
{% endnote %}

除了事件处理函数接收的参数有所区别外，浏览器的事件处理函数用法可以仿照 Node.js。

# 串联Promise

{% note info %}
- **每个 `then()` 和 `catch()` 都有自己的返回值**：
  1. 如果 `then()` 和 `catch()` 中有 `return` 语句，则返回 `return` 后的内容。
  2. 如果没有显示的使用 `return` 并且代码执行过程中没有抛出错误，那么默认返回 `undefined`。
  3. 如果代码抛出错误，那么默认返回错误对象。



- **每次调用 `then()` 或 `catch()` 都会创建并返回一个新的 Promise**：
  1. 如果 `then()` 或 `catch()` 的返回值为一个错误对象，那么这个返回的 Promise 等价于 `Promise.reject(err)`。
  2. 如果 `then()` 或 `catch()` 的返回值不是一个错误对象，那么这个返回的 Promise 等价于 `Promise.resolve(returnValue)`。
{% endnote %}

```js
let p1 = new Promise((resolve, reject) => {
  resolve()
})

let p2 = p1.then(() => {
})
p2.then(value => {
  // undefined
  console.log(value)          
})

let p3 = p1.then(() => {
  return 'return something from then()'
})
p3.then(value => {
  // return something from then()
  console.log(value)          
})

let p4 = p1.then(() => {
  throw new Error('something wrong in then()')
})
p4.catch(err => {
  // something wrong in then()
  console.log(err.message)    
})

//===============华丽的分界线==================

let p5 = new Promise((resolve, reject) => {
  reject()
})

let p6 = p5.catch(() => {
})
p6.then(value => {
  // undefined
  console.log(value)          
})

let p7 = p5.catch(() => {
  return 'return something from catch()'
})
p7.then(value => {
  // return something from catch()
  console.log(value)          
})

let p8 = p5.catch(() => {
  throw new Error('something wrong in catch()')
})
p8.catch(err => {
  // something wrong in catch()
  console.log(err.message)    
})
```

既然明白了 `then()` 或 `catch()` 的工作原理和实现的效果，那么理解如何将多个 Promise 串联在一起形成 Promise 链就很容易了，比如：

```js
let p1 = new Promise((resolve, reject) => {
  resolve('the first Promise')
})

p1.then(value => {
  // the first Promise
  console.log(value)          
  return 'the second Promise'
}).then(value => {
  // the second Promise
  console.log(value)          
})
// 等价于
let p2 = p1.then(value => {
  // the first Promise
  console.log(value)          
  return 'the second Promise'
})
p2.then(value => {
  // the second Promise
  console.log(value)          
})
```

上面的例子只是展示 Promise 链的原理，实际上 Promise 链有更强大的功能。

假设你有两个异步操作需要完成，但是第二个操作必须在第一个操作成功结束后才开始执行，你要如何实现呢？

通过 Promise 链可以轻松解决这个问题，请研究如下代码：

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

上面例子中 `p1.then()` 的返回值不在是单纯的传递数据，而是一个包含异步操作的 Promise。虽然第一个 `setTimeout()` 将延迟时间设置为 `500`，第二个 `setTimeout()` 将延迟时间设置为 `200`。但是还是第一个 `setTimeout()` 内的函数先执行，在其完成之后，第二个 `setTimeout()` 才会再 200ms 后将它的回调函数加入作业队列的尾部等待执行。

通过上面例子可以看出，**在 Promise 链上，每个 Promise 都会等待上一个 Promise 决议后，才会执行自己的完成/拒绝处理函数。如果前一个 Promise 被完成，则后一个 Promise 的 `then()` 方法被调用。如果前一个 Promise 被拒绝，则后一个 Promise 的 `catch()` 方法被调用。**

# 响应多个 Promise 

## Promise.all

`Promise.all()` 方法接收单个可迭代对象（如数组）作为参数，并返回一个 Promise。这个可迭代对象的元素都是 thenable，只有在它们都完成后，所返回的 Promise 才会被完成。例如：

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
  // true
  console.log(Array.isArray(value))   

  // 42
  console.log(value[0])           
  
  // 43
  console.log(value[1])               

  // 44
  console.log(value[2])               
})
```

此处前面的每个 thenable 都用一个数值进行了决议，对 `Promise.all()` 的调用创建了新的 Promise，在 `p1`、`p2` 和 `p3` 都被完成后，`p4` 才会被完成。**传递给 `p4` 的完成处理函数的结果是一个包含每个决议值的数组，这些值的存储顺序保持了待决议的 thenable 的顺序（与完成的先后顺序无关）。**

若传递给 `Promise.all()` 的任意 thenable 被拒绝了，那么方法所返回的 Promise 就会立刻被拒绝，而不必等待其他的 thenable 结束。**拒绝处理函数总会接收到单个值，而不是一个数组，该值就是被拒绝的 thenable 所返回的拒绝值。**：

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
  // false
  console.log(Array.isArray(value))   

  // 43
  console.log(value)                  
})
```

## Promise.race

`Promise.race()` 方法接受也接受单个可迭代对象作为参数，并返回一个 Promise。这个可迭代对象的元素都是 thenable，只要这些 thenable 中有一个被完成，所返回的 Promise 就会立刻被完成。例如：

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
  // 42
  console.log(value)          
})
```

传递给 `Promise.race()` 的 thenable 确实在进行赛跑，看哪一个首先被完成。若胜出的 thenable 是被完成的，则返回的新 Promise 也会被完成。若胜出的 thenable 是被拒绝的，则新 Promise 也会被拒绝。此处有个使用拒绝的范例：

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
  // 43
  console.log(value)          
})
```

# 继承 Promise

正像其他内置类型，你可将一个 Promise 用作派生类的基类。这允许你自定义变异的 Promise，在内置 Promise 的基础上扩展功能。例如，假设你想创建一个可以使用 `success()` 与 `failure()` 方法的 Promise，对常规的 `then()` 与 `catch()` 方法进行扩展，可以像下面这样创建该 Promise 类型：

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
  // 42
  console.log(value)          
}).failure(function (value) {
  console.log(value)
})
```

在此例中，`MyPromise` 从 `Promise` 上派生出来，并拥有两个附加方法。`success()` 方法模拟了 `resolve()`，`failure()` 方法则模拟了 `reject()`。

每个附加方法都使用了 `this` 来调用它所模拟的方法。派生的 Promise 函数与内置的 Promise 几乎一样，除了可以随你需要调用 `success()` 与 `failure()`。

由于静态方法被继承了，`MyPromise.resolve()` 方法、`MyPromise.reject()` 方法、`MyPromise.race()` 方法与 `MyPromise.all()` 方法在派生的 Promise 上都可用。后两个方法的行为等同于内置的方法，但前两个方法则有轻微的不同。

`MyPromise.resolve()` 与 `MyPromise.reject()` 方法都会返回 `MyPromise` 的一个实例，无视传递进来的值的类型，这是由于这两个方法使用了 `Symbol.species` 属性来决定需要返回的 Promise 的类型。若传递内置 Promise 给这两个方法，将会被决议或被拒绝，并且会返回一个新的 `MyPromise`，以便绑定完成或拒绝处理函数。例如：

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
  // 42
  console.log(value)                  
})

// true
console.log(p2 instanceof MyPromise)    
```

# 异步任务运行

对照[生成器实现的异步运行器](https://aadonkeyz.com/posts/990a6acc/#异步任务运行器)，我们借助 Promise 对其进行进一步的完善。

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

      // 决议一个Promise，让任务处理变简单
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
