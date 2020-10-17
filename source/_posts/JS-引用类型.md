---
title: 引用类型详解
categories:
  - JavaScript
abbrlink: 71de0ab2
mathjax: true
date: 2019-03-03 12:14:38
---

# Object类型

**我们看到的大多数引用类型值都是 Object 类型的实例！对，连数组、函数等都是 Object 类型的实例！**

我所知道的例外有 `Object.create(null)` 返回的空对象。

{% note info %}
- 创建 Object 实例方式有：`new Object()`和**对象字面量表示法**；
- 对象属性名一定是**字符串**或**符号**，如果不是，会自动转换为字符串；
- 访问对象属性的方式：**点表示法**和**方括号表示法**。
{% endnote %}

# Array类型

## 创建数组

数组的每一项都可以保存任何类型的数据，下面用例子展示下几种创建数组的方式：

```js
// new 操作可以省略，即 Array() 和 new Array()效果一样
var a = new Array()                     // a = []

var b = Array('red')                    // b = ['red']

// 只传递一个数值，会创建一个有length属性的数组，但内部空空如也，无法进行迭代等操作
var c = Array(3)                        // c = [empty × 3]

var d = Array(3, 4)                     // d = [3, 4]

var e = ['red', 'blue', ]               // 不要这样！会创建一个包含2或3项的数组

var f = Array.apply(null, { length: 3 })  // f = [undefined, undefined, undefined]

var g = Array.from(Array(3).keys())       // g = [0, 1, 2]

var h = Array.from(Array(3), () => 1)     // h = [1, 1, 1]

var i = [...Array(3).keys()]              // i = [0, 1, 2]
```

{% note warning %}
- **通过数组的length可以清空或截断数组.**
- **数组会自动将字符串索引转换为对应数值索引，即`array['0'] === array[0]`。**
{% endnote %}

## 检测数组

ES5 提出了 `Array.isArray()` 方法来检测数组，不过并不是所有版本的浏览器都兼容这个方法。兼容各种情况的方法如下所示：

```js
function isArray (value) {
    return Object.prototype.toString.call(value) == '[object Array]'
}
```

## 转换方法

`join()`接收一个字符串作为参数，然后以该字符串为分隔符将数组（或类数组对象）的所有元素连接成一个字符串并返回这个字符串。如果数组中只有一个元素，那么将返回该元素的字符串形式而不使用分隔符。如果没有给`join()`传递参数，则默认使用`,`作为分隔符

```js
// "Fire,Air,Water"
console.log(['Fire', 'Air', 'Water'].join())

// "FireAirWater"
console.log(['Fire', 'Air', 'Water'].join(''))

// "Fire-Air-Water"
console.log(['Fire', 'Air', 'Water'].join('-'))

// "[object Object],[object Object]"
console.log([{ a: 1 }, { b: 2 }].join())
```

## 栈方法

栈是一种后进先出的数据结构，ECMAScript 为数组提供了 `push()` 和 `pop()` 方法来实现类似栈的行为。

{% note info %}
- `push()` 方法接收任意数量的参数，把他们逐个添加到数组末尾，然后返回修改后数组的长度；
- `pop()` 方法从数组末尾移除最后一项，将数组长度减 1，然后返回移除的项。
{% endnote %}

## 队列方法

队列的访问规则是先见先出，可以通过 `unshift()` 和 `shift()` 配合实现。

{% note info %}
- `unshift()` 方法在数组前端添加任意个项, 然后返回新数组的长度。
- `shift()` 方法移除数组中的第一项，将数组长度减 1，然后返回移除的项；
{% endnote %}

## 重排序方法

`reverse()` 方法会反转数组项的顺序。`sort()` 方法默认按升序排列数组项，并且可以接收一个比较函数作为参数。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个参数之后则返回一个正数。

```js
var a = [1, 3, 2, 6]

a.reverse()
console.log(a)    // [6, 2, 3, 1]

a.sort()
console.log(a)    // [1, 2, 3, 6]

function compare (value1, value2) {
  if (value1 > value2) {
    return -1
  } else if (value1 < value2) {
    return 1
  } else {
    return 0
  }
}

var values = [0, 1, 5, 10, 15]
values.sort(compare)
console.log(values)    // [15, 10, 5, 1, 0]
```

## 操作方法

