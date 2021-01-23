---
title: 引用类型
categories:
  - JavaScript
abbrlink: 71de0ab2
mathjax: true
date: 2019-03-03 12:14:38
---

# Object类型

[对象&原型&继承](https://aadonkeyz.com/posts/bb5d40f2/)

[Object](https://aadonkeyz.com/posts/a7625ff7/)

# Array类型

[Array](https://aadonkeyz.com/posts/6e01c1fe/)

# RegExp类型

[RegExp](https://aadonkeyz.com/posts/ce76031c/)

# Function类型

[函数&作用域&闭包](https://aadonkeyz.com/posts/f1587c53/)

[函数](https://aadonkeyz.com/posts/9595c646/)

# Date类型

通过 `var now = new Date()` 可以创建一个日期对象，在调用 `Data` 构造函数而不传参数时，新创建的对象自动获得当前日期和时间（**当前日期和时间是浏览器从本机操作系统获取的时间，所以不一定准确！**）。如果想根据特定的日期和时间创建日期对象，必须传入表示该日期的毫秒数（即从 UTC 时间 2017 年 1 月 1 日午夜起至该日期经过的毫秒数）。为了简化这一计算过程，ES提出了两个方法：`Date.parse()`和`Date.UTC()`。

{% note info %}
- `Date.parse()`：接收一个表示日期的字符串参数，然后尝试根据这个字符串返回相应日期的毫秒数。但是它支持哪种日期格式并不一定。
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

# 基本包装类型

ES提供了 3 个**特殊的引用类型**：`Boolean`、`Number` 和 `String`。这些类型与引用类型相似，但同时也具有与各自的基本类型相应的特殊行为，我们称它们为基本包装类型。

**实际上，每当读取一个基本类型值的时候，后台会自动创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。**

**引用类型与基本包装类型的主要区别就是对象的生存期。**使用 `new` 操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中。而**自动创建**的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。

```js
// 创建了一个基本类型值并保存在s1中
var s1 = 'some text'        

// 调用对应的基本包装类型
s1.color = 'red'       

// undefined，因为基本包装类型的生存期已过
console.log(s1.color)    

// string
console.log(typeof s1)      

// 显示的创建了String的实例对象并保存在s2中，实际上就是创建了一个引用类型值
var s2 = new String('some text')    
s2.color = 'red'

// String {"some text", color: "red"}
console.log(s2)        

// object
console.log(typeof s2)      
```

通过上面例子认识到通过 `new` 显示创建的就是一个引用类型值，它已经不能称之为基本包装类型了。换句话说，基本包装类型都是后台自己创建的，不存在显示创建的情况。在第六章会介绍，通过 `new` 显示创建的就是一个引用类型的实例对象。**记住最好别用new将基本包装类型给实例化就行。**

**要注意的是，使用 `new` 调用基本包装类型的构造函数，与直接调用同名的转型函数是不一样的。**

```js
var value = '25'

// 转型函数
var number = Number(value)      

// number
console.log(typeof number)      

// 构造函数
var obj = new Number(value)     

// object
console.log(typeof obj)         
```

## Boolean类型

没啥说的。

## Number类型

{% note info %}
- `toFixed()`：按照指定的小数位返回数值的字符串表示，银行家喜欢用这个方法，这个方法不是真正意义的四舍五入。
- `toExponential()`：返回以指数表示法表示的数值的字符串形式。
- `toPrecision()`：返回固定大小格式，也可能返回指数格式。
{% endnote %}

## String类型

### 字符方法

`charAt()` 和 `charCodeAt()`方法都接收一个参数，即基于 0 的字符位置。`charAt()` 方法以单字符字符串的形式返回给定位置的那个字符，而 `charCodeAt()` 方法返回的是字符编码。

```js
var string = 'hello world'

// e
console.log(string.charAt(1))       
// 101
console.log(string.charCodeAt(1))   
```

ES5 还定义了访问个别字符的方法，可以像数组那样使用索引来访问某个位置的字符。

```js
var string = 'hello'

// e
console.log(string[1])      
```

### 操作方法

#### 提取子字符串方法

{% note info %}
- `slice()`：类比于数组的`slice()`方法。
- `substr()`：第一个参数用于指定子字符串的开始位置，第二个参数用于指定返回的字符个数;
- `substring()`：类比于数组的`slice()`方法。
{% endnote %}

当传递负值作为参数时，`slice()` 方法会将传入的负值与字符串的长度相加。`substr()` 方法将负的第一个参数加上字符串的长度，而将负的第二个参数转换为0。`substring()` 方法会把所有负值参数都转换为0。

**另外需要注意的是，`substring()` 方法会将参数中较小的数作为开始位置，将较大的数作为结束位置。即 `string.substring(3, 0)` 等价于 `string.substring(0, 3)`**。

```js
var string = 'hello world'

// rld
console.log(string.slice(-3))         
// hello world  
console.log(string.substring(-3))     
// rld  
console.log(string.substr(-3))          

// lo w
console.log(string.slice(3, -4))       
// hel 
console.log(string.substring(3, -4))    
// ''
console.log(string.substr(3, -4))       
```

#### concat

类比于数组的`concat()`方法。

#### trim

ES5 定义了这个方法，会创建一个字符串的副本，删除前置及后缀的所有空格，然后返回结果。

#### repeat

该方法接收一个参数作为字符串的重复次数，返回一个将初始字符串重复指定次数的新字符串。

```js
// xxx
console.log('x'.repeat(3))          

// hellohello
console.log('hello'.repeat(2))      

// abcabcabcabc
console.log('abc'.repeat(4))        
```

### 位置方法

`indexOf(searchValue[, fromIndex])` 和 `lastIndexOf(searchValue[, fromIndex])` 都是找到就返回索引，找不到就返回 -1。

### 大小写转换方法

`toLowerCase()`、`toLocaleLowerCase()`、`toUpperCase()` 和 `toLocaleUpperCase()` 方法。

### 模式匹配方法

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

// cond, bat, sat, fat
console.log(result)     

result = text.replace(/at/g, 'ond')

// cond, bond, sond, fond
console.log(result)     

result = text.replace(/(.at)/g, 'word ($1)')

// word (cat), word (bat), word (sat), word (fat)
console.log(result)     
```

#### split

{% note info %}
`split()` 方法可以基于指定的分隔符将一个字符串分割成多个子字符串并将结果存放在一个数组中。分隔符可以是字符串，也可以是一个正则表达式。该方法有一个可选的第二参数，用于指定数组的大小。**如果将空字符串作为分隔符传递给 `split()`，那么字符串会基于每一个 UTF-16 codeunit 进行划分。如果传递给 `split()` 的正则表达式中包含捕获组，那么捕获组会被保留下来**
{% endnote %}

```js
var colorText = 'red,blue,green,yellow'

// [ 'red', 'blue', 'green', 'yellow' ]
console.log(colorText.split(','))         

// [ 'red', 'blue' ]  
console.log(colorText.split(',', 2))       

// [ 'red', 'blue', 'green', 'yellow' ]
console.log(colorText.split(/\,/))          

// [ 'red', ',', 'blue', ',', 'green', ',', 'yellow' ]
console.log(colorText.split(/(\,)/))        

// [ '', ',', ',', ',', '' ]
console.log(colorText.split(/[^\,]+/))      

// [ '', 'red', ',', 'blue', ',', 'green', ',', 'yellow', '' ]
console.log(colorText.split(/([^\,]+)/))    
```

#### 识别子字符串的方法

ES6新增了三个方法：

{% note info %}
- `includes(searchString[, fromIndex])`：在给定文本出现在字符串的任意位置时返回 `true`，否则返回 `false`；
- `startsWith(searchString[, fromIndex])`：在给定文本出现在字符串起始处时返回 `true`，否则返回 `false`；
- `endsWith(searchString[, length])`：在给定文本出现在字符串结尾处时返回 `true`，否则返回 `false`。
{% endnote %}

详细 API 请移步 MDN，这里有 demo。

```js
var msg = 'hello world!'

// true
console.log(msg.startsWith('hello'))   
// false
console.log(msg.startsWith('o')) 
// true
console.log(msg.startsWith('o', 4)) 

// true
console.log(msg.endsWith('!'))         
// true
console.log(msg.endsWith('world!'))  
// true
console.log(msg.endsWith('o', 8)) 

// true
console.log(msg.includes('o'))
// false
console.log(msg.includes('x'))
// false
console.log(msg.includes('o', 8))       
```

### localeCompare

[MDN 上的介绍](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare)


### fromCharCode

该方法接收一个或多个字符编码，然后将它们转换成字符串。

### 模板字面量

#### 基本语法

模板字面量的最简单语法，是使用反引号（`）来包裹普通字符串，而不是用双引号或单引号。若你想在字符串中包含反引号，只需要使用反斜杠（\）转义即可。

```js
let message = `hello \` world`

// hello ` world
console.log(message)        
```

#### 多行字符串

ES6 的模板字面量使多行字符串更易创建，因为它不需要特殊的语法。只需在想要的位置包含换行即可，而且它会显示在结果中。

```js
let message = `Multiline
string`

// Multiline
// string
console.log(message)            
                                
// 16
console.log(message.length)     
```

反斜杠之内的所有空白符都是字符串的一部分，因此需要留意缩进。

```js
let message = `Multiline
              string`

// Multiline
//              string
console.log(message)            

// 30
console.log(message.length)     
```

如果让多行文本保持合适的缩进对你来说很重要，请考虑将多行模板字面量的第一行空置并在此后进行缩进，如下所示：

```js
let html = `
<div>
  <h1>Title</h1>
</div>
`.trim()
```

#### 制造替换位

此时模板字面量看上去仅仅是普通 JavaScript 字符串的升级版，但二者之间真正的区别在于前置的“替换位”。替换位允许你将任何有效的 JavaScript 表达式嵌入到模板字面量中，并将其结果输出为字符串的一部分。

替换位由起始的 `${` 与结束的 `}` 来界定，之间允许放入任意的 JavaScript 表达式。例如：

```js
var name = 'Nicholas'
var message = `hello, ${name}.`

// hello, Nicholas.
console.log(message)        

var count = 10
var price = 0.25
var message = `${count} items cost $${(count * price).toFixed(2)}.`

// 10 items cost $2.50.
console.log(message)        

var name = 'Nicholas'
var message = `hello, ${
  `my name is ${name}`
}.`

// hello, my name is Nicholas.
console.log(message)        
```

#### 标签化模板

一个标签仅是一个函数，只不过这个函数的调用形式有些特别：``tagName`模板字面量数据` ``

标签函数会将调用时接收的模板字面量数据中的非替换位提取出来保存为一个数组，这个数组将作为标签函数的第一个参数；数据中的每个替换位的解释值都将作为标签函数的参数传递进来。为了方便处理，一般在定义标签函数时使用[剩余参数](http://aadonkeyz.com/posts/9595c646/#剩余参数)形式。

```js
function passthru (literals, ...substitutions) {
  // ["", " items cost $", ""]
  console.log(literals)           

  // [10, "2.50"]
  console.log(substitutions)     
 
  return '返回啥都行'
}

var count = 10
var price = 0.25
var message = passthru`${count} items cost $${(count * price).toFixed(2)}`
```

注意观察上面的例子，**`literals` 数组的第一个和最后一个元素均为空字符串，说明标签函数在对传入的模板字面量进行处理时，总是以非替换位形式的字符串作为数据的开始和结尾；如果模板字面量实际上是以替换位作为开始和结尾，那么会自动在开始和结尾处添加上空字符串！所以 `literals` 的长度总是比 `substitutions` 的长度多 1**。

模板标签也能访问字符串的原始信息，主要指的是可以访问字符在转义之前的形式。获取原始字符串值的最简单方式是使用内置的 `String.raw()` 标签。不过我按照书中例子试验的时候，关于原始形式这里有些出入，详细的就不介绍了。

# 单体内置对象

内置对象的定义是：“由ECMAScript实现提供的、不依赖于宿主环境的对象，这些对象在ECMAScript程序执行之前就已经存在了。”意思就是说，开发人员不必显示地实例化内置对象，因为它们已经实例化了。前面介绍的 `Object`、`Array`、`String` 等都是内置对象，除此之外还有两个单体内置对象：`Global` 和 `Math`。

## Global对象

`Global` (全局)对象可以说是 ECMAScript 中最特别的一个对象了，因为不管你从什么角度上看，这个对象都是不存在的。ES中的`Global` 对象在某种意义上是作为一个终极的“兜底儿的对象”来定义的。换句话说，不属于任何其他对象的属性和方法，最终都是它的属性和方法。事实上，没有全局变量或全局函数。所有在全局作用域中定义的属性和函数，都是 `Global` 对象的属性。

ES虽然没有指出如何直接访问 `Global` 对象,但 Web 浏览器都是将这个全局对象作为 `window` 对象的一部分加以实现的。

```js
// 取得 Global 对象的方法
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

// http://www.wrox.com/illegal%20value.html#start
console.log(encodeURI(uri))

// http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.html%23start
console.log(encodeURIComponent(uri))
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
- `Math.ceil()`：执行向上（大）舍入，即它总是将数值向上舍入为最接近的整数。
- `Math.floor()`：执行向下（小）舍入，即它总是将数值向下舍入为最接近的整数。
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