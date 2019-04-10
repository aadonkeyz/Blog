---
title: 第九章 JS的类
categories:
  - JavaScript
  - 《深入理解ES6》
abbrlink: e1a12c4d
date: 2019-04-10 20:14:09
---

# 类的声明

## 基本的类声明

类声明以`class`关键字开始，其后是类的名称；剩余部分的语法看起来就像对象字面量中的方法简写，并且在方法之间不需要使用逗号。作为范例，此处有个简单的类声明：

```js
class PersonClass {

    // 等价于 PersonType 构造器
    constructor (name) {
        this.name = name
    }

    // 等价于 PersonType.prototype.sayName
    sayName () {
        console.log(this.name)
    }
}

let person = new PersonClass('Nicholas')
person.sayName()                                    // Nicholas

console.log(person instanceof PersonClass)          // true
console.log(person instanceof Object)               // true

console.log(typeof PersonClass)                     // function
console.log(typeof PersonClass.prototype.sayName)   // function
```

类声明允许你在其中使用特殊的`constructor`方法名称直接定义一个构造器，而不需要先定义一个函数再把它当做构造器使用。由于类的方法使用了简写语法，于是就不再需要使用`function`关键字。`constructor`之外的方法名称则没有特别的含义，因此可以随你高兴自由添加方法。

自有属性：该属性出现在实例上而不是原型上，只能在类的构造器或方法内部进行创建。在本例中，`name`就是一个自有属性。建议应在构造器函数内创建所有可能出现的自有属性，这样在类中声明变量就会被限制在单一位置（有助于代码检查）。

## 为何要使用类的语法

{% note info %}
- 类声明不会被提升，这与函数定义不同。类声明的行为与`let`相似，因此在程序的执行到达声明处之前，类会存在于暂时性死区内；
- 类声明中的所有代码会自动运行在严格模式下，并且也无法退出严格模式；
- 类的所有方法都是不可枚举的；
- 类的所有方法内部都没有`[[Construct]]`，因此使用`new`来调用它们会抛出错误；
- 调用类构造器时不使用`new`，会抛出错误；
- 在类的内部不允许重写类名，在类的外部则可以。
{% endnote %}

这样看来，上例中的`PersonClass`声明实际上就直接等价于一下未使用类语法的代码：

```js
// 直接等价于 PersonClass
let PersonType2 = (function () {
    'use strict'

    const PersonType2 = function (name) {
        // 确认函数被调用时使用了 new
        if (typeof new.target === 'undefined') {
            throw new Error('Constructor must be called with new.')
        }

        this.name = name
    }

    Object.defineProperty(PersonType2.prototype, 'sayName', {
        value: function () {

            // 确认函数被调用时没有使用 new
            if (typeof new.target !== 'undefined') {
                throw new Error('Method cannot be called with new.')
            }

            console.log(this.name)
        },
        enumerable: false,
        writable: true,
        configurable: true
    })
    
    return PersonType2
})()
```

首先要注意这里有两个`PersonType2`声明：一个在外部作用域的`let`声明，一个在IIFE内部的`const`声明。这就是为何类的方法不能对类名进行重写、而类外部的代码则被允许。构造器函数检查了`new.target`，以保证被调用时使用了`new`，否则就抛出错误。接下来，`sayName()`方法被定义为不可枚举，并且此方法也检查了`new.target`，它则要保证在被调用时没有使用`new`。最后一步是将构造器函数返回出去。

# 类表达式

类与函数有相似之处，即它们都有两种形式：声明与表达式。函数声明与类声明都以适当的关键字为起始（分别是`function`与`class`），随后是标识符（即函数名或类名）。函数具有一种表达式形式，无须在`function`后面使用标识符；类似的，类也有不需要标识符的表达式形式。类表达式被设计用于变量声明，或可作为参数传递给函数。

## 基本的类表达式

```js
let PersonClass = class {

    // 等价于 PersonType 构造器
    constructor (name) {
        this.name = name
    }

    // 等价于 PersonType.prototype.sayName
    sayName () {
        console.log(this.name)
    }
}

let person = new PersonClass('Nicholas')
person.sayName()                                    // Nicholas

console.log(person instanceof PersonClass)          // true
console.log(person instanceof Object)               // true

console.log(typeof PersonClass)                     // function
console.log(typeof PersonClass.prototype.sayName)   // function
```

## 具名类表达式

上面的例子使用了一个匿名的类表达式，不过就像函数表达式那样，你也可以为类表达式命名。为此需要在`class`关键字后添加标识符，就像这样：

```js
let PersonClass = class PersonClass2 {

    // 等价于 PersonType 构造器
    constructor (name) {
        this.name = name
    }

    // 等价于 PersonType.prototype.sayName
    sayName () {
        console.log(this.name)
    }
}

console.log(typeof PersonClass)                     // function
console.log(typeof PersonClass2)                    // undefined
```

与命名函数表达式相似，类表达式的标识符`PersonClass2`只在类定义内部存在，在外部则不存在。

# 作为一级公民的类

ES6延续了传统，让类成为一级公民，这就使得类可以被多种方式所使用。例如，类可以作为参数传入函数，也可以立即调用类构造器。

```js
let person = new class {
    constructor (name) {
        this.name = name
    }

    sayName () {
        console.log(this.name)
    }
}('Nicholas')

person.sayName()    // 'Nicholas'
```

# 访问器属性

自有属性需要在类构造器中创建，而类还允许你在原型上定义访问器属性。为了创建一个`getter`，要使用`get`关键字，并要与后方标识符之间留出空格；创建`setter`用相同方式，只是要换用`set`关键字。例如：

```js
class CustomHTMLElement {
    constructor (element) {
        this.element = element
    }

    get html () {
        return this.element.innerHTML
    }

    set html (value) {
        this.element.innerHTML = value
    }
}

var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, 'html')

console.log('get' in descriptor)           // true
console.log('set' in descriptor)           // true
console.log(descriptor.enumerable)         // false
```

# 需计算的成员名

# 生成器方法

# 静态成员

# 使用派生类进行继承
## 屏蔽类方法
## 继承静态成员
## 从表达式中派生类
## 继承内置对象
## Symbol.species属性

# 在类构造器中使用new.target
