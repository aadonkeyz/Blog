---
title: Array
abbrlink: 6e01c1fe
date: 2020-10-24 23:26:22
categories:
  - JavaScript
---

# 创建数组

## ES5 及之前
数组的每一项都可以保存任何类型的数据，可以通过 `Array` 构造器和数组字面量语法两种方式创建数组，如果通过 `Array` 构造器创建数组，`new` 操作符可以省略。

```js
var a = new Array('red', 1);
var b = Array('red', 1);
var c = ['red', 1];

// [ 'red', 1 ]
console.log(a)

// [ 'red', 1 ]
console.log(b)

// [ 'red', 1 ]
console.log(c)
```
{% note warning %}
- **通过数组的 length 可以清空或截断数组。**
- **数组会自动将字符串索引转换为对应数值索引，即`array['0'] === array[0]`。**
{% endnote %}

## Array 构造器的怪异点

{% note warning %}
**通过 `Array` 构造器创建数组时，如果只传递一个数值，会创建一个有 `length` 属性的数组，但内部空空如也，无法进行如 `forEach` 等迭代操作。**
{% endnote %}

```js
var normal = Array('3');

// [ '3' ]
console.log(normal);

var empty = Array(3);

// [ <3 empty items> ]
console.log(empty);

// 3
console.log(empty.length);

for (let i = 0; i < empty.length; i++) {
  // 0 undefined
  // 1 undefined
  // 2 undefined
  console.log(i, empty[i]);
}

empty.forEach(item => {
  console.log(item);
})
// 等价于
var a = [];
a.forEach((item, index) => {
  console.log(item);
})
```

## 用 Array.apply 创建数组

```js
var a = Array.apply(null, { length: 3 });

// [ undefined, undefined, undefined ]
console.log(a);

a.forEach((item, index) => {
  // 0 undefined
  // 1 undefined
  // 2 undefined
  console.log(index, item);
})
```

## 通过解构&迭代器创建数组

```js
var a = [...Array(3).entries()];
var b = [...Array(3).keys()];
var c = [...Array(3).values()];

// [ [ 0, undefined ], [ 1, undefined ], [ 2, undefined ] ]
console.log(a);

// [ 0, 1, 2 ]
console.log(b);

// [ undefined, undefined, undefined ]
console.log(c);
```

## Array.of

ES6 引入了 `Array.of()` 方法来解决 `Array` 构造器创建数组的怪异点。该方法的作用非常类似 `Array` 构造器，但在使用单个数值参数的时候并不会导致特殊结果。`Array.of()` 方法总会创建一个包含所有传入参数的数组，而不管参数的数量与类型。下面例子演示了 `Array.of()` 的用法：

```js
var a = Array.of(1, 2)

// 2
console.log(a.length)       

// 1
console.log(a[0])           

// 2
console.log(a[1])           

var b = Array.of(2)

// 1
console.log(b.length)       

// 2
console.log(b[0])           

var c = Array.of('2')

// 1
console.log(c.length)       

// '2'
console.log(c[0])           
```

{% note warning %}
`Array.of()` 方法并没有使用 `Symbol.species` 属性来决定返回值的类型，而是使用了当前的构造器（即 `of()` 方法内部的 `this`）来做决定。例如，`MyArray.of()` 返回 `MyArray` 类型的实例，`Array.of()` 返回 `Array` 类型的实例。
{% endnote %}

## Array.from

将**可迭代对象**或者**类数组对象**作为第一个参数传入，`Array.from()` 就能返回一个数组。这里有个简单的例子：

```js
function doSomething() {
  var args = Array.from(arguments)

  // 使用 args
}
```

此处调用 `Array.from()` 方法，使用 `arguments` 对象创建了一个新数组 `args`，它是一个数组实例，并且包含了 `arguments` 对象的所有项，同时还保持了项的顺序。

{% note warning %}
与 `Array.of()` 一样，`Array.from()` 方法同样使用当前的构造器（即 `from()` 方法内部的 `this`）来决定要返回什么类型的数组。例如，`MyArray.from()` 返回 `MyArray` 类型的实例，`Array.from()` 返回 `Array` 类型的实例。
{% endnote %}

如果你想实行进一步的数组转换，你可以向 `Array.from()` 方法传递一个映射用的函数作为第二个参数。此函数会将类数组对象的每一个值转换为目标形式，并将其存储在目标数组的对应位置上。例如：

```js
function translate () {
  return Array.from(arguments, value => value + 1)
}

const numbers = translate(1, 2, 3)

// [ 2, 3, 4 ]
console.log(numbers)
```

如果映射函数需要在对象上工作，你可以手动传递第三个参数给 `Array.from()` 方法，从而指定映射函数内部的 `this` 值。

