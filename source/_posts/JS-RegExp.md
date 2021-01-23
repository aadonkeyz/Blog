---
title: RegExp
abbrlink: ce76031c
date: 2020-10-24 23:26:54
categories:
  - JavaScript
---

# 创建正则表达式

正则表达式可以通过 `var expression = /pattern/flags` 的字面量形式来创建。其中的 `pattern` 代表任何简单或复杂的正则表达式，`flag` 代表一个或多个标志。

{% note info %}
- `g`：表示全局模式，即模式将被应用于所有字符串，而非在发现第一个匹配项时立即停止。
- `i`：表示不区分大小写模式。
- `m`：表示多行模式。
- `y`：表示从正则表达式的 `lastIndex` 属性值的位置开始向后检索字符串中的匹配字符。如果在该位置没有匹配成功，那么正则表达式将停止检索并返回 `null`
{% endnote %}

同样也支持使用 `RegExp` 构造函数来创建正则表达式，它接收两个参数：一个是要匹配的字符串模式，另一个是可选的标志字符串。

```js
// pattern1 和 pattern2 的作用是完全一样的
var pattern1 = /[bc]at/i
var pattern2 = new RegExp('[bc]at', 'i')
```

{% note warning %}
由于 `RegExp` 构造函数的模式参数都是字符串，所以在某些情况下要对字符进行双重转义。所有元字符都必须双重转义，那些已经转义过的字符也是如此。
{% endnote %}

# 复制正则表达式

在 ES5 中，你可以将正则表达式传递给 `RegExp` 构造器来复制它，就像这样：

```js
var re1 = /ab/i
var re2 = new RegExp(re1)
```

`re2` 变量只是 `re1` 的一个副本。但如果你向 `RegExp` 构造器传递了第二个参数，即正则表达式的标志，那么该代码就无法正常工作，正如该范例：

```js
var re1 = /ab/i

// ES5中会抛出错误，ES6中可用
var re2 = new RegExp(re1, 'g')
```

如果你在 ES5 中运行这段代码，那么你会收到一条错误信息，表示在第一个参数已经是正则表达式的情况下不能再使用第二个参数。ES6 则修改了这个行为，允许使用第二个参数，并且让它覆盖第一个参数中的标志。例如：

```js
var re1 = /ab/i

// ES5 中会抛出错误，ES6 中可用
var re2 = new RegExp(re1, 'g')

// /ab/i
console.log(re1.toString())     

// /ab/g
console.log(re2.toString())     

// true
console.log(re1.test('ab'))     

// true
console.log(re2.test('ab'))     

// true
console.log(re1.test('AB'))     

// false
console.log(re2.test('AB'))     
```

# flags

在 ES5 中，你可以使用 `source` 属性来获取正则表达式的文本，但若想获取标志字符串，你必须解析 `toString()` 方法的输出，就像下面展示的那样：

```js
function getFlags (re) {
  var text = re.toString();
  return text.substring(text.lastIndexOf('/') + 1, text.length);
}

// toString() 的输出为 "/ab/g"
var re = /ab/g;

console.log(getFlags(re));
```

ES6 新增了 `flags` 属性用于配合 `source` 属性，让标志的获取变得更容易。这两个属性均为只有 `get` 函数，无 `set` 函数的访问器属性，因此都是只读的。

```js
var re = /ab/gi

// ab
console.log(re.source)      

// gi
console.log(re.flags)       
```

# y 标志

```js
var text = 'hello1 hello2 hello3'
var pattern = /hello\d\s?/
var globalPattern = /hello\d\s?/g
var stickyPattern = /hello\d\s?/y

var result = pattern.exec(text)
var globalResult = globalPattern.exec(text)
var stickyResult = stickyPattern.exec(text)

// hello1
console.log(result[0])                  

// hello1
console.log(globalResult[0])            

// hello1
console.log(stickyResult[0])            

// 0
console.log(pattern.lastIndex)          

// 7
console.log(globalPattern.lastIndex)    

// 7
console.log(stickyPattern.lastIndex)    

pattern.lastIndex = 1
globalPattern.lastIndex = 1
stickyPattern.lastIndex = 1

result = pattern.exec(text)
globalResult = globalPattern.exec(text)
stickyResult = stickyPattern.exec(text)

// hello1
console.log(result[0])                  

// hello2
console.log(globalResult[0])            

// Uncaught TypeError: Cannot read property '0' of null
console.log(stickyResult[0])            
```

有两个关于粘连标志的微妙细节需要牢记：

{% note warning %}
- 只有调用正则表达式对象上的方法（例如 `exec()` 与 `test()` 方法），`lastIndex` 属性才会生效。而将正则表达式作为参数传递给字符串上的方法（例如 `match()` 方法），并不会体现粘连特性；
- 当使用 `^` 字符来匹配字符串的起始处时，粘连的正则表达式只会匹配字符串的起始处（或者在多行模式下匹配行首）。当 `lastIndex` 为 0 时，`^` 不会让粘连的正则表达式与非粘连的有任何区别。而当 `lastIndex` 不为 0 时，粘连的正则表达式永远不会匹配成功。
{% endnote %}

可以使用如下方法来检测粘连标志是否被支持：

```js
function hasRegExpY () {
  try {
    var pattern = new RegExp('.', 'y')
    return true
  } catch (ex) {
    return false
  }
}
```

# exec

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

# test

