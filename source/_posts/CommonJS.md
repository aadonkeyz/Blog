---
title: CommonJS
categories:
  - Node.js
abbrlink: b3fe2fad
date: 2019-04-19 14:33:00
---

# CommonJs模块规范

CommonJs是Node.js的规范，是一直沿用至今的一个模块规范,。虽然ES6提出了新的模块规范，但目前为止Node.js无法直接兼容ES6，我们按照ES6的模块规范来书写代码，但是实际上它们最终会被编译为CommonJs规范对应的代码来执行。

需要记住，Node对获取的JavaScript文件内容进行了头尾包装。在头部添加了`function (exports, require, module, __filename, __dirname) {\n`，而在尾部添加了`\n}`。所以一个正常的JavaScript文件会被包装成如下的样子：

```js
function (exports, require, module, __filename, __dirname) {
    // .js文件的内容
    // ......
}
```

{% note info %}
模块文件5个变量的含义：
- **`exports`**：一个对象，用来挂载当前模块中要导出的内容；
- **`require`**：一个用来引入其他模块的函数；
- **`module`**：一个代表模块自身的对象；
- **`__filename`**：当前模块文件的绝对路径；
- **`__dirname`**：当前模块文件所在目录的绝对路径。

**注意千万不要重写这些变量，否则会导致它们与模块文件之间的联系被切断。**
{% endnote %}

下面的例子是我创建了一个名为test.js的文件并用node运行得到的：

```js
// test.js文件
console.log(exports)            // {}
console.log(typeof require)     // function
console.log(__filename)         // D:\Others\test.js
console.log(__dirname)          // D:\Others
console.log(module)             // Module {
                                //   id: '.',
                                //   exports: {},
                                //   parent: null,
                                //   filename: 'D:\\Others\\test.js',
                                //   loaded: false,
                                //   children: [],
                                //   paths: [ 'D:\\Others\\node_modules', 'D:\\node_modules' ]
                                // }
```

现在我创建了两个文件a.js和b.js，在a.js中演示`exports`的使用，在b.js中演示`require`的使用。

```js
// a.js
let name = 'a.js'

let sayName = function () {
    console.log(`my name is ${name}`)
}

exports.sayName = sayName
```
```js
// b.js
var moduleA = require('./a.js')

moduleA.sayName()      // my name is a.js
```

在a.js文件中，通过将函数`sayName()`挂载到`exports`对象上，来导出`sayName()`。而在b.js文件中，通过`require()`函数获取了a.js文件中要导出内容组成的对象。

我们已经知道了Node会将每个模块文件包装为一个函数，而`exports`对象实际上是通过形参的方式传入的，直接赋值形参会改变形参的引用。所以**为了防止无意间重写`exports`，推荐使用`module.exports`来替代它**。下面演示下初学者容易犯的错误，和推荐的用法：

```js
// a1.js
let name = 'a1.js'

let sayName = function () {
    console.log(`my name is ${name}`)
}

// 错误用法
exports = {
    name,
    sayName
}
```
```js
// a2.js
let name = 'a2.js'

let sayName = function () {
    console.log(`my name is ${name}`)
}
// 推荐用法
module.exports = {
    name,
    sayName
}
```
```js
// b.js
var moduleA1 = require('./a1.js')
var moduleA2 = require('./a2.js')

moduleA1.sayName()      // TypeError: moduleA.sayName is not a function

moduleA2.sayName()      // my name is a2.js
```

# 模块加载机制

CommonJS模块的重要特性是加载时执行，即模块文件内的代码会在被`require()`的时候，就会全部执行。当执行完成后，`module.exports`对象就代表着被加载的模块文件要导出的内容。

{% note info %}
有两点需要注意：
- `var moduleA = require('./a.js')`中的`moduleA`，是由a.js文件中的`module.exports`经过一次浅拷贝得到的；
- 同一个模块文件，也许会被多次加载，但是只会执行一次。
{% endnote %}

浅拷贝的操作类似于`clone`函数：

```js
function clone(obj) {
    let newObj = {}

    for (let key in ) {
        newObj[key] = obj[key]
    }

    return newObj
}
```

在明白上面说述内容后，请观察一个例子：

```js
// a.js
let count = 0
let obj = {
    num: 0
}

let show = function () {
    console.log(`count is: ${count}`)
    console.log(`obj.num is: ${obj.num}`)
}

let change = function () {
    count++
    obj.num++
}

module.exports = {
    count,
    obj,
    show,
    change
}
```
```js
// b.js
var moduleA = require('./a.js')

console.log(moduleA.count)      // 0
console.log(moduleA.obj.num)    // 0
moduleA.show()                  // count is: 0
                                // obj.num is: 0

moduleA.change()

console.log(moduleA.count)      // 0
console.log(moduleA.obj.num)    // 1
moduleA.show()                  // count is: 1
                                // obj.num is: 1
```

通过`change()`函数同时改变了`count`和`obj.num`的值。在a.js文件中，`count`和`obj.num`的值均变为`1`，而在b.js文件中`moduleA.count`的值还是为0没有变化，`moduleA.obj.num`的值则变为`1`。如果你没看懂，你可能需要[**看这里**](https://aadonkeyz.com/posts/9b1cd8c7/#复制变量值)。

# 模块循环引用

在CommonJs中如果出现某个模块被“循环加载”，只会输出已经执行的部分，还未执行的部分不会输出。下面贴出[官方文档里面的例子](https://nodejs.org/api/modules.html#modules_cycles)。

```js
// a.js
console.log('a starting')

exports.done = false

const b = require('./b.js')
console.log('in a, b.done = %j', b.done)

exports.done = true

console.log('a done')
```
```js
// b.js
console.log('b starting')

exports.done = false

const a = require('./a.js')
console.log('in b, a.done = %j', a.done)

exports.done = true

console.log('b done')
```
```js
// main.js
console.log('main starting')

const a = require('./a.js')
const b = require('./b.js')

console.log('in main, a.done = %j, b.done = %j', a.done, b.done)
```
```md
依次输出：
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```