---
title: 块级绑定
categories:
  - JavaScript
abbrlink: 3432e35f
date: 2019-03-24 17:43:55
---

# 块级声明

块指由花括号封闭的代码块。

块级声明包括 `let` 和 `const` 两种声明方式，首先介绍它们之间的共性特点：

{% note info %}
- **块级作用域**：块级声明的变量，无法在指定的块作用域外访问；
- **禁止重复声明**：如果一个标识符已经在代码块内部被定义，那么禁止在此代码块内部对同一个标识符进行块级声明，否则会抛出错误；
- **暂时性死区**：块级声明的变量不仅不会进行变量提升，在代码运行到声明处之前还会被存放在**暂时性死区（TDZ）**的区域内。如果在声明处之前试图访问变量会导致一个引用错误，即使是使用 `typeof` 运算符也是如此！
{% endnote %}

`let` 和 `const` 之间的差异在于，用 `const` 声明的变量会被认为是常量，意味着它们的值在被设置完成后就不能再被改变。如果使用 `const` 声明了一个对象，变量中保存的是对象的引用，所以只要不重写对象，其他的也都随意更改。

# 循环中的块级绑定

开发者最需要使用变量的块级作用域的场景，或许就是在 `for` 循环内，也就是想让一次性的循环计数器只能在循环内部使用。例如，以下代码在 JavaScript 中并不罕见：

```js
for (var i = 0; i < 10; i++) {
  console.log(i)
}

// i在此处仍然可被访问
// 10
console.log(i)		
```

在其他默认使用块级作用域的语言中，这个例子能够按照预期工作，也就是只有 `for` 才能访问变量 `i`。然后在JavaScript中，循环结束后 `i` 仍然可被访问，因为 `var` 声明导致了变量提升。若像如下代码那样换为使用 `let`，则会看到逾期行为：

```js
for (let i = 0; i < 10; i++) {
  console.log(i)
}

// i在此处不可访问，抛出错误
// Uncaught ReferenceError: i is not defined
console.log(i)		
```

## 循环内的函数

长期以来，`var` 的特点使得循环变量在循环作用域外仍然可被访问，于是在循环内部创建函数就变得很有问题。考虑如下代码：

```js
var funcs = []

for (var i = 0; i < 10; i++) {
  funcs.push(function () { console.log(i) })
}

funcs.forEach(function (func) {
  // 输出数值 10 十次
  func()		
})
```

为了修正上述问题，开发者在循环内使用立即调用函数表达式（IIFE），以便在每次迭代中强制创建变量的一个新副本，示例如下：

```js
var funcs = []

for (var i = 0; i < 10; i++) {
  funcs.push((function (value) {
    return function () {
      console.log(value)
    }
  }(i)))
}

funcs.forEach(function (func) {
  // 从 0 到 9 依次输出
  func()		
})
```

虽然解决了问题，但是这种写法相对繁琐。幸运的是，使用 `let` 与 `const` 的块级绑定可以在 ES6 中为你简化这个循环。

## 循环内的 let 声明

`let` 声明通过有效模仿上例中的 IIFE 的作用而简化了循环。在每次迭代中，都会创建一个新的同名变量并对其进行初始化。这意味着你可以完全省略IIFE从而获得预期的结果，就像这样：

```js
var funcs = []

for (let i = 0; i < 10; i++) {
  funcs.push(function () {
    console.log(i)
  })
}

funcs.forEach(function (func) {
  // 从 0 到 9 依次输出
  func()		
})
```

与使用 `var` 声明以及IIFE相比，这里代码能达到相同效果，但无疑更加简洁。在循环中 `let` 声明每次都创建了一个新的 `i` 变量，因此在循环内部创建的函数获得了各自的 `i` 副本，而每个 `i` 副本的值都在每次循环迭代声明变量的时候被确定了。这种方式在 `for-in` 和 `for-of` 循环中同样适用，如下所示：

```js
var funcs = []
var object = {
  a: true,
  b: true,
  c: true
}

for (let key in object) {
  funcs.push(function () {
    console.log(key)
  })
}

funcs.forEach(function (func) {
  // 依次输出a、b、c
  func()		
})
```

