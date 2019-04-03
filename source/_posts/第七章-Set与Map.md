---
title: 第七章 Set与Map
categories:
  - JavaScript
  - 《深入理解ES6》
abbrlink: d766545b
date: 2019-04-01 21:03:23
---

`Set`是不包含重复值的列表。你一般不会像对待数组那样来访问`Set`中的某个项；相反更常见的是，只在`Set`中检查某个值是否存在。

`Map`则是键与相对应的值的集合。因此，`Map`中的每个项都存储了两块数据，通过指定所需读取的键即可检索对应的值。`Map`常被用作缓存，存储数据以便此后快速检索。

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

造成上面例子中问题的原因是对象属性只可以是字符串或符号类型，所以当你为对象定义属性时，如果属性名不是字符串或符号类型，那么它会调用属性名的`toString()`方法将属性名转换为字符串，然后再进行后续操作。所以`map[key1] === map[key2]`、`map[key3] === map[key4]`。

# ES6的Set

ES6新增了`Set`类型，这是一种无重复值的有序列表。`Set`允许对它包含的数据进行快速访问，从而增加了一个追踪离散值的更有效方式。

## 创建Set并添加项目

`Set`使用`new Set()`来创建，而调用`add()`方法就能向`Set`中添加项目，检查`size`属性还能查看其中包含有多少项。

```js
let set = new Set()

set.add(5)
set.add('5')

console.log(set.size)   // 2
```

`Set`不会使用强制类型转换来判断值是否重复。这意味着`Set`可以同时包含数值`5`与字符串`"5"`，将它们都作为相对独立的项（在`Set`内部的比较使用了`Object.is()`方法来判断两个值是否相等，唯一的例外是+0与-0被判断为是相等的）。你还可以向`Set`添加多个对象，它们不会被合并为同一项：

```js
let set = new Set(),
    key1 = {},
    key2 = {}

set.add(key1)
set.add(key2)

console.log(set.size)   // 2
```

由于`key1`与`key2`并不会被转换为字符串，所以它们在这个`Set`内部被认为是两个不同的项（记住：如果它们被转换为字符串，那么都会等于`"[object Object]"`）。

如果`add()`方法用相同值进行了多次调用，那么在第一次之后的调用实际上会被忽略：

```js
let set = new Set()

set.add(5)
set.add('5')
set.add(5)      // 重复了，该调用被忽略

console.log(set.size)   // 2
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

Set上的`forEach()`方法类似于数组中的`forEach()`方法，它接收两个参数：要在每一项上运行的函数和（可选的）运行该函数时的`this`值。传入的函数会接收三个参数：Set元素的值，Set元素的值和目标Set自身。你没看错，第一个和第二个参数是完全相同的，这么设计的目的是为了保持所有对象上`forEach()`方法的统一。

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

由于`Set`类型存储对象引用的方式，它也可以被称为Strong Set。对象存储在`Set`的一个实例中时，实际上相当于把对象存储在变量中。只要对`Set`实例的引用仍然存在，所存储的对象就无法被垃圾回收机制回收，从而无法释放内存。例如：

```js
let set = new Set(),
    key = {}

set.add(key)
console.log(set.size)       // 1

// 取消原始引用
key = null

console.log(set.size)       // 1

// 重新获得原始引用
key = [...set][0]
```

在本例中，将`key`设置为`null`清楚了对`key`对象的一个引用，但是另一个引用还存在于`set`内部。你仍然可以使用扩展运算符将`Set`转换为数组，然后访问数组的第一项，`key`变量就取回了原先的对象。

为了缓解这个问题，**ES6也包含了Weak Set，该类型只允许存储对象弱引用，而不能存储基本类型的值。对象的弱引用在它自己成为该对象的唯一引用时，不会阻止垃圾回收。**

### 创建Weak Set

Weak Set使用`WeakSet`构造器来创建，并包含`add()`方法、`has()`方法以及`delete()`方法。以下例子使用了这三个方法：

```js
let set = new WeakSet(),
    key = {}

set.add(key)

console.log(set.has(key))       // true

