---
title: 手动实现一个Promise
categories:
    - JavaScript
    - 杂七杂八
abbrlink: 90beba90
date: 2019-07-31 15:54:42
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
            fn.call(this, MyPromise.resolve.bind(this), MyPromise.reject.bind(this))
        } catch (err) {
            MyPromise.reject.call(this, err)
        }
    }

    static resolve (value) {
        // 直接在外部通过MyPromise.resolve()调用时
        // MyPromise.resolve()返回的一定是MyPromise的实例对象
        // 传递任何状态的的MyPromise给MyPromise.resolve()方法，都会将该MyPromise原样返回
        if (typeof this === 'function') {
            if (value instanceof MyPromise) {
                return value
            } else if (value !== null && value.then && typeof value.then === 'function') {
                return new MyPromise(function (resolve, reject) {
                    value.then(MyPromise.resolve.bind(this), MyPromise.reject.bind(this))
                })
            } else {
                // 需要注意的是，这里一定不要使用箭头函数，否则无法改变this的指向
                return new MyPromise(function (resolve, reject) {
                    // 下面的this是指向刚刚创建的MyPromise实例
                    // 因为在构造器中有fn.call(this, MyPromise.resolve.bind(this), MyPromise.reject.bind(this))
                    resolve.call(this, value)
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
                // 下面的this是指向刚刚创建的MyPromise实例
                // 因为在构造器中有MyPromise.reject.call(this, err)
                reject.call(this, err)
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

    then (onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : function (value) { return value }
        onRejected = typeof onRejected === 'function' ? onRejected : function (value) { return value }

        if (this.status === 'fulfilled') {
            return new MyPromise((resolve, reject) => {
                try {
                    resolve(onFulfilled(this.value))
                } catch (err) {
                    reject(err)
                }
            })
        }

        if (this.status === 'rejected') {
            return new MyPromise((resolve, reject) => {
                try {
                    resolve(onRejected(this.value))
                } catch (err) {
                    reject(err)
                }
            })
        }

        if (this.status === 'pending') {
            return new MyPromise((resolve, reject) => {
                this.onResolvedCallbacks.push(() => {
                    try {
                        resolve(onFulfilled(this.value))
                    } catch (err) {
                        reject(err)
                    }
                })

                this.onRejectedCallbacks.push(() => {
                    try {
                        resolve(onRejected(this.value))
                    } catch (err) {
                        reject(err)
                    }
                })
            })
        }
    }

    catch (onRejected) {
        return this.then(null, onRejected)
    }
}
```