`test()`方法接收一个字符串参数，在模式与该参数匹配的情况下返回 `true`，否则返回 `false`。

# 常用总结

## 常用元字符概述

| 元字符 | 描述 |
| :-: | :-: |
| `^` | 匹配字符串的起始位置。如果多行标志被设置为 `true`，那么也匹配换行符后紧跟的位置。 |
| `$` | 匹配字符串的结束位置。如果多行标志被设置为 `true`，那么也匹配换行符前的位置。 |
| `\b` | 匹配一个词的起始或结束位置。<br>例如，`\bm` 匹配 `moon` 中的 `m`，而 `oo\b` 并不匹配 `moon` 中的 `oo`。|
| `\B` | 匹配组成一个词的各个字符的中间位置。<br>例如，`\B..` 匹配 `noonday` 中的 `oo`，而 `y\B..` 匹配 `possibly yesterday` 中的 `yes`。 |
| `\` | 将元字符转义为字面量。例如，如果想在字符串中匹配 `^`，那么可以使用 `\^`。 |
| `{n}` | 匹配前一个表达式 `n` 次。`n` 必须为正整数。 |
| `{n,}` | 至少匹配前一个表达式 `n` 次。`n` 必须为正整数。 |
| `{n,m}` | 匹配前一个表达式 `n` 次到 `m` 次之间（包括 `n` 和 `m`）。`n` 和 `m` 必须为正整数，且 `n < m`。 |
| `*` | 匹配前一个表达式 0 次或多次，等价于 `{0,}`。 |
| `+` | 匹配前一个表达式 1 次或多次，等价于 `{1,}`。 |
| `?` | 匹配前一个表达式 0 次或 1 次，等价于 `{0,1}`。 |
| `.` | 匹配除换行符之外的任意单个字符。 |
| `(xyz)` | 匹配 `xyz` 并记住匹配项，括号内的内容被称为**捕获组**。<br>例如，`(foo)(bar)\1\2` 等价于 `foobarfoobar`。 |
| `(?:xyz)` | 匹配 `xyz` 但不记住匹配项。 |
| `x(?=y)` | 当且仅当 `x` 后面紧跟着 `y` 时，匹配 `x`。这叫做**先行断言**。 |
| `x(?!=y)` | 当且仅当 `x`后面没有紧跟着 `y` 时，匹配 `x`。这叫做**正向否定查找**。 |
| `(?<=y)x` | 当且仅当 `x` 前面就是 `y` 时，匹配 `x`。这叫做**后行断言**。 |
| `(?<!y)x` | 当且仅当 `x` 前面不是 `y` 时，匹配 `x`。这叫做**反向否定查找**。 |
| <code>x&#124;y</code> | 匹配 `x` 或 `y`。 |
| `[xyz]` | 匹配一个出现在方括号中的字符。**在方括号内部，你可以使用破折号 `-` 来指定一个范围**。<br>例如，`[a-z]` 代表匹配一个小写字母。 |
| `[^xyz]` | 匹配一个没有出现在方括号中的字符。**在方括号内部，你可以使用破折号 `-` 来指定一个范围**。<br>例如，`[^a-z]` 代表匹配一个小写字母以外的其他字符。 |
| `[\b]` | 匹配一个退格。 |
| `\cX` | 当 `X` 是处于 `A-Z` 之间的字符时，匹配字符串中的一个控制字符。<br>例如，`\cM` 匹配字符串中的 `control-M`。 |
| `\d` | 匹配一个数字字符。 |
| `\D` | 匹配一个非数字字符。 |
| `\f` | 匹配一个换页符。 |
| `\n` | 匹配一个换行符。 |
| `\r` | 匹配一个回车符。 |
| `\s` | 匹配一个空白符，包括空格、制表符、换页符和换行符。 |
| `\S` | 匹配一个非空白字符。 |
| `\t` | 匹配一个水平制表符。 |
| `\v` | 匹配一个垂直制表符。 |
| `\w` | 匹配一个单子字符（字母、数字或者下划线）。等价于 `[A-Za-z0-9_]`。 |
| `\W` | 匹配一个非单子字符。等价于 `[^A-Za-z0-9_]`。 |
| `\number` | 正数代表第 `number` 个捕获组。<br>例如，`apple(,)\sorange\1` 匹配 `apple, orange, cherry` 中的 `apple, orange,`。 |

## 贪婪/非贪婪匹配

参考自[【正则表达式系列】贪婪与非贪婪模式](https://dailc.github.io/2017/07/06/regularExpressionGreedyAndLazy.html)

{% note info %}
- **贪婪匹配**
  - **定义**：会尽量多的匹配符合条件的内容。
  - **元字符**：`+`、`?`、`*`、`{n}`、`{n,}`、`{n,m}`。
- **非贪婪匹配**
  - **定义**：会尽量少的匹配符合条件的内容，一旦发现符合规则的内容就立刻终止匹配。
  - **元字符**：`+?`、`??`、`*?`、`{n}?`、`{n,}?`、`{n,m}?`。
{% endnote %}

```js
let str = 'aacbacbc'

// aacbacb
console.log(str.match(/.*b/)[0])   

// aacb
console.log(str.match(/.*?b/)[0])  

// cb
console.log(str.match(/.?b/)[0])   

// cb
console.log(str.match(/.??b/)[0])  

// aacbacb
console.log(str.match(/.+b/)[0])   

// aacb
console.log(str.match(/.+?b/)[0])  
```