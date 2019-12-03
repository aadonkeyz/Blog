---
title: async/await
categories:
    - JavaScript
    - 杂七杂八
abbrlink: ca9ee217
date: 2019-07-08 09:06:55
---

# async的由来

JavaScript 是单线程的，为了处理异步操作，先是使用**回调函数**，接着使用 [**Promise**](https://aadonkeyz.com/posts/9a3eeeca/)，然后又使用 [**generator/yield**](https://aadonkeyz.com/posts/9a3eeeca/#异步任务运行)，最后到本文要介绍的**async/await**。

引用阮一峰大大的一句话 [**async 函数就是 generator 函数的语法糖。**](http://www.ruanyifeng.com/blog/2015/05/async.html)

首先看用 generator 函数处理异步的一个例子：

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
通过这个例子，大家也看出来了，**用generator 函数处理异步的麻烦之处在于，它还需要自己定义执行器。**
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

var asyncInstance = async function  (){
    var f1 = await readFile('./verify.js')
    var f2 = await readFile('./readline.js')
    console.log(f1.toString())
    console.log(f2.toString())
}

asyncInstance()
```

{% note info %}
**一比较就会发现，async 函数就是将 generator 函数的星号（*）替换成async，将yield替换成await。而且async函数不需要自己定义执行器！**
{% endnote %}

# async三两事

{% note info %}
- 实际上 async 函数与用 generator 函数实现的异步处理器还是存在一些区别的，区别在于它们的`return`在事件循环中的不同表现，[可以点击这里](https://aadonkeyz.com/posts/73ea086f/#加入async函数)。
- `await`和`return`返回的结果，都是`Promise`的实例。
- `await`返回的可能是`rejected`，所以最好把`await`命令放在`try...catch`代码块中。
- 与`yield`一样，`await`只能用在async函数内部，用于其他任意位置都是语法错误，即使在生成器内部的函数中也不行。
- 如果需要多个请求并发执行，可以`await Promise.all()`。
{% endnote %}
