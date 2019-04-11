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

这样看来，上例中的`PersonClass`声明实际上就直接等价于以下未使用类语法的代码：

```js
let PersonClass = (function () {
    'use strict'

    const PersonClass = function (name) {
        // 确认函数被调用时使用了 new
        if (typeof new.target === 'undefined') {
            throw new Error('Constructor must be called with new.')
        }

        this.name = name
    }

    Object.defineProperty(PersonClass.prototype, 'sayName', {
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
    
    return PersonClass
})()
```

首先要注意这里有两个`PersonClass`声明：一个在外部作用域的`let`声明，一个在IIFE内部的`const`声明。这就是为何类的方法不能对类名进行重写、而类外部的代码则被允许。构造器函数检查了`new.target`，以保证被调用时使用了`new`，否则就抛出错误。接下来，`sayName()`方法被定义为不可枚举，并且此方法也检查了`new.target`，它则要保证在被调用时没有使用`new`。最后一步是将构造器函数返回出去。

# 类表达式

类与函数有相似之处，即它们都有两种形式：声明与表达式。函数声明与类声明都以适当的关键字为起始（分别是`function`与`class`），随后是标识符（即函数名或类名）。函数具有一种表达式形式，无须在`function`后面使用标识符；类似的，类也有不需要标识符的表达式形式。类表达式被设计用于变量声明，或可作为参数传递给函数。

## 基本的类表达式

```js
let PersonClass = class {

    constructor (name) {
        this.name = name
    }

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

    constructor (name) {
        this.name = name
    }

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

person.sayName()    // Nicholas
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

类方法与类访问器属性也都能使用需计算的名称。语法相同于对象字面量中的需计算名称。例如：

```js
let methodName = 'sayName'

class PersonClass {
    constructor (name) {
        this.name = name
    }

    [methodName] () {
        console.log(this.name)
    }
}

let me = new PersonClass('Nicholas')
me.sayName()                    // Nicholas

// ====================================
let propertyName = 'html'

class CustomHTMLElement {
    constructor (element) {
        this.element = element
    }

    get [propertyName] () {
        return this.element.innerHTML
    }

    set [propertyName] (value) {
        this.element.innerHTML = value
    }
}
```

# 生成器方法

在类语法中，允许将任何方法变为一个生成器。因此可以使用`Symbol.iterator`来定义生成器方法，从而定义出类的默认迭代器。

```js
class Collection {
    constructor () {
        this.items = []
    }

    *[Symbol.iterator] () {
        yield *this.items
    }
}

var collection = new Collection()
collection.items.push(1, 2, 3)

for (let x of collection) {
    console.log(x)      // 依次输出1, 2, 3
}
```

# 静态成员

直接在构造器上添加额外方法来模拟静态成员，这在ES5及更早版本中是另一个通用的模式。例如：

```js
function PersonClass (name) {
    this.name = name
}

// 静态方法
PersonClass.create = function (name) {
    return new PersonClass(name)
}

// 实例方法
PersonClass.prototype.sayName = function () {
    console.log(this.name)
}

var person = PersonClass.create('Nicholas')
```

在其他变成语言中，`PersonClass.create()`会被认定为一个静态方法，它的数据不依赖`PersonClass`的任何实例。ES6的类简化了静态成员的创建，只要在方法与访问器属性的名称前添加正式的`static`标注。作为一个例子，此处有个与上例等价的类：

```js
class PersonClass {

    constructor (name) {
        this.name = name
    }

    sayName () {
        console.log(this.name)
    }

    // 等价于 PersonType.create
    static create (name) {
        return new PersonClass(name)
    }
}

let person = PersonClass.create('Nicholas')
```

**类中的任何方法与访问器属性上都可以使用`static`关键字，唯一的限制是不能将它用于`constructor`方法。**

# 使用派生类进行继承

ES6之前，实现自定义类型的继承是个繁琐的过程。类的出现让继承工作变得更轻易，使用熟悉的`extends`关键字来指定当前类所需要继承的函数即可。生成的类的原型会被自动调整，而你还能调用`super()`方法来访问基类的构造器。此处有个例子：

```js
class Rectangle {
    constructor (length, width) {
        this.length = length
        this.width = width
    }

    getArea () {
        return this.length * this.width
    }
}

class Square extends Rectangle {
    constructor (length) {

        // 与 Rectangle.call(this, length, length) 相同
        super(length, length)
    }
}

var square = new Square(3)

console.log(square.getArea())               // 9
console.log(square instanceof Square)       // true
console.log(square instanceof Rectangle)    // true
```

此次`Square`类使用了`extends`关键字继承了`Rectangle`。`Square`构造器使用了`super()`配合指定参数调用了`Rectangle`的构造器。

继承了其他类的类被称为派生类。如果派生类指定了构造器，就需要使用`super()`，否则会造成错误。若你选择不使用构造器，`super()`方法会被自动调用，并会使用创建新实例时提供的所有参数。例如，下列两个类是完全相同的：

```js
class Square extends Rectangle {
    // 没有构造器
}

