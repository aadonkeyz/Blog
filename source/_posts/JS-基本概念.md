---
title: 基本概念
categories:
  - JavaScript
abbrlink: 2c31c1a2
date: 2019-03-01 15:49:01
---

# 变量

ECMAScript 的变量是松散类型的，可以用来保存任何类型的数据，换句话说，变量只是一个用于保存值得占位符而已，变量没有类型，数据才有类型！

{% note info %}
- 使用 `var` 声明变量，变量为声明该变量的作用域中的局部变量。即在全局作用域中声明的变量为全局变量，在局部作用域中声明的变量为局部变量。
- 使用 `var` 声明的变量如果未初始化，则变量保存的值默认为 `undefined`。
- 对没有声明的变量直接进行赋值操作（或者说是 LHS 操作），在非严格模式下，会创建一个全局变量。但是在严格模式下，会抛出错误。
{% endnote %}

# 数据类型

JavaScript 的数据类型分为“基本数据类型”和“引用数据类型”两种。

{% note info %}
- **基本数据类型**：`Undefined`、`Null`、`Boolean`、`Number`、`String` 和 `Symbol`（ES6 提出）。
- **引用数据类型**：`Object`。
{% endnote %}

## 检测类型

### typeof 操作符

对一个值使用 `typeof` 操作符，可能返回下列某个字符串（均为英文单词小写形式）：

{% note info %}
- `"undefined"`: 如果这个值未定义。
- `"boolean"`: 如果这个值是布尔值。
- `"string"`: 如果这个值是字符串。
- `"number"`: 如果这个值是数值。
- `"symbol"`: 如果这个值是符号。
- `"function"`: 如果这个值是函数（其实函数属于对象的一种）。
- `"object"`: 如果这个值是对象或 `null`。
{% endnote %}

### instanceof 操作符

用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

`object instanceof constructor` => `true | false`

《JavaScript 高级程序设计》中说：“~~根据规定，所有引用数据类型的值都是Object的实例。因此，在检测一个引用数据类型值和Object构造函数时，instanceof操作符始终会返回true。~~”
**随着版本的迭代，这个说法变得不准确了，有例外了！！！**

```js
var a = Object.create(null)
console.log(a instanceof Object)    // false
```

### Object.prototype.toString.call()

返回一个表示类型的字符串，详情请看 [MDN官网](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)。

## 基本数据类型

### Undefined

`Undefined` 类型只有一个值，即特殊的 `undefined`。在使用 `var` 声明变量但未对其加以初始化时，这个变量的值就是 `undefined`。对未初始化的变量执行 `typeof` 操作会返回 `undefined` ，而对未声明的变量执行 `typeof` 操作同样会返回 `undefined`。

除了可以对未声明的变量执行 `typeof` 操作之外，在非严格模式下还可以对其进行 `delete` 操作。除了这两种情况之外，对未声明的变量进行任何其他操作，都会抛出错误。

### Null

`Null` 类型也只有一个值 - `null`。

```js
typeof null === 'object'    // true
null instanceof Object      // false
null == undefined           // true
Boolean(null) === false     // true
```

### Boolean

`Boolean` 类型的字面值 `true` 和 `false` 是区分大小写的。也就是说，`True` 和 `False` 都不是 `Boolean` 值，只是标识符。

### Number

{% note info %}
- 八进制字面量在严格模式下是无效的，会导致支持的JavaScript引擎抛出错误。
- 浮点数值的最高精度是17位小数，但在进行算术计算时其精确度远远不如整数。
- 绝对值超过 `Number.MAX_VALUE` 的值会被自动转换为 `infinity` 或 `-infinity`。
- 可以使用 `isFinite()` 函数来确定数值是否为无穷。如果不是无穷，则返回 `true`。
- `NaN` 与自身都不相等，可以使用 `isNaN()` 或 `Number.isNaN()` 确定数值是否为 `NaN`。在传入数据非数值时，`isNaN()` 会首先调用 `Number()` 将其转换为数值类型再进行判断，而 `Number.isNaN()` 则会直接返回 `false`。
- `Number()` 用于把任何类型数据转换为数值。
- `parseInt()` 和 `parseFloat()` 用于把字符串转换为数值。
- `parseInt()` 在转换字符串时比 `Number()` 好用，但是最好为 `parseInt()` 提供第二个参数（转换的基数2、8、10、16）。
{% endnote %}

### String