**`concat()` 和 `slice()`都不会影响原始数组,但是 `splice()` 会！**

### concat

`concat()` 方法会创建当前数组的一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。
如果传递的参数是一个或多个数组，则会将这些数组中的每一项都添加到结果数组中。如果传递的值不是数组，这些值就会被简单的添加到结果数组的末尾。

```js
var colors = ['red', 'green', 'blue']
var colors2 = colors.concat('yellow', ['black', 'brown'])

console.log(colors)     // ['red', 'green', 'blue']
console.log(colors2)    // ['red', 'green', 'blue', 'yellow', 'black', 'brown']
```

### slice

`slice(start, end)` 方法返回一个由原数组索引从 `start` 到 `end-1` 的项组成的新数组。

{% note info %}
- 如果省略 `start`，则默认从 0 开始；
- 如果省略 `end` 或 `end` 大于数组长度，则 `slice` 方法会一直提取到数组末尾（包括最后一个元素）
- 如果 `start` 或 `end` 中有负数，则用数组长度加上该数来确定相应的索引。
{% endnote %}

```js
var colors = [1, 2, 3, 4]
var colors2 = colors.slice(1, 3)           // colors2 = [2, 3]
var colors3 = colors.slice(undefined, 3)   // colors3 = [1, 2, 3]
var colors4 = colors.slice(1)              // colors4 = [2, 3, 4]
var colors5 = colors.slice(1, 100)         // colors5 = [2, 3, 4]
var colors6 = colors.slice(1, 2)           // colors6 = [2]
var colors7 = colors.slice(-3, -2)         // colors7 = [2]
```

### splice

`splice(start, deleteNum, replaceItem, replaceItem, replaceItem, ...)` 方法返回一个数组，该数组包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组）。它是最灵活的一个数组操作方法，它可以实现对原始数组的删除、插入和替换操作。

{% note info %}
- `start`：操作起始位置的索引；
- `deleteNum`：要删除的项数；
- `replaceItem`：要插入的项,一个或多个。
{% endnote %}

```js
var colors = ['red', 'green', 'blue']
var removed = colors.splice(0, 1)
console.log(colors)        // ['green', 'blue']
console.log(removed)       // ['red']

removed = colors.splice(1, 0, 'yellow', 'orange')
console.log(colors)        // ['green', 'yellow', 'orange', 'blue']
console.log(removed)       // []

removed = colors.splice(1, 1, 'red', 'purple')
console.log(colors)        // ['green', 'red', 'purple', 'orange', 'blue']
console.log(removed)       // ['yellow']
```

## 位置方法

`indexOf(searchValue[, fromIndex])` 和 `lastIndexOf(searchValue[, fromIndex])` 都是找到就返回索引，找不到就返回 -1。

## 迭代方法

### every

{% note info %}
`arr.every(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，如果所有 `callback()` 的返回值都等价于 `true`，返回 `true`。否则返回 `false`。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
var arr = [0, 1, 2];

// false
console.log(arr.every((item) => item));
```

### some

{% note info %}
`arr.some(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，只要有一个 `callback()` 的返回值都等价于 `true`，返回 `true`。否则返回 `false`。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
var arr = [0, 1, 2];

// true
console.log(arr.some((item) => item));
```

### filter

{% note info %}
`arr.every(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，返回 `callback()` 返回值等价于 `true` 对应的元素所组成的新数组。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
var arr = [0, 1, 2];
var newArr = arr.filter((item) => item);

// [1, 2]
console.log(newArr);
```

### forEach

{% note info %}
- `arr.forEach(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，该方法没有返回值。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
var arr = [0, {}, {}];
arr.forEach((item, index) => {
  item.index = index;
});

// [0, { index: 1 }, { index: 2 }]
console.log(arr);
```

### map

{% note info %}
- `arr.map(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，返回所有 `callback()` 返回值组成的新数组。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
var arr = [0, {}, {}];
var newArr = arr.map((item) => {
  return !!item;
});

// [false, true, true]
console.log(newArr);
```

### reduce

