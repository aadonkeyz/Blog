---
title: 函数&作用域&闭包
abbrlink: f1587c53
date: 2019-03-14 18:57:01
categories:
  - JavaScript
---

# 函数的定义

定义函数的两种常用方式为函数声明和函数表达式，他们有如下区别：

| 函数声明 | 函数函数表达式 |
| :-----: | :---------: |
| 会被提升 | 不会被提升 |
|名称标识符会存在于所在的作用域中 | 名称标识符只能在这个函数的内部被访问，外部作用域则不行 |

# 作用域是什么

## 引擎、编译器、作用域

{% note info %}
- **引擎**：从头到尾负责整个JavaScript程序的编译及执行过程。
- **编译器**：负责语法分析及代码生成。JavaScript的编译发生在代码执行前的几微秒（甚至更短！）的时间内。
- **作用域**：一套设计良好的、用来存储变量并能够根据名称方便地查找这些变量的规则。
{% endnote %}

以 `var a = 2` 为例，实际上引擎认为这里有两个完全不同的声明，一个由编译器在编译时处理，另一个由引擎在运行时处理。

{% note info %}
1. 首先，当遇到 `var a` 时编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的集合中（也就是说在**当前作用域**中该变量是否已经存在）。如果是，编译器会忽略该声明，否则它会要求作用域在当前作用域的集合中声明一个新的变量，并命名为 `a`;
2. 接下来，编译器会为引擎生成运行时所需的代码，这些代码被用来处理 `a = 2` 这个赋值操作。引擎运行时会首先询问作用域，在当前的作用域集合中是否存在一个叫做 `a` 的变量，如果是，引擎会使用这个变量。否则引擎会继续在外层作用域中查找该变量。
{% endnote %}

## 词法作用域和动态作用域

### 词法作用域

词法作用域是由你在写代码时将变量、块和函数写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变（大部分情况下是这样的，不过存在例外）。

**无论函数在哪里被调用，也无论它如何被调用，它的词法作用域都只由函数被声明时所处的位置决定。**

```js
function foo () {
  console.log(a)
}

function bar () {
  var a = 3
  foo()
}

var a = 2

// 2
bar()
```

词法作用域查找只会查找一级标识符，例如如果代码中引用了 `a.b.c`，词法作用域查找只会试图查找 `a` 标识符，找到这个变量之后，它就不管了，后续的 `b` 和 `c` 标识符的查找被交给对象属性访问规则了。

`eval()` 和 `with` 的使用都会欺骗词法作用域，如果引擎在代码中发现了 `eval()` 或 `with`，它只能简单地假设关于标识符位置的判断都是无效的，因为无法确定 `eval()` 或 `with` 会带来什么样的影响。这样造成的结果就是引擎无法在编译阶段完成一些性能的优化从而造成性能下降。

### 动态作用域

动态作用域并不关心函数和作用域是如何声明以及在何处声明的，只关心它们从何处调用。换句话说，**动态作用域链是基于调用栈的，而不是代码中的作用域嵌套**。因此，如果 JavaScript 具有动态作用域，理论上，下面代码中的 `foo()` 在执行时会输出 3。

```js
function foo () {
  console.log(a)
}

function bar () {
  var a = 3
  foo()
}

var a = 2

// 3
bar()
```

为什么会这样？因为当 `foo()` 无法找到 `a` 变量的引用时，会顺着调用栈在调用 `foo()` 的地方查找 `a`，而不是在嵌套的词法作用域链中向上查找。由于 `foo()` 是在 `bar()` 中调用的，引擎会检查 `bar()` 的作用域，并在其中找到值为 3 的变量 `a`。

{% note warning %}
**需要明确的是，事实上 JavaScript 并不具有动态作用域。它只有词法作用域，简单明了。但是 `this` 机制某种程度上很像动态作用域。**
{% endnote %}

## 执行环境、变量对象、作用域链

当一个块或函数嵌套在另一个块或函数中时，就发生了作用域的嵌套。当引擎查找某个变量时，首先从当前作用域开始查找，如果可以找到则返回该变量。如果在当前作用域中无法找到某个变量时，引擎就会在外层嵌套的作用域中继续查找，直到找到该变量，或抵达最外层的作用域（也就是全局作用域）为止。