{% note info %}
- 任何字符串的长度都可以通过访问其 `length` 属性取得，包含双子节字符的可能不准确。
- 字符串是不可变的，一旦创建就不能改变，要改变某个变量保存的字符串，首先要销毁原来的字符串然后再用新的替代。
- 数值、布尔值、对象和字符串值都有 `toString()` 方法，并且数值的 `toString()` 方法可以接收一个参数（数值的基数），但是 `null` 和 `undefined` 没有。
- 在不知道要转换值的类型时，可以使用 `String()` 方法。
{% endnote %}

## 引用数据类型

### Object

**ECMAScript 中的对象其实就是一组数据和功能的集合，它的属性或方法是没有顺序之分的！！！**

Object 类型是所有它的实例的基础，它所具有的任何属性和方法也同样存在于更具体的对象中。Object 的每个实例都具有下列属性和方法：

{% note info %}
- `constructor`
- `hasOwnProperty(propertyName)`
- `isPrototypeOf(object)`
- `propertyIsEnumerable(propertyName)`
- `toLocaleString()`
- `toString()`
- `valueOf()`
{% endnote %}

## 复制变量值

在将一个值赋给变量时，如果这个值是基本数据类型，则变量中保存的是这个值本身。如果这个值是引用数据类型，则变量中保存的只是该值的一个引用（通过引用，可以在内存中找到这个引用数据类型的值）！

```js
// ===================================================
// 在从一个变量向另一个变量复制基本数据类型值和引用数据类型值的区别
// ===================================================

//复制基本数据类型值
var a = 1
var b = a
b = 2
console.log(a, b)   // 1, 2

// 复制引用数据类型值
// c 和 d 中保存的都是指向内存中同一对象的引用
var c = { num: 1 }
var d = c
d.num = 2
console.log(c.num, d.num)   //2, 2
```

## 传递参数

ECMAScript 中所有函数的参数都是按值传递的。简单来说就是数据进行复制，然后传递给给函数作为参数。

# 语句

## for-in语句

ECMAScript 对象的属性没有顺序。因此，通过 `for-in` 循环输出的属性名的顺序是不可预测的。

`for-in` 不能对 `null` 和 `undefined` 进行迭代。

## switch语句

可以在 switch 语句中使用任何数据类型（在很多其他语言中只能使用数值），并且每个 case 的值不一定是常量，可以是变量，甚至是表达式。

switch对 case 进行匹配时，遵循的是 `===` 严格相等。

# 函数

## 理解参数

**首先说明我的观点，不推荐使用 `arguments` ！！！**

ECMAScript 中的函数参数是用一个数组来表示的，可以通过 `arguments` 对象来访问这个数组，`arguments` 是一个类数组对象。

关于 `arguments` 和 `命名参数` 之间是如何相互影响的，《JavaScript高级程序设计》与《深入理解ES6》的说法有些矛盾，我在谷歌浏览器中简单的试了一下，他们之间的同步关系好像是双向的。

## 没有重载

ECMAScript 中的函数没有重载，如果声明了多个同名函数，后面的会覆盖前面的。

# 执行环境及作用域

{% note info %}
- **执行环境**：每个函数都有自己的执行环境，执行环境中定义了变量或函数有权访问的其他数据，决定了它们各自的行为。全局执行环境是最外围的一个执行环境；
- **变量对象**：执行环境中定义的所有变量和函数都保存在这个对象中，我们在编程时无法访问这个对象。如果执行环境是一个函数，则将其活动对象作为变量对象。活动对象在最开始时只包含一个变量（函数的参数），即 `arguments` 对象（箭头函数没有 `arguments` 对象......而且不鼓励使用 `arguments` 对象）；
- **作用域**：一套设计良好的、用来存储变量并能够根据名称方便地查找这些变量的规则（《你不知道的JavaScript（上卷）》）；
- **作用域链**：从当前执行环境的变量对象出发，将外层每一个执行环境的变量对象串联起来，保证对当前执行环境有权访问的所有变量和函数的有序访问。作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。
{% endnote %}

每个函数都有自己的执行环境，在访问一个变量或函数时，首先会在当前环境下的变量对象中查找，如果没有找到就沿着作用域链逐层查找。

# 垃圾收集

JavaScript 中的变量分为全局变量和局部变量，其中全局变量一直存在，不会被清除。

而局部变量只在函数执行的过程中存在，当函数执行结束后，局部变量会被自动清除。

当然如果存在闭包的话，局部变量被清除的时机需要取决于闭包。