// 等价于：
class Square extends Rectangle {
    constructor (...args) {
        super(...args)
    }
}
```

此例中的第二个类展示了与所有派生类默认构造器等价的写法，所有的参数都按顺序传递给了基类的构造器。在当前需求下，这种做法并不完全准确，因为`Square`构造器只需要单个参数，因此最好手动定义构造器。

{% note info %}
使用`super()`时需要牢记以下几点：
- 你只能在派生类中使用`super()`。若尝试在非派生的类或函数中使用它，就会抛出错误；
- 在构造器中，你必须在访问`this`之前调用`super()`。由于`super()`负责初始化`this`，因此试图先访问`this`自然就会造成错误（具体原因看本文的[**继承内置对象**](https://aadonkeyz.com/posts/e1a12c4d/#继承内置对象)部分）；
- 唯一能避免调用`super()`的办法，是从派生类的构造器中返回一个对象。
{% endnote %}

## 屏蔽类方法

派生类中的方法总是会屏蔽基类的同名方法。例如，你可以将`getArea()`方法添加到`Square`类，以便重定义它的功能：

```js
class Rectangle {
    constructor (length, width) {
        this.length = length
        this.width = width
    }

    getArea () {
        return this.length * this.width
    }
}

class Square extends Rectangle {
    constructor (length) {
        super(length, length)
    }

    // 重写并屏蔽 Rectangle.prototype.getArea()
    getArea () {
        return this.length * this.length
    }
}
```

由于`getArea()`已经被定义为`Square`的一部分，`Rectangle.prototype.getArea()`方法就不能在`Square`的任何实例上被调用。当然，你总是可以使用`super.getArea()`方法来调用基类中的同名方法，就像这样：

```js
class Rectangle {
    constructor (length, width) {
        this.length = length
        this.width = width
    }

    getArea () {
        return this.length * this.width
    }
}

class Square extends Rectangle {
    constructor (length) {
        super(length, length)
    }

    // 重写、屏蔽并调用了 Rectangle.prototype.getArea()
    getArea () {
        return super.getArea()
    }
}
```

用这种方式使用`super`，其效果等同于在对象的简写方法中使用`super`。

## 继承静态成员

如果基类包含静态成员，那么这些静态成员在派生类中也是可用的。

```js
class Rectangle {
    constructor (length, width) {
        this.length = length
        this.width = width
    }

    getArea () {
        return this.length * this.width
    }

    static create (length, width) {
        return new Rectangle(length, width)
    }
}

class Square extends Rectangle {
    constructor (length) {

        // 与 Rectangle.call(this, length, length) 相同
        super(length, length)
    }
}

var rect = Square.create(3, 4)

console.log(rect instanceof Rectangle)      // true
console.log(rect.getArea())                 // 12
console.log(rect instanceof Square)         // false
```

在此代码中，一个新的静态方法`create()`被添加到`Rectangle`类中。通过继承，该方法会以`Square.create()`的形式存在，并且其行为方式与`Rectangle.create()`一样。

## 从表达式中派生类

在ES6中派生类的最强大能力，或许就是能够从表达式中派生类。只要一个表达式能够返回一个具有`[[Construct]]`属性以及原型的函数，你就可以对其使用`extends`。例如：

```js
function Rectangle (length, width) {
    this.length = length
    this.width = width
}

Rectangle.prototype.getArea = function () {
    return this.length * this.width
}

class Square extends Rectangle {
    constructor (length) {
        super(length, length)
    }
}

var x = new Square(3)
console.log(x.getArea())                // 9
console.log(x instanceof Rectangle)     // true
```

`Rectangle`被定义为ES5风格的构造器，而`Square`则是一个类。由于`Rectangle`具有`[[Construct]]`以及原型，`Square`类就能直接继承它。

`extends`后面能接受任意类型的表达式，这带来了巨大可能性，例如动态地决定所要继承的类：

```js
function Rectangle (length, width) {
    this.length = length
    this.width = width
}

Rectangle.prototype.getArea = function () {
    return this.length * this.width
}

function getBase () {
    return Rectangle
}

class Square extends getBase() {
    constructor(length) {
        super(length, length)
    }
}

var x = new Square(3)
console.log(x.getArea())                // 9
console.log(x instanceof Rectangle)     // true
```

## 继承内置对象

几乎从JS数组出现那天开始，开发者就想通过继承机制来创建他们自己的特殊数组类型。在ES5及早期版本中，这是不可能做到的。试图使用传统继承并不能产生功能正确的代码，例如：

```js
// 内置数组的行为
var colors = []
colors[0] = 'red'
console.log(colors.length)      // 1

