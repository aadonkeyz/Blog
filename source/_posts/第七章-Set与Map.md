---
title: 第七章 Set与Map
categories:
  - JavaScript
  - 《深入理解ES6》
abbrlink: d766545b
date: 2019-04-01 21:03:23
---

`Set`是不包含重复值的列表。你一般不会像对待数组那样来访问`Set`中的某个项；相反更常见的是，只在`Set`中检查某个值是否存在。

`Map`则是键与想对应的值的集合。因此，`Map`中的每个项都存储了两块数据，通过指定所需读取的键即可检索对应的值。`Map`常被用作缓存，存储数据以便此后快速检索。

# ES5中的Set与Map

在ES5中，开发者使用对象属性来模拟`Set`与`Map`，就像这样：

```js
let set = Object.create(null)

set.foo = true

// 检查属性的存在性
if (set.foo) {
    // 一些操作
}
```

本例中的`set`变量是一个原型为`null`的对象，确保在此对象上没有继承属性。使用对象的属性作为需要检查的唯一值在ES5中是很常用的方法。当一个属性被添加到`set`对象时，它的值也被设为`true`，因此条件判断语句就可以简单判断出该值是否存在。

使用对象模拟`Set`与模拟`Map`之间唯一真正的区别是所存储的值。例如：以下例子将对象作为`Map`使用：

```js
let map = Object.create(null)

map.foo = 'bar'

// 提取一个值
let value = map.foo

console.log(value)    // bar
```

此代码将字符串值`"bar"`存储在`foo`键上。与`Set`不同，`Map`多数被用来提取数据，而不是仅检查键的存在性。

# 变通方法的问题

尽管在简单情况下将对象作为`Set`与`Map`来使用都是可行的，但一旦接触到对象属性的局限性，此方式就会遇到更多麻烦。研究如下代码：

```js
let map = Object.create(null)
    key1 = 5,
    key2 = '5',
    key3 = {},
    key4 = {}

map[key1] = 'number'

map[key3] = 'foo'

console.log(map[key2])      // number
console.log(map[key4])      // foo
console.log(map)            // { '5': 'number', '[object Object]': 'foo' }
```

上面例子中问题的来源是对象属性只可以是字符串或者符号类型，所以当你为对象定义属性时，如果属性名不是字符串类型，那么它会在内部将其转换为对应的字符串类型，然后再进行后续操作。所以`map[key1] === map[key2]`、`map[key3] === map[key4]`。

# ES6的Set

ES6新增了`Set`类型，这是一种无重复值的有序列表。`Set`允许对它包含的数据进行快速访问，从而增加了一个追踪离散值的更有效方式。

## 创建Set并添加项目

`Set`使用`new Set()`来创建，而调用`add()`方法就能向`Set`中添加项目，检查`size`属性还能查看其中包含有多少项。

```js
let set = new Set()

set.add(5)
set.add('5')

console.log(set.size)   // 2
console.log(set)        // Set { 5, '5' }
```

`Set`不会使用强制类型转换来判断值是否重复。这意味着`Set`可以同时包含数值`5`与字符串`"5"`，将它们都作为相对独立的项（在`Set`内部的比较使用了`Object.is()`方法来判断两个值是否相等，唯一的例外+0与-0被判断为是相等的）。你还可以向`Set`添加多个对象，它们不会被合并为同一项：

```js
let set = new Set(),
    key1 = {},
    key2 = {}

set.add(key1)
set.add(key2)

console.log(set.size)   // 2
console.log(set)        // Set { {}, {} }
```

由于`key1`与`key2`并不会被转换为字符串，所以它们在这个`Set`内部被认为是两个不同的项（记住：如果它们被转换为字符串，那么都会等于`"[object Object]"`）。

如果`add()`方法用相同值进行了多次调用，那么在第一次之后的调用实际上会被忽略：

```js
let set = new Set()

set.add(5)
set.add('5')
set.add(5)      // 重复了，该调用被忽略

console.log(set.size)   // 2
console.log(set)        // Set { 5, '5' }
```

你可以使用数组来初始化一个`Set`，并且`Set`构造器会确保不重复地使用这些值。

```js
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5])
console.log(set.size)   // 5
```

`Set`构造器实际上可以接收任意可迭代对象作为参数。能使用数组是因为它们默认就是可迭代的，`Map`也是一样的。

你可以使用`has()`方法来检测某个值是否存在于`Set`中，就像这样：

```js
let set = new Set()

set.add(5)
set.add('5')

console.log(set.has(5))     // true
console.log(set.has(6))     // false
```

## 移除值

使用`delete()`方法可以移除`Set`中的某个值，使用`clear()`方法可以移除`Set`中的所有值。

```js
let set = new Set()
set.add(5)
set.add('5')

set.delete(5)
console.log(set.has(5))     // false

set.clear()
console.log(set.size)       // 0
```

## Set上的forEach()方法

`Set`上的`forEach()`方法类似于数组中的`forEach()`方法，它接收两个参数：要在每一项上运行的函数和（可选的）运行该函数时的`this`值。传入的函数会接收三个参数：`Set`元素的值，`Set`元素的值和目标`Set`自身。你没看错，第一个和第二个参数是完全相同的，这么设计的目的是为了保持所有对象上`forEach()`方法的统一。

## 将Set转换为数组

使用扩展运算符可以很轻松的将`Set`转换为数组，用一个数组去重的例子来展示它是如何转换的：

```js
function eliminateDuplicates (array) {
    return [...new Set(array)]
}

let numbers = [ 1, 2, 3, 3, 3, 4, 5 ],
    noDuplicates = eliminateDuplicates(numbers)

console.log(noDuplicates)       // [ 1, 2, 3, 4, 5 ]
```

## Weak Set
### 创建Weak Set
### Set类型之间的关键差异

# ES6的Map
## Map的方法
## Map的初始化
## Map上的forEach方法
## Weak Map
### 使用Weak Map
### Weak Map的初始化
### Weak Map的方法
### 对象的私有数据
### Weak Map的用法与局限性