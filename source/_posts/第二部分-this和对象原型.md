---
title: 第二部分 this和对象原型
abbrlink: d213b25d
date: 2019-03-21 20:08:09
  - JavaScript
  - 《你不知道的JavaScript（上卷）》
categories:
---

# 关于this

当一个函数被**调用**时，会创建一个活动记录（执行上下文、执行环境、context）。这个活动记录会包含函数在哪里被调用、函数的调用方法、传入的参数等信息。`this`就是活动记录中的一个属性，会在函数执行的过程中用到。

`this`是在运行时绑定的，它的绑定取决于函数调用时的各种方式。**`this`的绑定和函数的声明位置、函数的调用位置没有任何关系，只取决于函数的调用方式**。

关于`this`是什么有很多说法：函数本身、函数作用域、函数的执行上下文等，这里有的说法是错误的，有的说法不够准确。我觉得就不要纠结`this`到底是什么东西了，`this`就是`this`，你只要知道它的绑定规则和用法，就ok了！

# this全面解析

书里介绍了函数的调用位置，并说理解调用位置对理解`this`的绑定很重要。但其实`this`的绑定与**调用位置**没有任何关系，只与**调用方式**有关。

## 绑定规则

因为`this`的绑定取决于函数的调用方式，所以下面我们从函数的调用方式分析`this`的绑定规则。

### 默认绑定

首先要介绍的是最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则。

默认绑定是指当函数被调用时，是直接使用不带任何修饰的函数引用进行调用的，这种情况下，如果运行在非严格模式中，`this`会绑定到全局对象上。如果运行在严格模式下，`this`会绑定到`undefined`上。

```js
function foo () {
    console.log(this.a)
}
var a = 2
foo()   // 2
```

### 隐式绑定

在调用函数的时候，如果以对象方法的形式进行调用，这个时候`this`就会绑定到这个方法所属的对象上。

```js
function foo () {
    console.log(this.a)
}
var obj = {
    a: 2,
    foo: foo
}
obj.foo()   // 2

// 复杂点的
var obj2 = {
    a: 42,
    obj: obj
}
// foo方法是属于obj的
obj2.obj.foo()  // 2
```

一定要区分开默认绑定和隐式绑定，下面的例子里包含了让人容易忽略的情况。再次强调**`this`的绑定和函数的声明位置、函数的调用位置没有任何关系，只取决于函数的调用方式**。

```js
function foo () {
    console.log(this.a)
}
var obj = {
    a: 2,
    foo: foo,
    other: function () {
        console.log(this.a)     // 2

        // 默认绑定
        foo()   // oops, global
    }
}
var bar = obj.foo
var a = 'oops, global'
// 默认绑定！！！
bar()       // oops, global

// 隐式绑定
obj.other()
```

### 显示绑定

通过`apply()`和`call()`方法来调用函数时，`this`的绑定为显示绑定，`this`会绑定到传给`apply()`或`call()`方法的第一个参数上。

```js
function foo () {
    console.log(this.a)
}
var obj = {
    a: 2
}

foo.call(obj)   // 2
```

如果传给`apply()`或`call()`方法的第一个参数为基本数据类型，那么这个基本类型值会被转换为对应的对象形式（new String()、 new Boolean()等）。

硬绑定是显示绑定的一个变种，硬绑定是使用`bind()`方法强制指定`this`，该方法返回一个新的函数实例。

### new绑定

使用`new`来调用函数，或者说发生构造函数调用时，会自动执行下面的操作：

{% note info %}
1. 创建一个全新的对象；
2. 这个新对象会被执行[[原型]]连接；
3. 这个新对象会绑定到函数调用的`this`；
4. 如果函数没有返回其他对象，那么`new`表达式中的函数调用会自动返回这个新对象。
{% endnote %}

```js
function foo (a) {
    this.a = a
}
var bar = new foo(2)
console.log(bar.a)  // 2
```

## 优先级

new > 显示 > 隐式 > 默认

## bind的Polyfill

[下面的代码来源于MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

```js
if (!Function.prototype.bind) {
    Function.prototype.bind = function(oThis) {
        if (typeof this !== 'function') {
            // closest thing possible to the ECMAScript 5
            // internal IsCallable function
            throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
        }

        var aArgs   = Array.prototype.slice.call(arguments, 1),
            fToBind = this,     // 利用闭包记住调用bind方法的函数
            fNOP    = function() {},
            fBound  = function() {
                // fBound是要被返回的新函数
                // 这个return里的this含义与外面的不同
                // 这里this的绑定取决于函数的调用方式是new操作符调用还是其他方式调用
                return fToBind.apply(this instanceof fBound
                    ? this      // 这里是new绑定的优先级高于显示绑定优先级的原因
                    : oThis,
                    // 这里是bind方法可以进行柯里化的原因
                    // 这里的arguments是fBound函数的，与外面的那个不是同一个
                    aArgs.concat(Array.prototype.slice.call(arguments)))
            }

        // 维护原型关系
        if (this.prototype) {
            // Function.prototype doesn't have a prototype property
            fNOP.prototype = this.prototype
        }
        fBound.prototype = new fNOP()

        return fBound
    }
}
```