set.delete(key)

console.log(set.has(key))       // false
```

使用Weak Set很像在使用正规的`Set`。你可以在Weak Set上添加、移除或检查引用，也可以给构造器传入一个可迭代对象来初始化Weak Set的值：

```js
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2])

console.log(set.has(key1))      // true
console.log(set.has(key2))      // true
```

在本例中，一个数组被传给了`WeakSet`构造器。由于该数组包含了两个对象，这些对象就被添加到了Weak Set中。要记住若数组中包含了非对象的值，就会抛出错误，因为`WeakSet`构造器不接受基本类型的值。

### Set类型之间的关键差异

Weak Set与正规Set之间最大的区别是对象的弱引用。此处有个例子说明了这种差异：

```js
let set = new WeakSet(),
    key = {}

set.add(key)

console.log(set.has(key))       // true

// 移除对于键的最后一个强引用，同时从Weak Set中移除
key = null
```

当代码被执行后，Weak Set中的`key`引用就不能再访问了。核实这一点是不可能的，因为需要把对于该对象的一个引用传递给`has()`方法（而只要存在其他引用，Weak Set内部的弱引用就不会消失）。这会使得难以对Weak Set的引用特征进行测试，但JS引擎已经正确地将引用移除了，这一点你可以信任。

这些例子演示了Weak Set与正规Set的一些共有特征，但是它们还有一些关键的差异，即：

{% note info %}
- 对于`WeakSet`的实例，若调用`add()`方法时传入非对象的参数，就会抛出错误（`has()`或`delete()`则会在传入了非对象的参数时返回`false`）；
- Weak Set不可迭代，因此不能被用在`for-of`循环中；
- Weak Set无法暴露出任何迭代器，因此没有任何编程手段可用于判断Weak Set的内容；
- Weak Set没有`forEach()`方法；
- Weak Set没有`clear()`方法；
- Weak Set没有`size`属性。
{% endnote %}

# ES6的Map

ES6的`Map`类型是键值对的有序列表，而键和值都可以是任意类型。

## 创建Map

`Map`对键的处理方式与`Set`对值的处理一样，采用的是`Object.is()`方法，而不会进行强制类型转换，唯一的例外是+0与-0被判断为是相等的。

你可以调用`set()`方法并给它传递一个键与一个关联的值，来给Map添加项；此后使用键名来调用`get()`方法便能提取对应的值。如果任意一个键不存在于Map中，则`get()`方法就会返回特殊值`undefined`。例如：

```js
let map = new Map(),
    key1 = {},
    key2 = {}

map.set(key1, 5)
map.set(key2, 42)

console.log(map.get(key1))      // 5
console.log(map.get(key2))      // 42
console.log(map.get(111))       // undefined
```

此代码使用了对象`key1`与`key2`作为Map的键，并存储了两个不同的值。由于这些键不会被强制转换成其他形式，每个对象就都被认为是唯一的。这允许你给对象关联额外数据，而无须修改对象自身。

与Set类似，你能将数组传递给`Map`构造器，以便使用数据来初始化一个`Map`。该数组中的每一项也必须是数组，内部数组的首个项会作为键，第二项则为对应值。因此整个Map就被这些双项数组所填充。例如：

```js
let map = new Map([['name', 'Nicholas'], ['age', 25]])

