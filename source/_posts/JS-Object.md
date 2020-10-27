---
title: Object
abbrlink: a7625ff7
date: 2020-10-24 23:25:59
categories:
  - JavaScript
---

# 基础概念

**我们看到的大多数引用类型值都是 Object 类型的实例！对，连数组、函数等都是 Object 类型的实例！**

我所知道的例外有 `Object.create(null)` 返回的空对象。

{% note info %}
- 创建 Object 实例方式有：`new Object()` 和**对象字面量表示法**。
- 对象属性名一定是**字符串**或**符号**，如果不是，会自动转换为字符串。
- 访问对象属性的方式：**点表示法**和**方括号表示法**。
{% endnote %}

# 对象字面量语法

## 属性初始化的速记法

在 ES6 中，你可以使用属性初始化的速记法来消除对象名称与本地变量的重复情况。当对象的一个属性名称与本地变量名相同时，你可以简单书写名称而省略冒号与值。例如：

```js
function createPerson (name, age) {
  return {
    name,
    age
  }
}
```

当对象字面量中的属性只有名称时，引擎会在周边作用域查找同名变量。若找到，该变量的值将会被赋给对象字面量的同名属性。在本例中，局部变量 `name` 的值就被赋给了对象的 `name` 属性。

## 方法简写

ES6 允许在为对象字面量方法赋值时，省略冒号与 `function` 关键字。

```js
var person = {
  name: 'Nicholas',
  sayName () {
    console.log(this.name)
  }
}
```

## 需计算属性名

在 ES5 及更早版本中，可以在对象字面量中将字符串直接用作属性名，就像这样：

```js
var person = {
  'first name': 'Nicholas'
}

// Nicholas
console.log(person['first name'])
```

但是前提是这个属性名事先已知，并且能用字符串表示。如果用对象字面量语法创建对象时，想为它定义一个尚未确定名称的属性，ES5是做不到这点的。但是ES6可以呀！

在 ES6 中，需计算属性名是对象字面量语法的一部分，它用的也是方括号表示法。

```js
var lastName = 'last name'

var person = {
  'first name': 'Nicholas',
  [lastName]: 'Zakas'
}

// Nicholas
console.log(person['first name'])

// Zakas
console.log(person[lastName])
```

在对象字面量内的方括号表明该属性需要计算，其结果是一个字符串。这意味着其中可以包含表达式，像下面这样：

```js
var suffix = ' name'

var person = {
  ['first' + suffix]: 'Nicholas',
  ['last' + suffix]: 'Zakas'
}

// Nicholas
console.log(person['first name'])

// Zakas
console.log(person['last name'])
```

## 重复的对象字面量属性

在 ES5 严格模式下如果出现重复的对象字面量属性，会抛出错误。但 ES6 移除了重复属性的检查，严格模式与非严格模式都不在检查重复的属性。当存在重复属性时，排在后面的属性的值会成为该属性的实际值。

# 自有属性的枚举顺序

ES6 并没有定义对象属性的枚举顺序，而是把该问题留给了 JS 引擎厂商。而 ES6 则严格定义了对象自有属性在被枚举时返回的顺序。这对 `Object.getOwnPropertyNames()` 与 `Reflect.ownKeys` 如何返回属性造成了影响，还同样影响了 `Object.assign()` 处理属性的顺序。

自有属性枚举时基本顺序如下：

{% note info %}
1. 所有的数字类型键，按升序排列。
2. 所有的字符串类型键，按被添加到对象的顺序排列。
3. 所有的符号类型键，也按添加顺序排列。
{% endnote %}

```js
var obj = {
  a: 1,
  0: 1,
  c: 1,
  '2': 1,
  b: 1,
  1: 1
}

obj.d = 1

// 0,1,2,a,c,b,d
console.log(Object.getOwnPropertyNames(obj).join(','))
```

{% note warning %}
**`for-in` 循环的枚举顺序仍未被明确规定，因为并非所有的 JS 引擎都采用相同的方式。而 `Object.keys()` 和 `JSON.stringify()` 也使用了与 `for-in` 一样的枚举顺序。**
{% endnote %}

# super

{% note warning %}
- `super` 是指向**当前对象**的原型的一个引用，并且 `super` 只能在对象的简写方法中使用。
- 虽然 `super` 是当前对象原型的一个引用，但是通过 `super` 调用原型方法时，被调用的原型方法内部的 `this` 会与当前对象方法内部的 `this` 保持一致。
- ~~`super` 就是 `Object.getPrototypeOf(this)` 的值。~~这其实是错误的。
{% endnote %}