{% note info %}
- `arr.reduce(callback[, initialValue])`：对每个数组元素调用 `callback()` 方法，返回最后一个 `callback()` 的返回值。
  - `callback(accumulator, currentValue[, index[, array]])`：该回调函数会在数组中进行迭代，它的返回值将传递给下一次迭代的 `accumulator` 参数，它的参数介绍如下：
    - `accumulator`：上一次迭代的返回值；
    - `currentValue`：迭代的当前值；
    - `index`：当前索引；
    - `array`：数组对象。
  - `initialValue`：作为 `callback()` 进行第一次迭代时 `accumulator` 的值。如果没有传入这个参数，那么 `callback()` 在第一次迭代时会将数组的第一项作为 `accumulator` 的值。
{% endnote %}

```js
var values = [1, 2, 3]
var sum1 = values.reduce((acc, cur, index, array) => {
    console.log('acc: ' + acc, 'cur: ' + cur, 'index: ' + index)
    return acc + cur
})
// acc: 1 cur: 2 index: 1
// acc: 3 cur: 3 index: 2

var sum2 = values.reduce((acc, cur, index, array) => {
    console.log('acc: ' + acc, 'cur: ' + cur, 'index: ' + index)
    return acc + cur
}, 10)
// acc: 10 cur: 1 index: 0
// acc: 11 cur: 2 index: 1
// acc: 13 cur: 3 index: 2

console.log(sum1)       // 6
console.log(sum2)       // 16
```

### reduceRight

{% note info %}
- `arr.reduceRight(callback[, initialValue])`：按照倒序对每个数组元素调用 `callback()` 方法，返回最后一个 `callback()` 的返回值。
  - `callback(accumulator, currentValue[, index[, array]])`：该回调函数会在数组中进行迭代，它的返回值将传递给下一次迭代的 `accumulator` 参数，它的参数介绍如下：
    - `accumulator`：上一次迭代的返回值；
    - `currentValue`：迭代的当前值；
    - `index`：当前索引；
    - `array`：数组对象。
  - `initialValue`：作为 `callback()` 进行第一次迭代时 `accumulator` 的值。如果没有传入这个参数，那么 `callback()` 在第一次迭代时会将数组的最后一项作为 `accumulator` 的值。
{% endnote %}

```js
var values = [1, 2, 3]
var sum1 = values.reduceRight((acc, cur, index, array) => {
    console.log('acc: ' + acc, 'cur: ' + cur, 'index: ' + index)
    return acc + cur
})
// acc: 3 cur: 2 index: 1
// acc: 5 cur: 1 index: 0

var sum2 = values.reduceRight((acc, cur, index, array) => {
    console.log('acc: ' + acc, 'cur: ' + cur, 'index: ' + index)
    return acc + cur
}, 10)
// acc: 10 cur: 3 index: 2
// acc: 13 cur: 2 index: 1
// acc: 15 cur: 1 index: 0

console.log(sum1)       // 6
console.log(sum2)       // 16
```

# Date类型

通过 `var now = new Date()` 可以创建一个日期对象，在调用 `Data` 构造函数而不传参数时，新创建的对象自动获得当前日期和时间（**当前日期和时间是浏览器从本机操作系统获取的时间，所以不一定准确！**）。如果想根据特定的日期和时间创建日期对象，必须传入表示该日期的毫秒数（即从 UTC 时间 2017 年 1 月 1 日午夜起至该日期经过的毫秒数）。为了简化这一计算过程，ES提出了两个方法：`Date.parse()`和`Date.UTC()`。

{% note info %}
- `Date.parse()`：接收一个表示日期的字符串参数，然后尝试根据这个字符串返回相应日期的毫秒数。但是它支持哪种日期格式并不一定；
- `Date.UTC()`：同样返回表示日期的毫秒数，但接收参数分别为年份、**基于0的月份**、月中的哪一天、小时、分钟、秒以及毫秒。这些参数中只有年和月是必需的。
{% endnote %}

其实，在使用 `Date()` 构造函数创建日期对象时，后台会根据传入的参数自动调用 `Date.parse()` 或 `Date.UTC()` 方法，所以根本不需要我们去主动的调用它们。
ES5添加了 `Date.now()` 方法，返回表示调用这个方法时的日期和时间的毫秒数。在不兼容它的情况下可以使用 `+ new Date()` 来达到相同的效果。

```js
var now = new Date()
now                    // 2020-10-17T04:56:29.512Z
now.getFullYear()      // 2020
now.getMonth()         // 0-11
now.getDate()          // 1-31
now.getDay()           // 0-6，0代表周日，6代表周六
now.getHours()         // 0-23
now.getMinutes()       // 0-59
now.getSeconds()       // 0-59
now.getMilliseconds()  // 0-999
```