那么引擎怎么知道当前作用域的外层作用域是谁，又该怎么找到它呢？答案是通过作用域链。

{% note info %}
- **执行环境**：每个块或函数都有自己的执行环境，执行环境中定义了在块或函数内部有权访问的其他数据，决定了它们各自的行为。全局执行环境是最外围的一个执行环境。
- **变量对象**：执行环境中定义的所有变量和函数都保存在这个对象中，我们在编程时无法访问这个对象。如果执行环境是一个函数，则将其活动对象作为变量对象。活动对象在最开始时只包含一个变量（函数的参数），即 `arguments` 对象（箭头函数没有 `arguments` 对象，而且不鼓励使用 `arguments` 对象）。
- **（词法）作用域链**：从当前执行环境的变量对象出发，将外层每一个执行环境的变量对象串联起来，保证对当前执行环境有权访问的所有数据的有序访问。作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。
{% endnote %}

## LHS 和 RHS

{% note info %}
- **LHS**：试图找到变量的**容器本身**，并对其进行**赋值**；
- **RHS**：找到该变量**对应的值**，并**使用**这个值。
{% endnote %}

**将函数声明理解为先声明变量、再进行赋值并不合适。因为这样理解的话这个函数声明将需要进行LHS查询，但实际中并不是这样的。**

为什么区分 LHS 和 RHS 是一件很重要的事情？因为在变量还没有声明（在任何作用域中都无法找到该变量）的情况下，这两种查询的行为是不一样的。

如果 RHS 查询不到所需的变量，引擎就会抛出 ReferenceError 异常。

相比之下，当 LHS 查询不到变量时，如果在非严格模式下，就会在**全局作用域**中创建一个具有该名称的变量。但如果是在非严格模式下，就会抛出错误。

## 函数作用域和块作用域

### 函数作用域

函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复用。

根据作用域嵌套和作用域链的概念可以知晓，内部作用域可以访问外部作用域中的变量，但是反过来则不可以。

```js
var outter = '我在外面'
function foo () {
  var inner = '我在里面'
  console.log(outter)
}

// 我在外面
foo()       

// Uncaught ReferenceError: inner is not defined
console.log(inner)  
```

在任意代码片段外部添加包装函数，可以将内部的变量和函数定义“隐藏”起来，外部作用域无法访问包装函数内部的任何内容。

```js
var a = 2
function foo () {
  var a = 3
  console.log(a)
}

// 3
foo()

// 2
console.log(a)  
```

虽然这种技术可以解决一些问题，但是也产生了一些副作用。首先必须声明一个具名函数 `foo()`，意味着 `foo` 这个名称本身“污染”了所在作用域，其次必须显示地调用 `foo()`。下面介绍一种解决这两个问题的例子：

```js
var a = 2;
(function foo () {
  var a = 3

  // 3
  console.log(a)  

  // function
  console.log(typeof foo)    
})()

// 2
console.log(a)  

// Uncaught ReferenceError: foo is not defined
console.log(foo)    
```

首先，包装函数的声明是以圆括号开始的，尽管开上去这并不是一个很显眼的细节，但实际上确实非常重要的区别。函数会被当做函数表达式而不是一个标准的函数声明来处理。

上面的代码中使用了 IIFE：**立即执行函数表达式**。在一个函数表达式后面加上一对圆括号就表示要立即调用这个函数。

{% note warning %}
**函数声明和函数表达式之间最重要的区别是它们的名称标识符会绑定在何处！函数声明的名称标识符会存在于所在的作用域中。而函数表达式的名称标识符只能在这个函数的内部被访问，外部作用域则不行。**
{% endnote %}

```js
// 实际的代码
var foo = function bar () {
  // ...
}

// 等价于
var foo = function () {
  var bar = ...self...
  // ...
}
```

### 块作用域

#### with

用 `with` 从对象中创建出的作用域仅在 `with` 声明中而非外部作用域中有效。

#### try/catch

非常少有人会注意到 ES3 规范中规定 `try/catch` 的 `catch` 分句会创建一个块作用域，其中声明的变量仅在 `catch` 内部有效。例如：