本例中的 `for-in` 循环体现出了与 `for` 循环相同的行为。每次循环，一个新的 `key` 变量绑定就被创建，因此每个函数都能够拥有它自身的 `key` 变量副本，结果每个函数都输出了一个不同的值。

需要重点了解的是：`let` 声明在循环内部的行为是在规范中特别定义的，而与不提升变量声明的特征没有必然联系。事实上，在早期 `let` 的实现中并没有这种行为，它是后来才添加的。

## 循环内的 const 声明

ES6规范没有明确禁止在循环中使用 `const` 声明，然而它会根据循环方式的不同而有不同行为。在常规的 `for` 循环中，你可以在初始化时使用 `const`，但循环会在你试图改变该变量的值时抛出错误。例如：

```js
var funcs = []

// 在一次迭代后抛出错误
for (const i = 0; i < 10; i++) {
  funcs.push(function () {
    console.log(i)
  })
}
```

在此代码中，`i` 被声明为一个常量。循环的第一次迭代成功执行，此时 `i` 的值为0。在 `i++` 执行时，一个错误会被抛出，因为该语句试图更改常量的值。因此，在循环中你只能使用 `const` 来声明一个不会被更改的变量。

而另一方面，`const` 变量在 `for-in` 或 `for-of` 循环中使用时，与 `let` 变量效果相同。因此下面代码不会导致出错：

```js
var funcs = []
var object = {
  a: true,
  b: true,
  c: true
}

// 不会导致出错
for (const key in object) {
  funcs.push(function () {
    console.log(key)
  })
}

funcs.forEach(function (func) {
  // 依次输出a、b、c
  func()		
})
```

这段代码与“循环内的 let 声明”小节的第二个例子几乎完全一样，唯一的区别是 `key` 的值在循环内不能被更改。`const` 能够在 `for-in` 与 `for-of` 循环内工作，是因为循环为每次迭代创建了一个新的变量绑定，而不是试图去修改已绑定的变量的值。

# 全局块级绑定

`let` 与 `const` 不同于 `var` 的另一个方面是在全局作用域上的表现。当在全局作用域上使用 `var` 时，它会创建一个新的全局变量，并成为全局对象（在浏览器中是 `window`）的一个属性。这意味着使用 `var` 可能会无意覆盖一个已有的全局属性，就像这样：

```js
// 在浏览器中
var RegExp = 'hello'

// hello
console.log(window.RegExp)      

var ncz = 'hi'

// hi
console.log(window.ncz)         
```

尽管全局的 `RegExp` 是定义在 `window` 上的，它仍然不能防止被 `var` 重写。这个例子声明了一个新的全局变量 `RegExp` 而覆盖了原有对象。类似地，`ncz` 定义为全局变量后就立即成为了 `window` 的一个属性。

**然而若你在全局作用域上使用 `let` 或 `const`，虽然在全局作用域上会创建新的绑定，但不会有任何属性被添加到全局对象上**。这也就意味着你不能使用 `let` 或 `const` 来覆盖一个全局变量，你只能将其屏蔽。这里有个范例：

```js
// 在浏览器中
let RegExp = 'hello'

// hello
console.log(RegExp)                         

// false
console.log(window.RegExp === RegExp)       

const ncz = 'hi'

// hi
console.log(ncz)                            

// false
console.log('ncz' in window)                
```

此代码的 `let` 声明创建了 `RegExp` 的一个绑定，并屏蔽了全局的 `RegExp`。这表示 `window.RegExp` 与 `RegExp` 是不同的，因此全局作用域没有被污染。同样，`const` 声明创建了 `ncz` 的一个绑定，但并未在全局对象上创建属性。当你不想在全局对象上创建属性时，这种特性会让 `let` 与 `const` 在全局作用域中更安全。

# 块级绑定新的最佳实践

在默认情况下使用 `const`，并且只在知道变量值需要被更改的情况下才使用 `let`。