```js
let proto = {
  where: 'proto 自己',
  func () {
    console.log(this.where + '调用 proto 的 func')
  }
}

let superObj = {
  where: 'superObj',
  find () {
    super.func()
  }
}

let callObj = {
  where: 'callObj',
  find () {
    Object.getPrototypeOf(this).func()
  }
}

Object.setPrototypeOf(superObj, proto)
Object.setPrototypeOf(callObj, proto)

// superObj 调用 proto 的 func
superObj.find()

// proto 自己调用 proto 的 func
callObj.find()

let otherProto = {
  where: 'otherProto 自己',
  func () {
    console.log(this.where + '调用 otherProto 的 func')
  }
}

let otherObj = {
  where: 'otherObj'
}

Object.setPrototypeOf(otherObj, otherProto)

// otherObj 调用 proto 的 func
superObj.find.call(otherObj)

// otherProto 自己调用 otherProto 的 func
callObj.find.call(otherObj)
```

# 正式的“方法”定义

在 ES6 之前，“方法”的概念从未被正式定义，它从前仅指对象的函数属性（而非数据属性）。ES6 则正式做出了定义：方法是一个拥有 `[[HomeObject]]` 内部属性的函数，此内部属性指向该方法所属的对象。

{% note info %}
使用 `super` 调用原型方法时，背后的工作流程。
1. 在 `[[HomeObject]]` 上调用 `Object.getPrototypeOf()` 来获取对原型的引用。
2. 在该原型上查找同名函数。
3. 创建 `this` 绑定并调用该方法。
{% endnote %}

# Object.assign

该方法接受一个接收者，以及任意数量的供应者，接收者会按照供应者在参数中的顺序来依次接收它们的属性，最后返回接收者。在这一过程中如果出现属性名重复的情况，则后面的会覆盖前面的。

```js
var receiver = {
  name: 'origin'
}

Object.assign(receiver,
  {
    type: 'js',
    name: 'file.js'
  },
  {
    type: 'css'
  }
)

// { type: 'css', name: 'file.js' }
console.log(receiver)
```

需要注意的是，**`Object.assign()` 并未在接收者上创建访问器属性，即使供应者拥有访问器属性。由于 `Object.assign()` 使用赋值运算符，供应者的访问器属性就会转变成接收者的数据属性**，例如：

```js
var receiver = {},
  supplier = {
    get name () {
      return 'file.js'
    }
  }

Object.assign(receiver, supplier)

var descriptor = Object.getOwnPropertyDescriptor(receiver, 'name')

// file.js
console.log(descriptor.value)

// undefined
console.log(descriptor.get)
```

# Object.create

`Object.create(proto, [propertiesObject])` 以 `proto` 为原型创建一个新的对象。`Object.create()` 方法的第二个参数与 `Object.defineProperties()` 方法的第二个参数格式相同。

```js
var person = {
  name: 'Nicholas',
  friend: ['Shelby', 'Court', 'Van']
}
var anotherPerson = Object.create(person, {
  name: {
    value: 'Greg',
    enumerable: false,
  }
})

// true
console.log(person.isPrototypeOf(anotherPerson))

// {}
console.log(anotherPerson)

// Greg
console.log(anotherPerson.name)

// [ 'Shelby', 'Court', 'Van' ]
console.log(anotherPerson.friend)
```

# Object.is

当比较两个值的时候，大多数开发者倾向于使用严格相等运算符（`===`），但是它有一点小瑕疵。例如，它认为 `+0` 与 `-0` 相等，即使这两者在引擎中有不同的表示。另外 `NaN === NaN` 会返回 `false`，因此有必要使用 `Number.isNaN()` 函数来正确检测 `NaN`。

ES6 引入了 `Object.is()` 方法来弥补严格相等运算符残留的怪异点。此方法接受两个参数，并会在二者的值相等时返回 `true`，此时要求二者类型相同并且值也相等。

```js
// false
console.log(Object.is(+0, -0))

// true
console.log(Object.is(NaN, NaN))
```

在许多情况下，`Object.is()` 的结果与 `===` 运算符是相同的，仅有的例外是：它会认为 `+0` 与 `-0` 不相等，而且 `NaN` 等于 `NaN`。


# Object.setPrototypeOf

ES6 添加了 `Object.setPrototypeOf` 方法，此方法允许你修改任意指定对象的原型。它接受两个参数：需要被修改原型的对象，以及将会成为前者原型的对象。

```js
let person = {
  getGreeting () {
    return 'hello'
  }
}

let dog = {
  getGreeting () {
    return 'woof'
  }
}

let friend = Object.create(person)

// hello
console.log(friend.getGreeting())

// true
console.log(Object.getPrototypeOf(friend) === person)

Object.setPrototypeOf(friend, dog)

// woof
console.log(friend.getGreeting())

// true
console.log(Object.getPrototypeOf(friend) === dog)
```