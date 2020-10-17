---
title: 对象&原型&继承
categories:
    - JavaScript
abbrlink: bb5d40f2
date: 2019-03-08 22:46:34
---

# 理解对象

## 属性类型

### 数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入值。数据属性有 4 个描述其行为的特性。

{% note info %}
- `configurable`：表示能否通过 `delete` 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。直接在对象上定义的属性，这个特性默认值为 `true`；
- `enumerable`：表示能否通过 `for-in` 循环返回属性。直接在对象上定义的属性，这个特性默认值为 `true`；
- `writable`：表示能否修改属性的值。直接在对象上定义的属性，这个特性默认值为 `true`；
- `value`：包含这个属性的数据值。读取属性值得时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。这个特性的默认值为 `undefined`。
---
可以使用 `Object.defineProperty(obj, prop, descriptor)` 修改数据属性的特性：
- `obj`：数据属性所在的对象。
- `prop`：数据属性的名称，可以是字符串或符号。
- `descriptor`：描述符对象，可包含的属性有 `configurable`、`enumerable`、`writable` 和 `value`。

**如果通过 `Object.defineProperty(obj, prop, descriptor)` 定义一个新的数据属性，`descriptor` 中缺失的特性会被赋予 `false` 或 `undefined`。**
{% endnote %}

```js
var obj = { test: 1 }
var desc = Object.getOwnPropertyDescriptor(obj, 'test')

// { value: 1, writable: true, enumerable: true, configurable: true }
console.log(desc)

Object.defineProperty(obj, 'demo', {
    configurable: false
})
desc = Object.getOwnPropertyDescriptor(obj, 'demo')

// { value: undefined, writable: false, enumerable: false, configurable: false }
console.log(desc)
```

**关于writable**：当 `writable` 为 `false` 时，在非严格模式下通过赋值语句修改属性值，赋值操作将被忽略；在严格模式下则会抛出错误。**但是如果通过 Object.defineProperty() 方法修改 value 特性则不会有任何问题**。

```js
var obj = { test: 1}
Object.defineProperty(obj, 'test', {
    writable: false
})
obj.test = 2

// { test: 1 }
console.log(obj)       

Object.defineProperty(obj, 'test', {
    value: 3
})

// { test: 3 }
console.log(obj)       
```
**关于configurable**：当 `configurable` 为 `false` 时，不允许删除属性，不允许修改属性的 `enumerable`、`configurable`，不可以将 `writable` 由 `false` 修改为 `true`，但是可以将 `writable `由 `true` 修改为 `false`，也可以修改属性的 `value` 特性。

**当 writable 和 configurable 均为 false 时，不允许通过任何方式修改属性值，直接赋值或者通过 Object.defineProperty() 都不可以！**

```js
/**
 * 下面的实验运行于非严格模式下
 */
var obj = { test: 1 }

// 在 configurable 为 false 时尝试删除属性
Object.defineProperty(obj, 'test', {
    configurable: false
})
delete obj.test

// { test: 1 }
console.log(obj)        

var desc = Object.getOwnPropertyDescriptor(obj, 'test')

// { value: 1, writable: true, enumerable: true, configurable: false }
console.log(desc)       

// 在 configurable 为 false 时尝试修改 enumerable
// Uncaught TypeError: Cannot redefine property: test
Object.defineProperty(obj, 'test', {
  enumerable: false
})

// 在 configurable 为 false 时尝试修改 configurable
// Uncaught TypeError: Cannot redefine property: test
Object.defineProperty(obj, 'test', { 
  configurable: true
})

// 在 configurable 为 false 时尝试修改 value
Object.defineProperty(obj, 'test', {
  value: '此时 configurable 为 false'
})

// { test: "此时 configurable 为 false" }
console.log(obj)    

// 在 configurable 为 false 时尝试将 writable 由 true 修改为 false
Object.defineProperty(obj, 'test', {
  writable: false
})
var desc = Object.getOwnPropertyDescriptor(obj, 'test')

// { value: "此时 configurable 为 false", writable: false, enumerable: true, configurable: false }
console.log(desc)  

// 在 configurable 为 false 时尝试将 writable 由 false 修改为 true
// Uncaught TypeError: Cannot redefine property: test
Object.defineProperty(obj, 'test', { 
  writable: true
})

// 在 configurable 和 writable 均为 false时，尝试修改属性值
obj.test = '直接赋值可以吗'
// {test: "此时configurable为false"}
console.log(obj)       
// Uncaught TypeError: Cannot redefine property: test
Object.defineProperty(obj, 'test', {
  value: '通过 Object.defineProperty 可以吗'
})
```