```js
let helper = {
  diff: 1,
  add (value) {
    return value + this.diff
  }
}

function translate () {
  return Array.from(arguments, helper.add, helper)
}

let numbers = translate(1, 2, 3)

// [ 2, 3, 4 ]
console.log(numbers)
```

{% note warning %}
**只要一个对象有 `length` 属性，`Array.from()` 就会可以通过它得到对应长度的数组**
{% endnote %}

```js
let a = {
  length: 3
}

// [ undefined, undefined, undefined ]
console.log(Array.from(a))
```

# 检测数组

ES5 提出了 `Array.isArray()` 方法来检测数组，不过并不是所有版本的浏览器都兼容这个方法。兼容各种情况的方法如下所示：

```js
function isArray (value) {
  return Object.prototype.toString.call(value) == '[object Array]'
}
```

# 转换方法

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

# 栈方法

栈是一种后进先出的数据结构，ECMAScript 为数组提供了 `push()` 和 `pop()` 方法来实现类似栈的行为。

{% note info %}
- `push()` 方法接收任意数量的参数，把他们逐个添加到数组末尾，然后返回修改后数组的长度.
- `pop()` 方法从数组末尾移除最后一项，将数组长度减 1，然后返回移除的项。
{% endnote %}

# 队列方法

队列的访问规则是先见先出，可以通过 `unshift()` 和 `shift()` 配合实现。

{% note info %}
- `unshift()` 方法在数组前端添加任意个项, 然后返回新数组的长度。
- `shift()` 方法移除数组中的第一项，将数组长度减 1，然后返回移除的项.
{% endnote %}

# 排序方法

`reverse()` 方法会反转数组项的顺序。`sort()` 方法默认按升序排列数组项，并且可以接收一个比较函数作为参数。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个参数之后则返回一个正数。

```js
var a = [1, 3, 2, 6]

a.reverse()

// [6, 2, 3, 1]
console.log(a)

a.sort()

// [1, 2, 3, 6]
console.log(a)

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

// [15, 10, 5, 1, 0]
console.log(values)    
```

# 操作方法

## concat

`concat()` 方法会创建当前数组的一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。
如果传递的参数是一个或多个数组，则会将这些数组中的每一项都添加到结果数组中。如果传递的值不是数组，这些值就会被简单的添加到结果数组的末尾。

```js
var colors = ['red', 'green', 'blue']
var colors2 = colors.concat('yellow', ['black', 'brown'])

// ['red', 'green', 'blue']
console.log(colors)  
// ['red', 'green', 'blue', 'yellow', 'black', 'brown']   
console.log(colors2)    
```

## slice

`slice(start, end)` 方法返回一个由原数组索引从 `start` 到 `end-1` 的项组成的新数组。

{% note info %}
- 如果省略 `start`，则默认从 0 开始。
- 如果省略 `end` 或 `end` 大于数组长度，则 `slice` 方法会一直提取到数组末尾（包括最后一个元素）
- 如果 `start` 或 `end` 中有负数，则用数组长度加上该数来确定相应的索引。
{% endnote %}

```js
var colors = [1, 2, 3, 4]

// colors2 = [2, 3]
var colors2 = colors.slice(1, 3)       

// colors3 = [1, 2, 3]
var colors3 = colors.slice(undefined, 3)   

// colors4 = [2, 3, 4]
var colors4 = colors.slice(1)  

// colors5 = [2, 3, 4]    
var colors5 = colors.slice(1, 100)

// colors6 = [2]        
var colors6 = colors.slice(1, 2) 

// colors7 = [2]         
var colors7 = colors.slice(-3, -2)         
```

## splice

`splice(start, deleteNum, replaceItem, replaceItem, replaceItem, ...)` 方法返回一个数组，该数组包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组）。它是最灵活的一个数组操作方法，它可以实现对原始数组的删除、插入和替换操作。

{% note info %}
- `start`：操作起始位置的索引。
- `deleteNum`：要删除的项数。
- `replaceItem`：要插入的项,一个或多个。
{% endnote %}

```js
var colors = ['red', 'green', 'blue']
var removed = colors.splice(0, 1)

// ['green', 'blue']
console.log(colors)        
// ['red']
console.log(removed)       

removed = colors.splice(1, 0, 'yellow', 'orange')

// ['green', 'yellow', 'orange', 'blue']
console.log(colors)      
// []  
console.log(removed)       

removed = colors.splice(1, 1, 'red', 'purple')

// ['green', 'red', 'purple', 'orange', 'blue']
console.log(colors)    
// ['yellow']    
console.log(removed)       
```

## fill

{% note info %}
`fill(value, start, end + 1)` 方法使用参数 `value` 去替换数组索引从 `start` 至 `end` 的项。如果提供的起始位置（`start`）或结束位置（`end + 1`）为负数，则它们会被加上数组的长度来算出最终的位置。请观察下面的例子：
{% endnote %}

