---
title: 强制类型转换
categories:
  - JavaScript
abbrlink: 1750eb3e
date: 2019-05-05 13:40:29
---

# 强制类型转换规则

## 引用类型转换为基本类型

{% note info %}
1. 调用自身的 `valueOf()` 方法，如果返回基本类型的值，则转换成功。
2. 如果 `valueOf()` 方法返回的还是引用类型值，则改为调用自身的 `toString()` 方法。如果 `toString()` 方法返回基本类型的值，则转换成功。
3. 如果 `toString()` 方法返回的是引用类型值，抛出错误。

需要注意的是，**数组的默认 `toString()` 方法经过了重新定义，会将所有单元字符串化以后再用 `","` 连接起来。**
{% endnote %}

## 转换为字符串

{% note info %}
- `null`：`"null"`。
- `undefined`：`"undefined"`。
- 布尔值：`"true"` 或 `"false"`。
- 数值：通常直接转换为字符串即可，但是对于极小或极大的数值，会使用指数形式。
- 引用类型值：返回自身的 `toString()` 方法的返回值。
{% endnote %}

```js
// "null"
console.log(String(null))                    

// "undefined"
console.log(String(undefined))                  

// "false"
console.log(String(false))                      

// "123"
console.log(String(123))                        

// "1.23e+21"
console.log(String(123 * 10000000000000000000)) 

// "[object Object]"
console.log(String({a: 1}))                     

// "1,2,3"
console.log(String([1, 2, 3]))                  
```

## 转换为数值

{% note info %}
- `null`：`0`。
- `undefined`：`NaN`。
- 布尔值：`0` 或 `1`。
- 字符串：十进制数值或 `NaN`。
- 引用类型值：先将引用类型值转换为基本类型值，然后再将该基本类型值转换为数值。
{% endnote %}

```js
// 0
console.log(Number(null))           

// NaN
console.log(Number(undefined))      

// 0
console.log(Number(false))          

// 123
console.log(Number('123'))          

// NaN
console.log(Number('123abc'))       

// NaN
console.log(Number({a: 1}))         

// NaN
console.log(Number([1, 2, 3]))      

var obj = {
  valueOf () {
    return 42
  }
}

var array = [1, 2, 3]
array.valueOf = function () {
  return 123
}

// 42
console.log(Number(obj))            

// 123
console.log(Number(array))          
```

## 转换为布尔值

{% note info %}
在将值转换为布尔值时，**除了下述 5 种情况，其他所有情况都会转换为 `true`**：
- `null`
- `undefined`
- `false`
- `+0`、`-0` 或 `NaN`
- `""`
{% endnote %}

# 显示强制类型转换

{% note info %}
- 一元运算符 `+`、`-`、`++` 和 `--` 都会调用 `Number()` 将其他类型转换为数值，或将日期对象转换为对应的毫秒数。注意不要混淆 `+`、`++`、`+ +`，`-`、`--`、`- -`。
- 一元运算符 `~` 会先将值强制类型转换为 32 位数值，然后执行按位非操作，可以将 `~x` 等价于 `-(x+1)`。但是 `~-1` 的结果是 `0` 而不是 `-0`，因为 `~` 是字位操作而非数学运算。
- 将值转换为布尔值时，可以使用 `!` 运算符。
{% endnote %}

```js
var a = '1'
var b = +a
var c = -a
var d = - -a
var e = --a

// 1
console.log(b)      

// -1
console.log(c)      

// 1
console.log(d)      

// 0
console.log(e)      

var f = new Date()

// 1557055687256
console.log(+f)     

// ==============================

var x = -42

// 41
console.log(~x)     

// ==============================

var a = '0'
var b = []
var c = {}
var d = ''
var e = 0
var f = null
var g

// true
console.log(!!a)     

// true
console.log(!!b)     

// true
console.log(!!c)     

// false
console.log(!!d)     

// false
console.log(!!e)     

// false
console.log(!!f)     

// false
console.log(!!g)     
```

# 隐式强制类型转换

{% note warning %}
**`+` 和 `-` 作为一元运算符和作为多元运算符时具有不同的含义，别混淆了！**
{% endnote %}

## + 运算符

`+` 运算符既能用于数值加法，也能用于字符串拼接。JavaScript 怎样来判断我们要执行的是哪个操作？

{% note info %}
1. 如果操作数中有字符串，进行字符串拼接操作。
2. 如果操作数中有引用类型值，首先将引用类型值转换为基本类型值，然后进行后续操作。
3. 其他情况全都进行数值加法。
{% endnote %}

## - 运算符

与 `+` 运算符不同，`-` 运算符会只会执行减法运算。所以它会先将非数值类型的数据转换为数值，然后进行减法运算。

```js
var a = '12'
var b = 1

// 11
console.log(a-b)    
```

# 宽松相等和严格相等

{% note info %}
- `==` 允许在相等比较中进行强制类型转换，而 `===` 不允许。
- `NaN` 不等于 `NaN`。
- `+0` 严格等于 `-0`。
- `!=` 与 `!==` 分别类似于 `==` 与 `===`。
{% endnote %}

下面主要介绍 `==` 是如何进行强制类型转换的。

## 字符串和数值

{% note info %}
首先将字符串转换为数值类型，然后进行比较。
{% endnote %}

```js
var a = 42
var b = '42'

// true
console.log(a == b)     
```

## 其他类型和布尔类型

{% note info %}
首先将布尔类型的值转换为数值类型，然后进行比较。
{% endnote %}

```js
var a = '42'
var b = true

// false
console.log(a == b)         

// true
console.log('1' == true)    
```

## null 和 undefined

{% note info %}
在 `==` 中 `null` 和 `undefined` 相等（它们也与其自身相等），除此之外其他情况都不相等。
{% endnote %}

```js
var a = null
var b

// true
console.log(a == b)         

// true
console.log(a == null)      

// true
console.log(b == null)      

// false
console.log(a == false)     

// false
console.log(b == false)     

// false
console.log(a == '')        

// false
console.log(b == '')        

// false
console.log(a == 0)         

// false
console.log(a == 0)         
```

## 引用类型和基本类型

{% note info %}
首先将引用类型转换为基本类型，然后进行比较。
{% endnote %}

```js
var a = 42
var b = [42]

// true
console.log(a == b)     
```

## 引用类型和引用类型

这种情况就是判断两个变量的引用是否相同