console.log(map.get('name'))    // Nicholas
console.log(map.get('age'))     // 25
```

虽然有数组构成的数组看起来有点奇怪，但这对于准确表示键来说却是必要的；因为键允许是任意数据类型，将键存储在数组中，是确保它们在被添加到Map之前不会被强制转换为其他类型的唯一方法。

## Map的方法及属性

{% note info %}
- **`has(key)`方法**：判断指定的键是否存在于Map中；
- **`delete(key)`方法**：移除Map中的键以及对应的值；
- **`clear()`方法**：移除Map中所有的键与值；
- **`size`属性**：用于指示包含了多少个键值对。
{% endnote %}

## Map上的forEach方法

Map的`forEach()`方法类似于Set与数组上的`forEach()`方法，不过它是按照键值对被添加到Map中的顺序来迭代的。它接收两个参数：要在每一项上运行的函数和（可选的）运行该函数时的`this`值。传入的函数会接收三个参数：Map项的值、该值所对应的的键和目标Map自身。

## Weak Map

### 使用Weak Map

ES6的`WeakMap`类型是键值对的无序列表，其中键必须是非空的对象，值则允许是任意类型。`WeakMap`的接口与`Map`的非常相似，都使用`set()`与`get()`方法来分别添加与提取数据。

类似于Weak Set，它没有`size`属性，没有任何办法可以确定Weak Map是否为空。在其他引用被移除后，由于对键的引用不再有残留，也就无法调用`get()`方法来去取对应的值。Weak Map已经切断了对于该值的访问，其所占的内存在垃圾回收器运行时便会被释放。

### Weak Map的初始化

为了初始化Weak Map，需要把一个由数组构成的数组传递给`WeakMap`构造器。就像正规Map那样，每个内部数组都应当有两个项，第一项是作为键的对象，第二项则是对应的值（任意类型）。例如：

```js
let key1 = {},
    key2 = {},
    map = new WeakMap([[key1, 'hello'], [key2, 42]])

console.log(map.has(key1))      // true
console.log(map.get(key1))      // hello
console.log(map.has(key2))      // true
console.log(map.get(key2))      // 42
```

对象`key1`与`key2`被用作Weak Map的键，`get()`与`has()`方法则能访问他们。在传递给`WeakMap`构造器的参数中，若任意键值对使用了非对象的键，构造器就会抛出错误。

### Weak Map的方法

Weak Map只有两个附加方法能用来与键值对交互。`has()`方法用于判断指定的键是否存在于Map中，而`delete()`方法则用于移除一个特定的键值对。`clear()`方法不存在，这是因为没有必要对键进行枚举，并且枚举Weak Map也是不可能的，这与Weak Set相同。

### 对象的私有数据

ES5中的创建私有数据的方式：

```js
var Person = (function () {
    var privateData = {},
        privateId = 0

    function Person (name) {
        Object.defineProperty(this, '_id', {
            value: privateId++
        })

        privateData[this._id] = {
            name: name
        }
    }

    Person.prototype.getName = function () {
        return privateData[this._id].name
    }

    return Person
})()
```

此例用IIFE包裹了`Person`的定义，其中含有两个私有属性：`privateData`和`privateId`。`privateData`对象存储了每个实例的私有信息，而`privateId`则被用于为每个实例产生一个唯一ID。当`Person`构造器被调用时，一个不可枚举、不可配置、不可写入的`_id`属性就被添加了。

接下来在`privateData`对象中建立了与实例ID对应的一个入口，其中存储着`name`的值。随后在`getName()`函数中，就能使用`this._id`作为`privateData`的键来提取该值。由于`privateData`无法从IIFE外部进行访问，实际的数据就是安全的，尽管`this._id`在`privateData`对象上依然是公开暴露的。

此方式的最大问题在于`privateData`中的数据永远不会消失，因为在对象实例被销毁时没有任何方法可以获知该数据，`privateData`对象就将永远包含多余的数据。这个问题现在可以换用Weak Map来解决了，如下：

```js
let Person = (function () {
    let privateData = new WeakMap()

    function Person (name) {
        privateData.set(this, { name: name })
    }

    Person.prototype.getName = function () {
        return privateData.get(this).name
    }

    return Person
})()
```

此版本的`Person`范例使用了Weak Map而不是对象来保存私有数据。由于`Person`对象的实例本身能被作为键来使用，于是也就无须再记录单独的ID。当`Person`构造器被调用时，将`this`作为键在Weak Map上建立了一个入口，而包含私有信息的对象成为了对应的值，其中只存放了`name`属性。通过将`this`传递给`privateData.get()`方法，以获取值对象并访问其`name`属性，`getName()`函数便能提取私有信息。这种技术让私有信息能够保持私有状态，并且当与之关联的对象实例被销毁时，私有信息也会被同时销毁。