colors.length = 0
console.log(colors[0])          // undefined

// 在 ES5 中尝试继承数组
function MyArray () {
    Array.apply(this, arguments)
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        value: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
})

var colors = new MyArray()
colors[0] = 'red'
console.log(colors.length)      // 0

colors.length = 0
console.log(colors[0])          // 'red'
```

`MyArray`实例上的`length`属性以及数值属性，其行为与内置数组并不一致，因为这些功能并未被涵盖在`Array.apply()`或数组原型中。

在ES6中的类，其设计目的之一就是允许从内置对象上进行继承。为了达成这个目的，类的继承模型与ES5或更早版本的传统继承模型有轻微差异：

在ES5的传统继承中，`this`的值会先被派生类（例如`MyArray`）创建，随后基类构造器（例如：`Array.apply()`方法）才被调用。这意味着`this`一开始就是`MyArray`的实例，之后才使用了`Array`的附加属性对其进行了装饰。

在ES6基于类的继承中，`this`的值会先被基类（`Array`）创建，随后才被派生类的构造器（`MyArray`）所修改。结果是`this`初始就拥有作为基类的内置对象的所有功能，并能正确接收与之关联的所有功能。

以下范例实际展示了基于类的特殊数组：

```js
class MyArray extends Array {
    // 空代码块
}

var colors = new MyArray()
colors[0] = 'red'
console.log(colors.length)      // 1

colors.length = 0
console.log(colors[0])          // undefined
```

`MyArray`直接继承了`Array`，因此工作方式与正规数组一致。与数值索引属性的互动更新了`length`属性，而操纵`length`属性也能更新索引属性。这意味着你既能适当地继承`Array`来创建你自己的派生类数组，也同样能继承其他的内置对象。

## Symbol.species属性

继承内置对象一个有趣的方面是：任意能返回内置对象实例的方法，在派生类上却会自动返回派生类的实例。因此，若你拥有一个继承了`Array`的派生类`MyArray`，诸如`slice()`之类的方法都会返回`MyArray`的实例。例如：

```js
class MyArray extends Array {
    // 空代码块
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3)

console.log(items instanceof MyArray)          // true
console.log(subitems instanceof MyArray)       // true
```

`Symbol.species`知名符号被用于定义一个能返回函数的**静态访问器属性**。每当类实例的方法（**构造器除外**）必须创建一个实例时，`Symbol.species`返回的函数就会被用为新实例的构造器。

```js
class MyClass {
    static get [Symbol.species] () {
        // 静态成员为类构造器的方法，因此这个this指向MyClass
        return this
    }

    constructor (value) {
        this.value = value
    }
    
    clone () {
        return new this.constructor[Symbol.species](this.value)
    }
}
```

在此例中，`Symbol.species`知名符号被用于定义`MyClass`的一个静态访问器属性。注意此处只有个`getter`而没有`setter`，这是因为修改类的`species`是不允许的。

`Array`使用了`Symbol.species`来指定方法所使用的的类，让其返回值为一个数组。在`Array`派生类中，你可以决定这些继承方法应返回何种类型的对象，正如：

```js
class MyArray extends Array {
    static get [Symbol.species] () {
        return Array
    }
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3)

console.log(items instanceof MyArray)          // true
console.log(subitems instanceof Array)         // true
console.log(subitems instanceof MyArray)       // false
```

# 在类构造器中使用new.target

在类的构造器中可以使用`new.target`来判断类是被如何调用的。

```js
class Rectangle {
    constructor (length, width) {
        console.log(new.target === Rectangle)
        this.length = length
        this.width = width
    }
}

class Square extends Rectangle {
    constructor (length) {
        super(length, length)
    }
}

// new.target 就是 Square
var obj = new Square(3)         // false
```

`Square`调用了`Rectangle`构造器，因此当`Rectangle`构造器被调用时，`new.target`等于`Square`。这很重要，因为构造器能根据如何被调用而有不同行为，并且这给了更改这种行为的能力。例如，你可以使用`new.target`来创建一个抽象基类（一个不能被实例化的类），如下：

```js
// 静态的基类
class Shape {
    constructor() {
        if (new.target === Shape) {
            throw new Error('This class cannot be instantiated directly.')
        }
    }
}

class Rectangle extends Shape{
    constructor (length, width) {
        super()
        this.length = length
        this.width = width
    }
}

var x = new Rectangle(3, 4)
console.log(x instanceof Shape)     // true

var y = new Shape()                 // Error: This class cannot be instantiated directly.
```

此例中的`Shape`类构造器会在`new.target`为`Shape`的时候抛出错误，意味着`new Shape()`永远都会抛出错误。然而，你依然可以将`Shape`用作一个基类，正如`Rectangle`所做的那样。`super()`的调用执行了`Shape`构造器，而且`new.target`的值等于`Rectangle`，因此该构造器能够无错误地继续执行。