# RegExp类型

正则表达式可以通过 `var expression = /pattern/flags` 的字面量形式来创建。其中的 `pattern` 代表任何简单或复杂的正则表达式，`flag` 代表一个或多个标志。

{% note info %}
- `g`：表示全局模式，即模式将被应用于所有字符串，而非在发现第一个匹配项时立即停止；
- `i`：表示不区分大小写模式；
- `m`：表示多行模式。
{% endnote %}

同样也支持使用 `RegExp` 构造函数来创建正则表达式，它接收两个参数：一个是要匹配的字符串模式，另一个是可选的标志字符串。

```js
// pattern1 和 pattern2 的作用是完全一样的
var pattern1 = /[bc]at/i
var pattern2 = new RegExp('[bc]at', 'i')
```

{% note warning %}
由于`RegExp` 构造函数的模式参数都是字符串，所以在某些情况下要对字符进行双重转义。所有元字符都必须双重转义，那些已经转义过的字符也是如此。
{% endnote %}

## exec

{% note info %}
`exec()` 方法在一个指定字符串中 **执行一次匹配搜索**。如果匹配失败，返回 `null`。如果匹配成功，返回一个数组。这个数组有以下特点：
- 该方法一次只会返回一个匹配项（第一个匹配项），这个匹配项被保存在数组的第一项。
- 捕获组从数组第二项开始保存。如果没有捕获组，则返回的数组只包含一项。
- 具有 `input` 属性，表示应用正则表达式的字符串。
- 具有 `index` 属性，表示匹配项在字符串中的索引。
- 具有 `groups` 属性，表示一个捕获组或 `undefined`（如果没有定义命名捕获组）
{% endnote %}

{% note warning %}
**对于 `exec()` 方法而言，即使在模式中设置了 `g` 标志，它每次也只会返回一个匹配项。在不设置 `g` 标志的情况下，在同一个字符串上多次调用 `exec()` 将始终返回第一个匹配项的信息。而在设置了 `g` 标志的情况下，每次调用 `exec()` 都会在字符串中继续查找新匹配项**
{% endnote %}

```js
var text = 'mom and dad and baby'
var pattern = /mom (and dad (and baby)?)?/

// [ 'mom and dad and baby', 'and dad and baby', 'and baby', index: 0, input: 'mom and dad and baby', groups: undefined ]
console.log(pattern.exec(text))



var str = 'cat, bat, sat, fat'
var p1 = /.at/

// [ 'cat', index: 0, input: 'cat, bat, sat, fat', groups: undefined ]
console.log(p1.exec(str))

// [ 'cat', index: 0, input: 'cat, bat, sat, fat', groups: undefined ]
console.log(p1.exec(str))

var p2 = /.at/g

// [ 'cat', index: 0, input: 'cat, bat, sat, fat', groups: undefined ]
console.log(p2.exec(str))

// [ 'bat', index: 5, input: 'cat, bat, sat, fat', groups: undefined ]
console.log(p2.exec(str))
```

## test

`test()`方法接收一个字符串参数，在模式与该参数匹配的情况下返回 `true`，否则返回 `false`。

# Function类型

创建函数有三种方式：函数声明语法、函数表达式和使用 `Function` 构造函数。

## 函数声明与函数表达式

它们的区别在于使用函数声明创建的函数会经历一个叫做**函数声明提升**的过程，而函数表达式创建的函数则不会。

```js
console.log(sum1(10, 10))       // 20
console.log(sum2(10, 10))       // Uncaught ReferenceError: sum2 is not defined
function sum1 (num1, num2) {
  return num1 + num2
}
sum2 = function (num1, num2) {
  return num1 + num2
}
```

## 作为值的函数

ES中函数名本身就是变量，所以可以将一个函数作为参数传递给另一个函数，也可以将一个函数作为另一个函数的返回值。

**函数名后不加圆括号表示的是函数的引用，加上圆括号表示执行函数！**

```js
var a = function (num) {
  return num + 1
}
console.log(a(1))       // 2
console.log(a)          // f (num) { return num + 1 }
```

## 函数属性和方法

ES 中的函数是对象，因此函数也有属性和方法。

### 属性

