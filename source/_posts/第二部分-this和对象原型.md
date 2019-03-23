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
ps：如果对象的某个属性是函数，就称这个属性为对象的方法。

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

**apply()和call()是要指定函数运行时的this并运行函数，而bind()是返回一个this已经绑定完的函数实例**。

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
        // fNOP的存在我觉得是多余的，但是MDN上是这么写的，具体原因我没有深究
        if (this.prototype) {
            // Function.prototype doesn't have a prototype property
            fNOP.prototype = this.prototype
        }
        fBound.prototype = new fNOP()

        return fBound
    }
}
```

## 绑定例外

### 被忽略的this

如果你把`null`或者`undefined`作为`this`的绑定对象传入`call`、`apply`或者`bind`，这些值在调用时会被忽略，实际应用的是默认绑定规则。一般都是在函数并不关心`this`的情况下才会传入`null`或者`undefined`作为参数。

但是如果某个函数确实使用了`this`，那么这种做法就会产生一些副作用（如修改全局对象）。

所以比较推荐的做法是传入`Object.create(null)`替代`null`和`undefined`。

### 间接引用

另一个需要注意的是，你有可能（有意或者无意地）创建一个函数的“间接引用”，在这种情况下，调用这个函数会应用默认绑定规则。

```js
function foo () {
    console.log(this.a)
}

var a = 2
var o = { a: 3, foo: foo }
var p = { a: 4 }

o.foo()     //3
(p.foo = o.foo)()   // 2
```

赋值表达式`p.foo = o.foo`的返回值是目标函数的引用，因此`(p.foo = o.foo)()`等价于`foo()`，所以会应用默认绑定。

### 软绑定

软绑定的原理类似于硬绑定，直接上代码。

```js
if (!Function.prototype.softBind) {
    Function.prototype.softBind = function (obj) {
        var fn = this
        var curried = [].slice.call(arguments, 1)
        var bound = function () {
            return fn.apply(
                (!this || this === (window || global)) ? obj : this),
                curried.concat.apply(curried, arguments
            )
        }

        bound.prototype = Object.create(fn.prototype)
        return bound
    }
}

function foo () {
    console.log(this.name)
}

var obj = { name: 'obj' }
var obj2 = { name: 'obj2' }
var obj3 = { name: 'obj3' }

var fooObj = foo.softBind(obj)

fooObj()        // obj

obj2.foo = foo.softBind(obj)
obj2.foo()      // obj2

fooObj.call(obj3)   // obj3

setTimeout(obj2.foo, 10)    // obj
```

可以看到，软绑定版本的`foo()`可以手动将`this`绑定到`obj2`或者`obj3`上，但如果应用默认绑定，则会将`this`绑定到`obj`。

## this词法

ES6的箭头函数并不使用`function`关键字定义，而是使用被称为“胖箭头”的操作符 `=>` 定义的。箭头函数不使用`this`的四种标准规则，而是根据外层（函数或者全局）作用域来决定`this`。

```js
function foo () {
    // 返回一个箭头函数
    return (a) => {
        // this继承自foo()
        console.log(this.a)
    }
}

var obj1 = { a: 2 }
var obj2 = { a: 3 }

var bar = foo.call(obj1)
bar.call(obj2)      // 2
```

`foo()`内部创建的箭头函数会捕获调用时`foo()`的`this`。由于`foo()`的`this`绑定到`obj1`，`bar`（引用箭头函数）的`this`也会绑定到`obj1`，**箭头函数的绑定无法被修改（new也不行！）**

**箭头函数可以像`bind()`一样确保函数的`this`被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域取代了传统的this机制**。

# 对象、原型、继承

**该章的对象、混合对象的“类”、原型小节与[《JavaScript高级程序设计》的第六章-面向对象的程序设计](https://aadonkeyz.com/posts/bb5d40f2/)多有重复，就不重复介绍了。下面介绍对象的三个方法。**

## 禁止扩展

如果你想禁止一个对象添加新属性并且保留已有属性，可以使用`Object.preventExtensions()`：

```js
var myObject = { a: 2 }
Object.preventExtensions(myObject)

myObject.b = 3
myObject.b      // undefined
```

在非严格模式下，创建属性`b`会静默失败。在严格模式下，将会抛出`TypeError`错误。

## 密封

`Object.seal()`会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用`Object.preventExtensions()`并把所有现有属性标记为`configurable:false`。

## 冻结

`Object.freeze()`会创建一个冻结对象，这个方法实际上会在一个现有对象上调用`Object.seal()`并把所有“数据访问”属性标记为`writable:false`。