```js
let numbers = [1, 2, 3, 4]

numbers.fill(1)

// [ 1, 1, 1, 1 ]
console.log(numbers)

numbers.fill(2, 1)

// [ 1, 2, 2, 2 ]
console.log(numbers)

numbers.fill(3, 1, 3)

// [ 1, 3, 3, 2 ]
console.log(numbers)
```

{% note warning %}
**需要注意的是，`fill()` 方法是使用浅复制来完成的。**
{% endnote %}

```js
let a = [1, 2];
a.fill([]);

// [ [], [] ]
console.log(a);

a[1].push(33);

// [ [ 33 ], [ 33 ] ]
console.log(a);

// true
console.log(a[0] === a[1]);
```

## copyWith

`copyWithin()` 方法与 `fill()` 类似，可以一次性修改数组的多个元素。不过，与 `fill()` 使用单个值来修改数组不同，`copyWithin()` 方法允许你在数组内部复制自身元素。为此你需要传递两个参数给 `copyWithin()` 方法：从什么位置开始进行修改，以及被用来复制的数据的起始位置索引。

```js
let numbers = [1, 2, 3, 4]

// 从索引 2 的位置开始粘贴
// 从索引 0 的位置开始复制数据
numbers.copyWithin(2, 0)

// [ 1, 2, 1, 2 ]
console.log(numbers)     
```

默认情况下，`copyWithin()` 方法总是会一直复制到数组末尾，不过你还可以提供一个可选参数来限制到底有多少元素会被修改。这第三个参数指定了复制停止的位置（不包括该位置自身），这里有个范例：

```js
let numbers = [1, 2, 3, 4]

// 从索引 2 的位置开始粘贴
// 从索引 0 的位置开始复制数据
// 在索引 1 的位置停止复制
numbers.copyWithin(2, 0, 1)

// [ 1, 2, 1, 4 ]
console.log(numbers)
```

{% note info %}
类似于 `fill()` 方法，如果你向 `copyWithin()` 方法传递负数参数，数组的长度会自动被加到该参数的值上，以便算出正确的索引位置。
{% endnote %}

# 位置方法

`indexOf(searchValue[, fromIndex])` 和 `lastIndexOf(searchValue[, fromIndex])` 都是找到就返回索引，找不到就返回 -1。

# 迭代方法

## every

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

## some

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

## filter

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

## forEach

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

## map

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

## reduce

{% note info %}
- `arr.reduce(callback[, initialValue])`：对每个数组元素调用 `callback()` 方法，返回最后一个 `callback()` 的返回值。
  - `callback(accumulator, currentValue[, index[, array]])`：该回调函数会在数组中进行迭代，它的返回值将传递给下一次迭代的 `accumulator` 参数，它的参数介绍如下：
    - `accumulator`：上一次迭代的返回值。
    - `currentValue`：迭代的当前值。
    - `index`：当前索引。
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

// 6
console.log(sum1)    
// 16   
console.log(sum2)       
```

## reduceRight

{% note info %}
- `arr.reduceRight(callback[, initialValue])`：按照**倒序**对每个数组元素调用 `callback()` 方法，返回最后一个 `callback()` 的返回值。
  - `callback(accumulator, currentValue[, index[, array]])`：该回调函数会在数组中进行迭代，它的返回值将传递给下一次迭代的 `accumulator` 参数，它的参数介绍如下：
    - `accumulator`：上一次迭代的返回值。
    - `currentValue`：迭代的当前值。
    - `index`：当前索引。
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

// 6
console.log(sum1)       
// 16
console.log(sum2)
```

## find

{% note info %}
- `arr.find(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，如果某一个 `callback()` 返回值为 `true`，则返回其对应的数组元素。如果有多个 `callback()` 都返回 `true`，则只返回第一个。如果所有 `callback()` 都返回 `false`，则返回 `undefined`。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
const a = [1, 2, 3];

// 2
console.log(a.find((item, index) => item === 2 || index === 2));

// undefined
console.log(a.find(item => item === 4));
```

## findIndex

{% note info %}
- `arr.find(callback[, thisArg])`：对每个数组元素调用 `callback()` 方法，如果某一个 `callback()` 返回值为 `true`，则返回其对应的索引。如果有多个 `callback()` 都返回 `true`，则只返回第一个。如果所有 `callback()` 都返回 `false`，则返回 -1`。
  - `callback(item, index, array)`：
    - `item`：元素。
    - `index`：元素对应索引。
    - `array`：`arr` 的引用。
  - `thisArg`：作用域对象。
{% endnote %}

```js
const a = [1, 2, 3];

// 1
console.log(a.findIndex((item, index) => item === 2 || index === 2));

// -1
console.log(a.findIndex(item => item === 4));
```