{% note info %}
- `length`：表示函数希望接收的命名参数的个数；
- `prototype`：后续会详细介绍，此处知道该属性是不可枚举的，使用 `for-in` 无法发现即可。
{% endnote %}

### 方法

每个函数都包含两个非继承而来的方法：`apply()` 和 `call()` 方法。这两个方法的用途都是在特定的作用于中调用函数，实际上等于设置函数体内的 `this` 值。关于 `this` 对象，我们暂时记住，在全局函数中，`this` 等于 `window` 对象，而当函数被作为某个对象方法调用时，`this` 等于那个对象。

{% note info %}
- `apply()`：接收两个参数：一个是用来指定 `this` 值，另一个是参数数组。其中第二个参数可以是 `Array` 的实例，也可以是 `arguments` 对象。
- `call()`：与 `apply()` 方法的区别在于接收参数的方式，它的第一个参数同样是用来指定 `this` 值，但是后续传入的所有参数均会直接传递给函数。换句话说，在使用 `call()` 方法时，传给函数的参数必须逐个列举出来。
- `bind()`：该方法会返回一个新创建的函数实例，其 `this` 值会被绑定到传递给 `bind()` 函数的值。该方法常常被用于函数柯里化。
{% endnote %}

**如果用来指定 `this` 值的参数是基本数据类型，那么这个基本类型值会被转换为对应的基本包装类型（new String()、 new Boolean()等）。**

# 基本包装类型

ES提供了 3 个**特殊的引用类型**：`Boolean`、`Number` 和 `String`。这些类型与引用类型相似，但同时也具有与各自的基本类型相应的特殊行为，我们称它们为基本包装类型。

**实际上，每当读取一个基本类型值的时候，后台会自动创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。**

**引用类型与基本包装类型的主要区别就是对象的生存期。**使用 `new` 操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中。而**自动创建**的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。

```js
var s1 = 'some text'        // 创建了一个基本类型值并保存在s1中
s1.color = 'red'            // 调用对应的基本包装类型
console.log(s1.color)       // undefined，因为基本包装类型的生存期已过
console.log(typeof s1)      // string

var s2 = new String('some text')    // 显示的创建了String的实例对象并保存在s2中，实际上就是创建了一个引用类型值
s2.color = 'red'
console.log(s2)             // String {"some text", color: "red"}
console.log(typeof s2)      // object
```

通过上面例子认识到通过 `new` 显示创建的就是一个引用类型值，它已经不能称之为基本包装类型了。换句话说，基本包装类型都是后台自己创建的，不存在显示创建的情况。在第六章会介绍，通过 `new` 显示创建的就是一个引用类型的实例对象。**记住最好别用new将基本包装类型给实例化就行。**

**要注意的是，使用 `new` 调用基本包装类型的构造函数，与直接调用同名的转型函数是不一样的。**

```js
var value = '25'
var number = Number(value)      // 转型函数
console.log(typeof number)      // number

var obj = new Number(value)     // 构造函数
console.log(typeof obj)         // object
```

## Boolean类型

没啥说的。

## Number类型

{% note info %}
- `toFixed()`：按照指定的小数位返回数值的字符串表示，银行家喜欢用这个方法，这个方法不是真正意义的四舍五入；
- `toExponential()`：返回以指数表示法表示的数值的字符串形式；
- `toPrecision()`：返回固定大小格式，也可能返回指数格式。
{% endnote %}

## String类型

### 字符方法

`charAt()` 和 `charCodeAt()`方法都接收一个参数，即基于0的字符位置。`charAt()` 方法以单字符字符串的形式返回给定位置的那个字符，而 `charCodeAt()` 方法返回的是字符编码。

```js
var string = 'hello world'
console.log(string.charAt(1))       // e
console.log(string.charCodeAt(1))   // 101
```

ES5 还定义了访问个别字符的方法，可以像数组那样使用索引来访问某个位置的字符。

```js
var string = 'hello'
console.log(string[1])      // e
```

### 字符串操作方法

**下面所列的四个方法，返回值均是被操作字符串的子字符串。并且这些方法都不会对原始字符串产生任何影响。**

{% note info %}
- `concat()`：类比于数组的`concat()`方法；
- `slice()`：类比于数组的`slice()`方法；
- `substr()`：第一个参数用于指定子字符串的开始位置，第二个参数用于指定返回的字符个数;
- `substring()`：类比于数组的`slice()`方法。
{% endnote %}

