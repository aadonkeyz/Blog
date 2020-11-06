---
title: import 与 export
categories:
  - JavaScript
abbrlink: 588c1481
date: 2019-04-17 16:05:23
---

# 何为模块？

{% note info %}
模块是使用不同方式加载的 JS 文件（与 JS 原先的脚本加载方式相对）。这种不同模式很有必要，因为它与脚本（script）有大大不同的语义：
- 模块代码自动运行在严格模式下，并且没有任何办法跳出严格模式。
- 在模块的顶级作用域创建的变量，不会被自动添加到共享的全局作用域，它们只会在模块顶级作用域内部存在。
- 模块顶级作用域的 `this` 值为 `undefined`。
- 模块不允许在代码中使用HTML风格的注释。
- 对于需要让模块外部代码访问的内容，模块必须导出它们。
- 允许模块从其他模块导入绑定。
{% endnote %}

CommonJS 是在 ES6 之前的模块规范，至今仍然被广泛使用。[**点击这里了解 CommonJS**](https://aadonkeyz.com/posts/b3fe2fad/)！

# 导出

ES6 的导出有两种形式：命名导出和默认导出。

## 命名导出

命名导出要求导出的内容必须有自己的标识符，这就意味着不能使用这种方式来导出匿名函数或匿名类。

你可以将 `export` 放置在任意变量、函数或类声明之前，从模块中将它们公开出去，就像这样：

```js
export var color = 'red'

export let name = 'Nicholas'

export const magicNumber = 7

export function sum(num1, num2) {
  return num1 + num1
}

export class Rectangle {
  constructor (length, width) {
    this.length = length
    this.width = width
  }
}
```

或者使用花括号包裹想要导出内容的标识符，一起将它们导出。在使用这种方式的时候，允许你对导出的内容进行重命名：

```js
function multiply (num1, num2) {
  return num1 * num2
}

function sum (num1, num2) {
  return num1 + num2
}

export { multiply, sum as add }
```

## 默认导出

模块的默认值是使用 `default` 关键字所指定的单个变量、函数或类，而你在每个模块中最多只能设置一个默认导出，将 `default` 关键字用于多个导出会是语法错误。

使用 `default` 可以导出匿名函数或匿名类，实际上默认导出并不在乎导出内容的标识符，就算导出的内容原本是有标识符的，在其他模块中引入时，也会忽略它原本的标识符。

以下三种用法都是正确的：

```js
export default function (num1, num2) {
  return num1 + num2
}
```
```js
function sum (num1, num2) {
  return num1 + num2
}

export default sum
```
```js
function sum (num1, num2) {
  return num1 + num2
}

export { sum as default }
```

**使用 `default` 导出的内容必须是已经声明过或正在声明的，否则会抛出错误：**

```js
// ReferenceError: sum is not defined
export default sum = function () {}
```

# 导入

## 导入的语法

创建了 a.js 文件用于导出，b.js 文件用于导入：

```js
// a.js
export var count = 0
export var color = 'red'

export function multiply (num1, num2) {
  return num1 * num2
}

export function sum (num1, num2) {
  return num1 + num2
}

var num = 666
export default num
```
```js
// b.js
import { count } from './a.js'
import { sum as add, multiply } from './a.js'

// 0
console.log(count)        

// [Function: sum]
console.log(add)            

// [Function: multiply]
console.log(multiply)       

import * as moduleA from './a.js'

// {
//      multiply: [Function: multiply],
//      sum: [Function: sum],
//      count: 0,
//      color: 'red',
//      default: 666
// }
console.log(moduleA)        

import default1 from './a.js'

// 666
console.log(default1)       

import { default as default2 } from './a.js'

//666
console.log(default2)       

import default3, { color } from './a.js'

// 666
console.log(default3)       

// red
console.log(color)          
```

{% note warning %}
根据上面例子，总结以下几点注意事项：
- 在使用 `import` 引入一个模块的内容时，除了默认导出，其他导出均需要根据标识符来进行匹配。
- 默认导出是一个特立独行的存在，它在原模块中的标识符是被忽略的。比如上面例子中的 `moduleA` 对象不包含 `num` 属性，但是包含 `default` 属性。
{% endnote %}

## 只读绑定

ES6 的 `import` 语句为变量、函数与类创建了**只读绑定**，不允许修改这个只读绑定的值，否则会抛出错误。

{% note warning %}
**其实只要不是重写这个导入的变量，其他任何操作都是可以的，比如修改对象的属性。**
{% endnote %}

```js
// a.js
export var name = 'Nicholas'

export var obj = {
  a: 1
}

export function setName (newName) {
  name = newName
}
```
```js
// b.js
import { name, obj, setName } from './a.js'

// Nicholas
console.log(name)       

setName('Greg')

// Greg
console.log(name)       

obj.a = 2

// {a: 2}
console.log(obj)      

// SyntaxError: "name" is read-only
name = 'Nicholas'       
```

调用 `setName('Greg')` 会回到导出 `setName()` 的模块内部，并在那里执行，从而将 a.js 文件内的 `name` 设置为 `'Greg'`，注意这个变化会自动反映到 b.js 文件内所导入的 `name` 绑定上。

## 模块的实例化与缓存

{% note warning %}
**无论你对同一个模块使用了多少次 `import` 语句，该模块都只会被执行一次。在导出模块的代码执行之后，已被实例化的模块就被保留在内存中，并随时都能被其他 `import` 所引用。**
{% endnote %}

```js
// a.js
import * as b from './b.js'
import * as c from './c.js'

// a.js: b change c.obj
console.log(`a.js: ${c.obj.b}`); 
```

```js
// b.js
import * as c from './c.js'

c.obj.b = 'b change c.obj'
```

```js
// c.js
export let obj = {};
```

# 绑定的再导出

也许有时你会想将当前模块已导入的内容重新再导出，你可以这样做：

```js
import { sum } from './example.js'
export { sum }
```

此方法能奏效，但还可以使用单个语句来完成相同的任务：

```js
export { sum } from './example.js'
```

这种形式的 `export` 会进入指定模块查看 `sum` 的定义，随后将其导出。当然，你也可以选择一个值用不同名称导出：

```js
export { sum as add } from './example.js'
```

若你想将来自另一个模块的所有值完全导出，可以使用星号（`*`）模式：

```js
export * from './example.js'
```

使用完全导出，就可以导出目标模块的默认值及所有具名导出，但这可能影响你从当前模块所能导出的值。例如，假设 example.js 具有一个默认导出，当你使用这种语法时，你就无法为当前模块另外再定义一个默认导出。

# 无绑定的导入

有些模块也许没有进行任何导出，相反只是修改全局作用域的对象。尽管这种模块的顶级变量、函数或类最终并不会自动被加入全局作用域，但这并不意味着该模块无法访问全局作用域。诸如 `Array` 与 `Object` 之类的内置对象的共享定义在模块内部可访问的，并且对于这些对象的修改会反映到其他模块中。

例如，若你想为所有数组添加一个 `pushAll()` 方法，你可以像下面这样定义一个模块文件 a.js：

```js
// a.js
// 没有导出与导入的模块
Array.prototype.pushAll = function(items) {
  // items 必须是一个数组
  if (!Array.isArray(items)) {
    throw new TypeError('Argument must be an array.')
  }
  
  // 使用内置的 push() 与扩展运算符
  return this.push(...items)
}
```

这是一个有效的模块，尽管此处没有任何导出与导入。此代码可以作为模块或脚本来使用。由于它没有导出任何东西，你可以使用简化的导入语法来执行此模块的代码，而无须导入任何绑定：

```js
// b.js
import './a.js'

let colors = ['red', 'green', 'blue']
let items = []

items.pushAll(colors)

// [ 'red', 'green', 'blue' ]
console.log(items)      
```

此代码导入并执行了包含 `pushAll()` 的模块，于是 `pushAll()` 就被添加到数组的原型上。这意味着现在 `pushAll()` 在当前模块内的所有数组上都可用。

# import/export 的限制

`import` 与 `export` 都有一个重要的限制，那就是它们必须被用在其他语句或表达式的外部。例如，以下代码会抛出错误：

```js
function tryImport () {
  // SyntaxError: 'import' and 'export' may only appear at the top level
  import flag from './a.js'    
}
```
```js
let flag = true

if (flag) {
  // SyntaxError: 'import' and 'export' may only appear at the top level
  export flag                 
}
```

# 模块的循环引用

ES6 根本不会关心是否发生了"循环加载"，只是生成一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

```js
// even.js
import { odd } from './odd.js'

export var counter = 0

export function even(n) {
  counter++

  return n == 0 || odd(n - 1)
}
```
```js
// odd.js
import { even } from './even'

export function odd(n) {
  return n != 0 && even(n - 1)
}
```
```js
// main.js
import * as m from './even.js'

// true
console.log(m.even(10))     

// 6
console.log(m.counter)      

// true
console.log(m.even(20))     

// 17
console.log(m.counter)      
```

调用 `m.even(10)` 时，参数 `n` 从 10 变为 0 的过程中，`even()` 一共会执行 6 次，所以 `counter` 等于 6。调用 `m.even(20)` 时，参数 `n` 从 20 变为 0，`even()` 一共会执行 11 次，加上前面的 6 次，所以变量 `counter` 等于 17。