```js
try {
  // 执行一个非法操作来强制制造一个异常
  undefined()         
}
catch (err) {
  // 能够正常执行！
  console.log(err)    
}

// ReferenceError: err not found
console.log(err)        
```

#### let 和 const

ES6 中引入了新的 `let` 关键字，提供了除 `var` 以外的另一种变量声明方式。`let` 关键字可以将变量绑定到所在的任意作用域中（通常是{...}内部）。

```js
var foo = true
if (foo) {
  let bar = 2

  // 2
  console.log(bar)    
}

// Uncaught ReferenceError: bar is not defined
console.log(bar)        
```

`const` 和 `let` 的性质类似，主要的区别在于 `const` 是用来定义常量的。有关它们的详细特性后续就不在这里展开说了。

# 提升

## 如何提升

提升：包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。但是只有声明本身会被提升，而赋值或其他运行逻辑会留在原地。

另外值得注意的是，每个作用域都会进行提升操作。

```js
// ========实际代码=========
foo()
function foo() {
  console.log(a)
  var a = 2
}

// =====编译器眼中的代码=====
function foo() {
  var a
  console.log(a)
  a = 2
}
foo()
```

函数声明会被提升，但是函数表达式不会。

```js
/**
 * 提升前
 */

// TypeError
foo()   

// ReferenceError
bar()   
var foo = function bar () {
  // ...
}

/**
 * 提升后
 */

var foo

// TypeError
foo()  

// ReferenceError
bar()   

foo = function () {
  var bar = ...self...
  // ...
}
```

## 函数优先

函数声明和变量声明都会被提升。但是一个值得注意的细节是函数会首先被提升，然后才是变量。

```js
/**
 * 提升前
 */

// 1
foo ()    

var foo

function foo () {
  console.log(1)
}

foo = function () {
  console.log(2)
}

/**
 * 提升后
 */

function foo () {
  console.log(1)
}

// 1
foo () 

foo = function () {
  console.log(2)
}
```

注意到，`var foo` 尽管出现在 `function foo()` 的声明之前，但是函数声明会首先被提升，所以 `var foo` 被当作重复的声明而忽略了。

如果有重复的函数声明，后面的会覆盖前面的，看下面例子：

```js
// 3
foo()   

function foo () {
  console.log(1)
}

var foo = function () {
  console.log(2)
}

function foo () {
  console.log(3)
}
```

# 闭包

## 什么是闭包

闭包：当函数可以记住并访问所在的**词法作用域**时，就产生了闭包，即使函数是在当前词法作用域之外执行。

```js
function foo () {
  var a = 2
  function bar () {
    console.log(a)
  }
  return bar
}

var baz = foo()

// 2
// 这就是闭包的效果
baz()       
```

在 `foo()` 执行后，通常会将 `foo()` 的整个内部作用域都销毁，但是由于闭包的存在，`foo()` 的内部作用域被保存下来了。拜 `bar()` 所声明的位置所赐，它拥有涵盖 `foo()` 内部作用域的闭包，使得该作用域能够一直存活，以供 `bar()` 在之后任何时间进行引用。如果想要释放被闭包住的内存，需要解除对 `bar` 的引用，即执行 `baz = null`。

## 循环和闭包

先运行下面的例子，观察结果：

```js
// 以每秒一次的频率输出五次 6
for (var i = 1; i <= 5; i++) {
  (function () {
    setTimeout(function timer () {
      console.log(i)
    }, i * 1000)
  })()
}

// 以每秒一次的频率以此输出 1、2、3、4、5
for (var i = 1; i <= 5; i++) {
  (function (j) {
    setTimeout(function timer () {
      console.log(j)
    }, j * 1000)
  })(i)
}
```

我们通过下面的例子，详细解释闭包与循环的爱恨纠葛。

```js
function createFunctions () {
  var result = new Array()

  for (var i = 0; i < 3; i++) {
    result[i] = function () {
      return i
    }
  }
  return result
}
var array = createFunctions()

// 3
console.log(array[0]())

// 3
console.log(array[1]())

// 3
console.log(array[2]())     


// 上面的等价形式
function createFunctions () {
  var result = new Array()

  var i = 0
  result[i] = function () {
    return i
  }

  i = 1
  result[i] = function () {
    return i
  }

  i = 2
  result[i] = function () {
    return i
  }

  i = 3
  return result
}
var array = createFunctions()

// 3
console.log(array[0]())

// 3
console.log(array[1]()) 

// 3
console.log(array[2]())   
```