当传递负值作为参数时，`slice()` 方法会将传入的负值与字符串的长度相加；`substr()` 方法将负的第一个参数加上字符串的长度，而将负的第二个参数转换为0；`substring()` 方法会把所有负值参数都转换为0。

**另外需要注意的是，`substring()` 方法会将参数中较小的数作为开始位置，将较大的数作为结束位置。即 `string.substring(3, 0)` 等价于 `string.substring(0, 3)`**。

```js
var string = 'hello world'

console.log(string.slice(-3))           // rld
console.log(string.substring(-3))       // hello world
console.log(string.substr(-3))          // rld

console.log(string.slice(3, -4))        // lo w
console.log(string.substring(3, -4))    // hel
console.log(string.substr(3, -4))       // ''
```

### 字符串位置方法

`indexOf(searchValue[, fromIndex])` 和 `lastIndexOf(searchValue[, fromIndex])` 都是找到就返回索引，找不到就返回 -1。

### trim

ES5定义了这个方法，会创建一个字符串的副本，删除前置及后缀的所有空格，然后返回结果。

### 字符串大小写转换方法

`toLowerCase()`、`toLocaleLowerCase()`、`toUpperCase()` 和 `toLocaleUpperCase()` 方法。

### 字符串的模式匹配方法

#### match

{% note info %}
- `match()`：如果没有匹配成功就返回 `null`。如果匹配成功了，根据下面情况返回不同的内容。
  - 如果正则表达式使用了 `g` 标志，则返回所有匹配项组成的数组，该数组无特殊属性。
  - 如果正则表达式没有使用 `g` 标志，则仅返回第一个匹配项与捕获组组成的数组，该数组还含有3个特殊属性。
    - `index`：匹配项在字符串中的索引。
    - `input`：输入的字符串。
    - `group`：一个捕获组或 `undefined`（如果没有定义命名捕获组）。
{% endnote %}

```js
var text = 'mom and dad and baby'

// [ 'mom and dad and baby' ]
console.log(text.match(/mom (and dad (and baby)?)?/g))

// [ 'mom and dad and baby', 'and dad and baby', 'and baby', index: 0, input: 'mom and dad and baby', groups: undefined ]
console.log(text.match(/mom (and dad (and baby)?)?/))
```

#### search

{% note info %}
`search()` 方法接收正则表达式作为参数，返回字符串中第一个匹配项的索引，如果没有找到匹配项则返回-1。
{% endnote %}

#### replace

{% note info %}
`replace()`：该方法接收两个参数，第一个参数可以是正则表达式或一个字符串，第二个参数可以是一个字符串或一个函数。**如果第一个参数是字符串，那么只会替换第一个子字符串，要想替换所有子字符串，唯一的办法就是提供正则表达式作为第一参数，并且要指定全局标志**。第二个参数与正则表达式的捕获组有关。
- [如果第二个参数是字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_string_as_a_parameter)
- [如果第二个参数是函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_function_as_a_parameter)
{% endnote %}

```js
var text = 'cat, bat, sat, fat'
var result = text.replace('at', 'ond')
console.log(result)     // cond, bat, sat, fat

result = text.replace(/at/g, 'ond')
console.log(result)     // cond, bond, sond, fond

result = text.replace(/(.at)/g, 'word ($1)')
console.log(result)     // word (cat), word (bat), word (sat), word (fat)
```

#### split

{% note info %}
`split()` 方法可以基于指定的分隔符将一个字符串分割成多个子字符串并将结果存放在一个数组中。分隔符可以是字符串，也可以是一个正则表达式。该方法有一个可选的第二参数，用于指定数组的大小。**如果将空字符串作为分隔符传递给 `split()`，那么字符串会基于每一个 UTF-16 codeunit 进行划分。如果传递给 `split()` 的正则表达式中包含捕获组，那么捕获组会被保留下来**
{% endnote %}

```js
var colorText = 'red,blue,green,yellow'
console.log(colorText.split(','))           // [ 'red', 'blue', 'green', 'yellow' ]
console.log(colorText.split(',', 2))        // [ 'red', 'blue' ]
console.log(colorText.split(/\,/))          // [ 'red', 'blue', 'green', 'yellow' ]
console.log(colorText.split(/(\,)/))        // [ 'red', ',', 'blue', ',', 'green', ',', 'yellow' ]
console.log(colorText.split(/[^\,]+/))      // [ '', ',', ',', ',', '' ]
console.log(colorText.split(/([^\,]+)/))    // [ '', 'red', ',', 'blue', ',', 'green', ',', 'yellow', '' ]
```

