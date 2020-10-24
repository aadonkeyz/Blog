---
title: Array
abbrlink: 6e01c1fe
date: 2020-10-24 23:26:22
categories:
  - JavaScript
---

# 创建数组

数组的每一项都可以保存任何类型的数据，下面用例子展示下几种创建数组的方式：

```js
/**
 * new 操作可以省略，即 Array() 和 new Array() 效果一样
 */
// a = []
var a = new Array()

// b = ['red']
var b = Array('red')

/**
 * 只传递一个数值，会创建一个有 length 属性的数组，但内部空空如也，无法进行迭代等操作
 */
// c = [empty × 3]
var c = Array(3)

// d = [3, 4]
var d = Array(3, 4)

// 不要这样！会创建一个包含2或3项的数组
var e = ['red', 'blue', ]

// f = [undefined, undefined, undefined]
var f = Array.apply(null, { length: 3 })

// g = [0, 1, 2]
var g = Array.from(Array(3).keys())

// h = [1, 1, 1]
var h = Array.from(Array(3), () => 1)     

// i = [0, 1, 2]
var i = [...Array(3).keys()]              
```

{% note warning %}
- **通过数组的length可以清空或截断数组.**
- **数组会自动将字符串索引转换为对应数值索引，即`array['0'] === array[0]`。**
{% endnote %}

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

**`concat()` 和 `slice()`都不会影响原始数组,但是 `splice()` 会！**

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
- `arr.reduceRight(callback[, initialValue])`：按照倒序对每个数组元素调用 `callback()` 方法，返回最后一个 `callback()` 的返回值。
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