内部函数中使用了 `createFunctions` 的活动对象的属性 `i`，而在真正调用内部函数时，这个 `i` 的值是 3，所以它们返回的都是 3。

实际上在 `for` 循环中使用 `let` 代替 `var` 就可以轻松解决这个问题，具体原因就先不赘述了。

```js
function createFunctions () {
  var result = new Array()

  for (let i = 0; i < 3; i++) {
    result[i] = function () {
      return i
    }
  }
  return result
}
var array = createFunctions()

// 0
console.log(array[0]())

// 1
console.log(array[1]())

// 2
console.log(array[2]())
```

# 私有变量

严格来讲，JavaScript 中没有私有成员的概念，所有对象属性都是公开的。不过，倒是有一个私有变量的概念。任何在函数中定义的变量，都可以认为是私有变量，因为不能再函数的外部访问这些变量。私有变量包括函数的参数、局部变量和在函数内部定义的其他函数。

我们把有权访问私有变量和私有函数的公有方法成为特权方法。可以在构造函数中定义特权方法，基本模式如下：

```js
function MyObject () {
  // 私有变量和私有函数
  var privateVariable = 10
  function privateFunction () {
    return false
  }

  // 特权方法
  this.publicMethod = function () {
    privateVariable++
    return privateFunction()
  }
}
```

在创建 `MyObject` 的实例后，除了使用 `publicMethod()` 这一个途径外，没有任何办法可以直接访问 `privateVariable` 和 `privateFunction()`。

不过在构造函数中定义特权方法也有一个缺点，那就是你必须使用构造函数模式来达到这个目的。构造函数的缺点是针对每一个实例都会创建同样一组新方法，而使用静态私有变量来实现特权方法就可以避免这个问题。

## 静态私有变量

```js
(function () {
  // 私有变量和私有函数
  var privateVariable = 10
  function privateFunction () {
    return false
  }

  // 构造函数
  MyObject = function () {}
  MyObject.prototype.publicMethod = function () {
    privateVariable++
    return privateFunction()
  }
})()
```

这个模式创建了一个私有作用域，并在其中封装了一个构造函数及相应的方法。需要注意的是，这个模式在定义构造函数时并没有使用函数声明，而是使用了函数表达式，函数声明只能创建局部函数，但那并不是我们想要的。出于同样的原因，我们也没有在声明 `MyObject` 时用 `var` 关键字（记住：初始化未经声明的变量，在非严格模式下会创建一个全局变量，在严格模式下会抛出错误），这样 `MyObject` 就成了一个全局变量。而且特权方法是在原型上定义的，因此所有实例都能使用同一个函数。这个特权方法作为一个闭包，总是保存着对包含作用域的引用。

{% note warning %}
**以这种方式创建静态私有变量会因为使用原型而增进代码复用，但每个实例都没有自己的私有变量。**
{% endnote %}

## 模块模式

前面的模式是用于为自定义类型创建私有变量和特权方法的。而道格拉斯所说的模块模式则是为单例创建私有变量和特权方法。所谓单例，指的就是只有一个实例的对象。

```js
var singleton = function () {
  var privateVariable = 10
  function privateFunction () {
    return false
  }

  // 特权/公有方法和属性
  return {
    publicProperty: true,
    publicMethod: function () {
      privateVariable++
      return privateFunction()
    }
  }
}()
```

## 增强的模块模式

这种增强的模块模式适合那些单例必须是某种类型的实例，同时还必须添加某些属性和方法对其加以增强的情况。

```js
var singleton = function () {
  var privateVariable = 10
  function privateFunction () {
    return false
  }

  // 创建对象
  var object = new CustomType()

  // 添加特权/公有属性和方法
  object.publicProperty = true
  object.publicMethod = function () {
    privateVariable++
    return privateFunction()
  }

  // 返回这个对象
  return object
}()
```