### 访问器属性

访问器属性不包含数据值；它们包含一对 `get` 和 `set` 函数（不过，这两个函数都不是必需的）。在读取访问器属性时，会调用 `get` 函数，这个函数负责返回有效的值；在写入访问器属性时，会调用 `set` 函数并传入新值，这个函数负责决定如何处理数据。访问器属性有如下4个特性。

{% note info %}
- `configurable`：表示能否通过`delete`删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。直接在对象上定义的属性，它们的这个特性默认值为`true`；
- `enumerable`：表示能否通过`for-in`循环返回属性。直接在对象上定义的属性，它们的这个特性默认值为`true`；
- `get`：在读取属性时调用的函数。默认值为`undefined`；
- `set`：在写入属性时调用的函数。默认值为`undefined`。
{% endnote %}

**访问器属性不能直接定义，必须使用Object.defineProperty()来定义。**

```js
var book = {
  _year: 2004,
  edition: 1
}
Object.defineProperty(book, 'year', {
  get: function () {
    return _this.year
  },
  set: function (newValue) {
    if (newValue > 2004) {
      this._year = newValue
      this.edition += newValue - 2004
    }
  }
})
book.year = 2005

// { _year: 2005, edition: 2 }
console.log(book)   
```

不一定非要同时指定 `get `和 `set`。只指定 `get` 意味着属性是不能写，尝试写入属性会被忽略。在严格模式下，尝试写入只指定了 `get` 函数的属性会抛出错误。而读取只指定 `set` 的属性会返回 `undefined`

**可以通过 Object.defineProperty() 实现数据属性与访问器属性的转换，但是切记不能同时指定数据属性和访问器属性，这样会抛出错误！**

## 定义多个属性

ES5 定义了一个 `Object.defineProperties()` 方法用来为对象定义多个属性。

```js
var book = {}
Object.defineProperties(book, {
  _year: {
    value
  },
  edition: {
    value: 1
  },
  year: {
    get: function () {
      return _this.year
    },
    set: function (newValue) {
      if (newValue > 2004) {
        this._year = newValue
        this.edition += newValue - 2004
      }
    }
  }
})
```

## 读取属性的特性

使用ES5的 `Object.getOwnPropertyDescriptor()` 方法可以取得给定属性的描述符。该方法接收两个参数：属性所在的对象和要读取其描述符的属性名称。返回值是一个对象。**这个方法只能用于实例属性，要取得原型属性的描述符，必须直接在原型对象上调用。**

# 创建对象

## 工厂模式

工厂模式就是调用函数返回一个包含特定属性和方法的对象，工厂模式的问题在于**它没有解决对象识别的问题（即怎样知道一个对象的类型）**。

```js
function createPerson (name, age) {
  var o = {
    name: name,
    age: age,
    sayName: function () {
      console.log(this.name)
    }
  }
  return o
}
var person1 = createPerson('Nicholas', 29)
var person2 = createPerson('Greg', 27)
```

## 构造函数模式

ES 中可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法，下面使用构造函数模式重写工厂模式中的例子。

```js
function Person (name, age) {
  this.name = name
  this.age = age
  this.sayName = function () {
    console.log(this.name)
  }
}
var person1 = new Person('Nicholas', 29)
var person2 = new Person('Greg', 27)

// true
console.log(person1 instanceof Person)  
// true
console.log(person2 instanceof Person)  
```

在 `person1` 和 `person2` 中分别保存着 `Person` 的不同实例，这两个对象都有一个 `constructor` 属性，该属性指向 `Person`。对象的 `constructor` 属性最初是用来标识对象类型的，但是提到检测对象类型，还是 `instanceof` 操作符更可靠一些。创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类型；而这正是构造函数模式胜过工厂模式的地方。

**构造函数模式与工厂模式的区别**：

{% note info %}
- 没有显示地创建对象.
- 直接将属性和方法赋给了 `this` 对象.
- 没有 `return` 语句。
{% endnote %}

