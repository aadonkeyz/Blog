---
title: 手动实现一个Promise
categories:
    - JavaScript
    - 杂七杂八
---

{% note warning %}
**本文实现的`MyPromise`在任务队列的优先级上，与ES6的`Promise`存在差异。**
{% endnote %}

```js
class MyPromise {
    constructor (fn) {
        this.status = 'pending'
        this.onResolvedCallbacks = []
        this.onRejectedCallbacks = []

        try {
            fn(MyPromise.resolve, MyPromise.reject).call(this)
        } catch (err) {
            MyPromise.reject(err).call(this)
        }
    }

    static resolve (value) {
        // 直接在外部通过MyPromise.resolve()调用时
        // MyPromise.resolve()返回的一定是MyPromise的实例对象
        // 传递任何状态的的MyPromise给MyPromise.resolve()方法，都会将该MyPromise原样返回
        if (typeof this === 'function') {
            if (value instanceof MyPromise) {
                return value
            } else {
                // 需要注意的是，这里一定不要使用箭头函数，否则无法改变this的指向
                return new MyPromise(function (resolve, reject) {
                    resolve(value).call(this)
                })
            }
        }

        // 通过构造函数调用的resolve()
        if (this instanceof MyPromise) {
            setTimeout(() => {
                this.status = 'fulfilled'
                this.value = value
                this.onResolvedCallbacks.forEach(cb => {
                    cb(this.value)
                })
            })
        }
    }

    static reject (err) {
        // 直接在外部通过MyPromise.reject()调用时
        // MyPromise.reject()返回的一定是MyPromise的实例对象
        // 传递任何状态的的MyPromise给MyPromise.reject()方法，都会将该MyPromise作为其catch()方法的参数
        if (typeof this === 'function') {
            // 需要注意的是，这里一定不要使用箭头函数，否则无法改变this的指向
            return new MyPromise(function (resolve, reject) {
                reject(err).call(this)
            })
        }

        // 通过构造函数调用的reject()
        if (this instanceof MyPromise) {
            setTimeout(() => {
                this.status = 'rejected'
                this.value = err
                this.onRejectedCallbacks.forEach(cb => {
                    cb(this.value)
                })
            })
        }
    }
}
```