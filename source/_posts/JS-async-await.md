---
title: async/await
categories:
  - JavaScript
abbrlink: ca9ee217
date: 2019-07-08 09:06:55
---

# 基础概念

JavaScript 是单线程的，为了处理异步操作，先是使用**回调函数**，接着使用 [Promise](https://aadonkeyz.com/posts/9a3eeeca/)，然后又使用 [generator/yield](https://aadonkeyz.com/posts/9a3eeeca/#异步任务运行)，最后到本文要介绍的 **async/await**。

引用阮一峰大大的一句话 [async 函数就是 generator 函数的语法糖。](http://www.ruanyifeng.com/blog/2015/05/async.html)

首先看一个结合 Promise 和 generator 处理异步的例子：

```js
var fs = require('fs')

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) {
        reject(error)
      }
      resolve(data)
    })
  })
}

function run (taskDef) {
  let task = taskDef()
  let result = task.next()

  function step () {
    if (!result.done) {
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

  step()
}

var generator = function * (){
  var f1 = yield readFile('./verify.js')
  var f2 = yield readFile('./readline.js')
  console.log(f1.toString())
  console.log(f2.toString())
}

run(generator)
```

{% note info %}
通过这个例子，大家也看出来了，**用 generator 函数处理异步的麻烦之处在于，它还需要自己定义执行器。**
{% endnote %}

接下来用 async 函数改写这个例子。

```js
var fs = require('fs')

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) {
        reject(error)
      }
      resolve(data)
    })
  })
}

var asyncInstance = async function () {
  var f1 = await readFile('./verify.js')
  var f2 = await readFile('./readline.js')
  console.log(f1.toString())
  console.log(f2.toString())
}

asyncInstance()
```

{% note info %}
**一比较就会发现，`async` 函数就是将 generator 函数的星号（*）替换成 `async`，将 `yield` 替换成 `await`。而且 `async` 函数不需要自己定义执行器！**
{% endnote %}

# 注意事项

{% note warning %}
1. 区别于生成器的定义，可以使用箭头函数定义 `async`。
2. 与 `yield` 一样，`await` 只能用在 `async` 函数内部，用于其他任意位置都是语法错误，即使在 `async` 函数内部的函数中也不行。
3. `async` 函数返回的是 `Promise` 实例。
4. 如果 `async` 函数内抛出了错误，并且没有被 `try...catch` 包裹，`async` 会直接以该错误作为原因，返回一个状态为 rejected 的 `Promise` 实例。
5. 如果 `await` 后面是一个非 `Promise` 非 thenable 的值，则直接返回。
6. 如果 `await` 后面是一个 `Promise` 实例。
  6.1 当 `Promise` 实例被 fulfill 时，返回 `Promise` 实例的值。
  6.2 当 `Promise` 实例被 reject 时，则将 `Promise` 被 reject 的值作为错误在此处抛出（同 4）。
7. 如果 `await` 后面是一个 thenable 对象，会现将其作为 `Promise` 实例进行处理（同 6）。
{% endnote %}

```js
const asyncFun = async () => {
  return 'resolved'
}

const promise = asyncFun();
promise
  .then(
    (value) => {
      // resolved
      console.log(value)
    }
  )
  .catch(err => {
    console.log(err)
  })
```

```js
const asyncFun = async () => {
  throw 'err'
}

const promise = asyncFun();
promise
  .then(
    (value) => {
      console.log(value)
    }
  )
  .catch(err => {
    // err
    console.log(err)
  })
```

```js
const asyncFun = async () => {
  await Promise.reject('rejected')
}

const promise = asyncFun();
promise
  .then(
    (value) => {
      console.log(value)
    }
  )
  .catch(err => {
    // rejected
    console.log(err)
  })
```