**要创建 `Person` 的新实例，必须使用 `new` 操作符**。以这种方式调用构造函数实际上会经历一下 4 个步骤：

{% note info %}
1. 创建一个新对象（因为用了 `new`）。
2. 为新对象连接原型。
3. 将构造函数的作用域内的 `this` 绑定到这个新对象。
4. 执行构造函数的代码。
5. 返回新对象。 
{% endnote %}

{% note warning %}
关于使用 `new` 创建新对象的时候，如果存在类的继承，那么在 ES5 和 ES6 中这个过程是有差别的。[**查看详情**](https://aadonkeyz.com/posts/e1a12c4d/#继承内置对象)
{% endnote %}

构造函数与其他函数的唯一区别，就在于调用它们的方式不同。不过，构造函数毕竟也是函数，不存在定义构造函数的特殊语法。任何函数，只要通过 `new` 操作符来调用，那么它就可以作为构造函数。

构造函数模式虽然好用，但也并非没有缺点。使用构造函数的主要问题就是每个方法都要在每个实例上重新创建一遍，但是有的方法是所有实例都应该共享的，没有创建多次的必要。

## 原型模式

我们创建的每一个函数都有一个 `prototype`（原型）属性，这个属性值是一个对象的引用，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。

### 理解原型对象

**无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个 `prototype` 属性，这个属性指向函数的原型对象。**在默认情况下，所有原型对象都会自动获得一个 `constructor`（构造函数）属性，这个属性包含一个指向 `prototype` 属性所在函数的引用。

```js
function Person () {}
Person.prototype.name = 'Nicholas'
Person.prototype.age = 29
Person.prototype.job = 'Software Engineer'
Person.prototype.sayName = function () {
  console.log(this.name)
}

var person1 = new Person()

// Nicholas
person1.sayName()   

var person2 = new Person()

// Nicholas
person2.sayName()   

// true
console.log(person1.sayName === person2.sayName)    
```

![理解原型对象](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E7%90%86%E8%A7%A3%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1.png)

上图展示了 `Person` 构造函数、`Person` 的原型属性以及 `Person` 现有的两个实例之间的关系。注意 `Person` 的每个实例，`person1` 和 `person2` 都包含一个内部属性 `[[Prototype]]`，该属性仅仅指向了 `Person.prototype`；换句话说，**对象实例与构造函数没有直接的关系**。

{% note info %}
- `isPrototypeOf()`：用于测试一个对象是否存在于另一个对象的原型链上；
- `hasOwnProperty()`：用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在，该方法是从 `Object` 继承而来的；
- `Object.getPrototypeOf()`：返回指定对象的原型（对象内部 `[[Prototype]]` 属性的值）。
{% endnote %}

每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。搜索首先从对象实例开始，如果在实例中找到了具有给定名字的属性，则返回该属性；如果没有找到，则沿着对象的原型链向上逐层查找具有给定名字的属性，如果找到了则返回这个属性的值。

### 属性设置和屏蔽

当**用赋值语句**给实例对象设置已经在原型链上层存在的同名属性时，会有以下三种情况：

{% note info %}
- 如果在原型链上层存在的同名属性没有被标记为只读，即 `writable: true`，那么会直接在实例中添加一个同名的新属性，它是屏蔽属性；
- 如果在原型链上层存在的同名属性被标记为只读，即 `writable: false`，那么无法修改已有属性，也无法在实例对象上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。如果在非严格模式下，赋值语句会被忽略；
- 如果在原型链上层存在的同名属性具有 `set` 描述符，那么一定会调用这个 `set`。实例对象上并不会添加新的属性，也不会重新定义这个 `set`。
{% endnote %}

**但是 JS 这门语言就是贼灵活，如果上述所说的存在于原型链上层的同名属性中保存的是某一个引用类型值的引用，那么你还是可以修改这个引用类型的值的（并没有违反规则，因为保存的引用并没有改变）！比如，这个属性保存的是某一个数组的引用，那么我就可以通过 `push` 方法去改变这个数组。**

**如果你无论如何也想要屏蔽原型链上层的属性，那么你可以使用 `Object.defineProperty()` 方法！**

有些情况下会隐式产生屏蔽，一定要当心。思考下面的代码：

```js
var anotherObject = { a: 2 }
var myObject = Object.create(anotherObject)

// 2
anotherObject.a     

// 2
myObject.a          

// true
anotherObject.hasOwnProperty('a')   

// false
myObject.hasOwnProperty('a')        

// 隐式屏蔽！
myObject.a++    

// 2
anotherObject.a     

// 3
myObject.a          

// true
myObject.hasOwnProperty('a')        
```

### 属性的获取

有两种方式使用 `in` 操作符：单独使用和在 `for-in` 循环中使用。

在单独使用时，`in` 操作符会在通过对象能够访问给定属性时返回 `true`，无论该属性存在于实例中还是原型中。

在使用 `for-in` 循环时返回的是所有能够通过对象访问的、可枚举的属性，其中既包括存在于实例中的属性，也包含存在于原型中的属性。屏蔽了原型中不可枚举属性的实例属性也会在 `for-in` 循环中返回。

要取得对象上所有可枚举的实例属性，可以使用 ES5 的 `Object.keys()` 方法。这个方法接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组。

如果你想要得到所有实例属性，无论它是否可枚举，可以使用 `Object.getOwnPropertyNames()` 方法。

### 更简单的原型语法

简单来说就是用对象字面量形式来重写 `Person.prototype`，但是这样会导致新原型对象的 `constructor` 属性指向 `Object` 而不是 `Person`，尽管此时 `instanceof` 操作符还能返回正确的结果，但是通过 `constructor` 已经无法确定对象的类型了，所以如果 `constructor` 属性比较重要的话，还需要用 `Object.defineProperty()` 方法定义 `constructor` 的数据属性。

```js
function Person () {}

Person.prototype = {
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName: function () {
    console.log(this.name)
  }
}

var friend = new Person()

// true
console.log(friend instanceof Object)       

// true
console.log(friend instanceof Person)       

// true
console.log(friend.constructor === Object)  

// false
console.log(friend.constructor === Person)  

Object.defineProperty(Person.prototype, 'constructor', {
  enumerable: false,
  value: Person
})
```

### 原型的动态性

由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来————即使是**先创建了实例后修改原型**也是如此。

```js
function Person () {}
var friend = new Person ()
Person.prototype.sayHi = function () {
  console.log('hi')
}

// hi
friend.sayHi()      
```

但是如果**先创建了实例然后重写整个原型对象**，那么情况就不一样了。具体的变化看图吧!

![重写原型对象](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E9%87%8D%E5%86%99%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1.png)

**此时 `instanceof` 操作符已经不好使了！**
**构造函数找不到最初的原型对象了！**
**现有实例也找不到新的原型对象了！**

```js
function Person () {}

var friend = new Person()

Person.prototype = {
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName: function () {
    console.log(this.name)
  }
}
Object.defineProperty(Person.prototype, 'constructor', {
  enumerable: false,
  value: Person
})

// true
console.log(friend instanceof Object)       

// false
console.log(friend instanceof Person)       

// Uncaught TypeError: friend.sayName is not a function
friend.sayName()              
```

### 原型对象的问题

原型模式也不是没有缺点。首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值。虽然这会在某种程度上带来一些不方便，但还不是原型模式的最大问题。原型模式的最大问题是由其共享的本性所导致的（主要针对引用类型值的属性来说）。

## 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存，另外这种混成模式还支持向构造函数传递参数。

```js
function Person (name, age, job) {
  this.name = name
  this.age = age
  this.job = job
  this.friends = ['Shelby', 'Court']
}
Person.prototype = {
  constructor: Person,
  sayName: function () {
    console.log(this.name)
  }
}
var person1 = new Person('Nicholas', 29, 'Software Engineer')
var person2 = new Person('Greg', 27, 'Doctor')
person1.friends.push('Van')

// ["Shelby", "Court", "Van"]
console.log(person1.friends)  

// ["Shelby", "Court"]
console.log(person2.friends)  

// false
console.log(person1.friends === person2.friends)  

// true
console.log(person1.sayName === person2.sayName)  
```

## 动态原型模式

动态原型模式把所有信息封装在了构造函数中，而通过在构造函数中初始化原型（仅在必要的情况下），保持了同时使用构造函数和原型的优点。换句话说，可以通过检查某个应该存在的方法是否有效来决定是否需要初始化原型。

```js
function Person (name, age, job) {
  // 属性
  this.name = name
  this.age = age
  this.job = job

  // 方法
  if (typeof this.sayName !== 'function') {
    Person.prototype.sayName = function () {
        console.log(this.name)
    }
  }
}

var friend = new Person('Nicholas', 29, 'Software Engineer')

// Nicholas
friend.sayName()    
```

**在使用动态原型模式时，禁止使用对象字面量重写原型！**

## 寄生构造函数模式

在前面几种模式都不适用的情况下，可以使用寄生构造函数模式，这种模式的基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象。

```js
function Person (name, age, job) {
  var o = new Object()
  o.name = name
  o.age = age
  o.job = job
  o.sayName = function () {
    console.log(this.name)
  }
  return o
}

var friend = new Person('Nicholas', 29, 'Software Engineer')

// Nicholas
friend.sayName()    
```

在这个例子中，`Person` 函数创建了一个新对象，并以相应的属性和方法初始化该对象，然后又返回了这个对象。除了使用 `new` 操作符并把使用的包装函数叫做构造函数之外，这个模式跟工厂模式其实是一模一样的。**构造函数在不返回值的情况下，默认会返回新对象实例。而通过在构造函数的末尾添加一个 `return` 语句，可以重写调用构造函数时返回的值。**

关于寄生构造函数模式，有一点需要说明：返回的对象与构造函数或者与构造函数的原型属性之间没有关系；也就是说，构造函数返回的对象与在构造函数外部创建的对象没有什么不同。为此，不能依赖 `instanceof` 操作符来确定对象类型。由于存在上述问题，我们建议在可以使用其他模式的情况下，不要使用这种模式。

## 稳妥构造函数模式

道格拉斯·克罗克福德发明了JavaScript中的稳妥对象这个概念。所谓稳妥对象，指的是没有公共属性，而且其方法也不引用 `this` 的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用 `this` 和 `new`），或者在防止数据被其他应用程序改动时使用。稳妥构造函数遵循与计生构造函数类似的模式，但有两点不同：一是新创建对象的实例方法不引用 `this`；二是不使用 `new` 操作符调用构造函数。

```js
function Person (name, age, job) {
  // 创建要返回的对象
  var o = new Object()

  // 可以在这里定义私有变量和函数

  // 添加方法
  o.sayName = function () {
    console.log(name)
  }
  return o
}

var friend = Person('Nicholas', 29, 'Software Engineer')

// Nicholas
friend.sayName()    
```

## 小结

![创建对象模式总结](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%BC%8F%E6%80%BB%E7%BB%93.png)

# 继承

## 原型链

```js
function SuperType () {
  this.property = true
}
SuperType.prototype.getSuperValue = function () {
  return this.property
}

function SubType () {
  this.subproperty = false
}
SubType.prototype = new SuperType()
SubType.prototype.getSubValue = function () {
  return this.subproperty
}

var instance = new SubType()

// true
console.log(instance.getSuperValue())   
```

我觉得用文字解释这个原型链有点绕嘴，没有上图方便，就直接看下面的图片吧！

![原型链](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E5%8E%9F%E5%9E%8B%E9%93%BE.png)

{% note info %}
- `instanceof` 操作符用于测试构造函数的 `prototype` 属性是否出现在对象的原型链中
- `isPrototypeOf()` 方法用于测试一个对象是否存在于另一个对象的原型链上
- 子类型有时候需要重写超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但不管怎样，给原型添加方法的代码一定要放在替换原型的语句之后；
- 在通过原型链实现继承时，不能使用对象字面量语法重写原型。
{% endnote %}

原型链的第一个问题类似于上面介绍的原型模式的问题，这里就不详细介绍了。
它的第二个问题是在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。

## 借用构造函数

在解决原型中包含引用类型值所带来问题的过程中，开发人员开始使用一种叫做借用构造函数的技术。这种技术的基本思想相当简单，即在子类型构造函数的内部调用超类型构造函数（通过 `call()` 或 `apply()` 方法）。

```js
function SuperType (name) {
  this.name = name
}

function SubType () {
  // 继承了SuperType，同时还传递了参数
  SuperType.call(this, 'Nicholas')

  // 实例属性
  this.age = 29
}

var instance = new SubType()

// Nicholas
console.log(instance.name)      

// 29
console.log(instance.age)       
```

如果仅仅是借用构造函数，那么也将无法避免构造函数模式存在的问题————方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式。

## 组合继承

组合继承有时候也叫做伪经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。

```js
function SuperType (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function () {
  console.log(this.name)
}

function SubType (name, age) {
  // 继承属性
  SuperType.call(this, name)

  this.age = age
}
// 继承方法
SubType.prototype = new SuperType()
SubType.prototype.sayAge = function () {
  console.log(this.age)
}

var instance1 = new SubType('Nicholas', 29)
instance1.colors.push('black')

// ["red", "blue", "green", "black"]
console.log(instance1.colors)       

// Nicholas
instance1.sayName()                 

// 29
instance1.sayAge()                  

var instance2 = new SubType('Greg', 27)

// ["red", "blue", "green"]
console.log(instance2.colors)       

// Greg
instance2.sayName()                 

// 27
instance2.sayAge()                  
```

两个实例上的`colors`属性屏蔽了原型链上的同名属性。

## 原型式继承

```js
function object (o) {
  function F () {}
  F.prototype = o
  return new F()
}
```

在 `object()` 函数内部，先创建了一个临时性的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回了这个临时类型的一个新实例。从本质讲，`object()` 对传入其中的对象执行了一次浅复制。

ES5新增的 `Object.create()` 方法规范化了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。在传入一个参数的情况下，`Object.create()` 与 `object()` 方法行为相同。不过到目前为止存在一个例外，**当传入 `null` 作为参数时，`Object.create()` 与 `object()` 行为不相同！**

`Object.create()` 方法的第二个参数与 `Object.defineProperties()` 方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式制定的任何属性都会覆盖原型链上的同名属性。

```js
var person = {
  name: 'Nicholas',
  friengd: ['Shelby', 'Court', 'Van']
}
var anotherPerson = Object.create(person, {
  name: {
    value: 'Greg'
  }
})

// Greg
console.log(anotherPerson.name)     
```

## 寄生式继承

寄生式继承是与原型式继承紧密相关的一种思路，寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象，最后再像真地是它做了所有工作一样返回对象。

```js
function object (o) {
  function F () {}
  F.prototype = o
  return new F()
}

function createAnother (original) {
  // 通过调用函数创建一个新对象
  var clone = object(original)    

  // 以某种方式来增强这个对象
  clone.sayHi = function () {     
    console.log('hi')
  }

  // 返回这个对象
  return clone                    
}

var person = {
  name: 'Nicholas',
  friengd: ['Shelby', 'Court', 'Van']
}
var anotherPerson = createAnother(person)

// hi
anotherPerson.sayHi()       
```

## 寄生组合式继承

前面说过组合继承是 JavaScript 最常用的继承函数，不过它也有自己的不足。组合继承最大的问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。没错，子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子类型构造函数时重写这些属性。

```js
function SuperType (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function () {
  console.log(this.name)
}

function SubType (name, age) {
  // 第二次调用SuperType()
  SuperType.call(this, name)              

  this.age = age
}

// 第一次调用SuperType()
SubType.prototype = new SuperType()        
SubType.prototype.sayAge = function () {
  console.log(this.age)
}
```

为了解决上述问题，我们使用寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背后的基本思路是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。

```js
function object (o) {
  function F () {}
  F.prototype = o
  return new F()
}

function inheritPrototype (subType, superType) {
  // 创建对象
  var prototype = object(superType.prototype)     

  // 增强对象
  prototype.constructor = subType                 

  // 指定原型
  subType.prototype = prototype                   
}

function SuperType (name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function () {
  console.log(this.name)
}

function SubType (name, age) {
  SuperType.call(this, name)

  this.age = age
}

inheritPrototype(SubType, SuperType)

SubType.prototype.sayAge = function () {
  console.log(this.age)
}
```

这个例子的高效率体现在它只调用了一次 `SuperType` 构造函数，并因此避免了在 `SubType.prototype` 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 `instanceof` 和 `isPrototypeOf()`。

## 小结

![继承总结](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/JavaScript/%E7%BB%A7%E6%89%BF%E6%80%BB%E7%BB%93.png)