### localeCompare

[MDN 上的介绍](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare)


### fromCharCode

该方法接收一个或多个字符编码，然后将它们转换成字符串。

# 单体内置对象

内置对象的定义是：“由ECMAScript实现提供的、不依赖于宿主环境的对象，这些对象在ECMAScript程序执行之前就已经存在了。”意思就是说，开发人员不必显示地实例化内置对象，因为它们已经实例化了。前面介绍的 `Object`、`Array`、`String` 等都是内置对象，除此之外还有两个单体内置对象：`Global` 和 `Math`。

## Global对象

`Global` (全局)对象可以说是 ECMAScript 中最特别的一个对象了，因为不管你从什么角度上看，这个对象都是不存在的。ES中的`Global` 对象在某种意义上是作为一个终极的“兜底儿的对象”来定义的。换句话说，不属于任何其他对象的属性和方法，最终都是它的属性和方法。事实上，没有全局变量或全局函数；所有在全局作用域中定义的属性和函数，都是 `Global` 对象的属性。

ES虽然没有指出如何直接访问 `Global` 对象,但 Web 浏览器都是将这个全局对象作为 `window` 对象的一部分加以实现的。

```js
// 取得Global对象的方法
var global = function () {
    return this
}
```

### URI编码方法

Gloabl对象的 `encodeURI()` 和 `encodeURIComponent()` 方法可以对URI进行编码，以便发送给浏览器

它们的主要区别在于，`encodeURI()` 不会对本身属于URI的特殊字符进行编码，例如冒号、正斜杠、问号和井字号。而 `encodeURIComponent()` 则会对它发现的任何非标准字符串进行编码

与它们相对应的方法是 `decodeURI()` 和 `decodeURIComponent()`

```js
var uri = 'http://www.wrox.com/illegal value.html#start'
console.log(encodeURI(uri))
// http://www.wrox.com/illegal%20value.html#start

console.log(encodeURIComponent(uri))
// http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.html%23start
```

## Math对象

### 属性和方法

{% note info %}
属性：
- `Math.E`：自然对数的底数，即常量 e 的值。
- `Math.LN10`：10 的自然对数。
- `Math.LN2`：2 的自然对数。
- `Math.LOG2E`：以 2 为底 e 的对数。
- `Math.LOG10E`：以 10 为底 e 的对数。
- `Math.PI`：π 的值。
- `Math.SQRT1_2`：1/2 的平方根（即 2 的平方根的倒数）。
- `Math.SQRT2`：2 的平方根。

---
方法：
- `Math.abs(x)`：返回`x`的绝对值。
- `Math.pow(x, y)`：返回 $x^y$。
{% endnote %}

### min && max

```js
var max = Math.max(3, 54, 32, 16)
var min = Math.min(3, 54, 32, 16)
console.log(max, min)       // 54 3 

var values = [1, 2, 3, 4, 5, 6, 7, 8]
console.log(Math.max.apply(Math, values))   // 8
```

### 舍入方法

{% note info %}
- `Math.ceil()`：执行向上（大）舍入，即它总是将数值向上舍入为最接近的整数；
- `Math.floor()`：执行向下（小）舍入，即它总是将数值向下舍入为最接近的整数；
- `Math.round()`：执行标准舍入，即它总是将数值四舍五入为最接近的整数。

---
`Math.round()` 在数值是负数且小数部分恰好是 0.5 时会出现例外，如 `Math.round(-1.5)` 的值为 -1.
{% endnote %}

### random

`Math.random()` 方法返回介于 0 和 1 之间的一个随机数，包括 0，但不包括 1。

```js
// 得到一个随机数，包括 min，不包括 max
function getRandomArbitrary (min, max) {
  return Math.random() * (max - min) + min 
}

// 得到一个随机整数，包括 min，不包括 max
function getRandomInt (min, max) {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min)) + min
}

// 得到一个随机整数，包括 min 和 max
function getRandomIntInclusive (min, max) {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min + 1)) + min
}
```