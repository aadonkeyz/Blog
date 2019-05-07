---
title: 第二十章 JSON
categories:
    - JavaScript
    - 《JavaScript高级程序设计》
abbrlink: fdc54b8c
date: 2019-04-27 10:03:27
---

# 语法

关于JSON，最重要的是理解它是一个数据格式，不是一种编程语言。虽然具有相同的语法形式，但JSON并不从属于JavaScript。而且，并不是只有JavaScript才使用JSON，毕竟JSON只是一种数据格式。

{% note info %}
JSON的语法可以表示以下三种类型的值：
- **简单值**：使用与JavaScript相同的语法，可以在JSON中表示字符串、数值、布尔值和`null`。但JSON不支持JavaScript中的特殊值`undefined`；
- **对象**：对象作为一种复杂数据类型，表示的是一组有序的键值对儿。而每个键值对儿中的值可以是简单值，也可以是复杂数据类型的值；
- **数组**：数组也是一种复杂数据类型，表示一组有序的值的列表。可以通过数值索引来访问其中的值。数组的值也可以是任意类型——简单值、对象或数组。

**JSON中的字符串必须使用双引号。**
{% endnote %}

举个简单的例子：

```json
{
    "name": "json demo",
    "number": 1,
    "array": [
        1,
        2,
        {
            "name": "obj"
        }
    ],
    "obj": {
        "name": "whatever"
    }
}
```

# JSON.stringify()

{% note info %}
`JSON.stringify()`接受三个参数：
- **value**：需要被转换为JSON字符串的值。
- **replacer（可选）**：该参数可以是一个数组或者过滤器函数。
    1. 当该参数为数组时，这个数组相当于是一个白名单，不在白名单中的属性将被过滤掉；
    2. 当该参数为函数时，这个函数接收两个参数：属性（键）名和属性值。这个函数的返回值就相当于对应的属性被序列化后的结果，但是如果函数返回了`undefined`，那么对应属性将被过滤掉。
- **space（可选）**：该参数可以是数值或字符串。当为数值时，这个数值表示序列化结果的缩进空格数；当为字符串时，这个字符串会替换空格，被用作缩进字符。

---
注意事项：
- 如果`value`中出现循环引用会抛出错误；
- 如果`value`是对象且没有传入`replacer`参数，那么如果它的某一个属性值是JSON所不支持的，比如`undefined`，那么这个属性会被自动过滤掉；
- 如果`value`是数组且没有传入`replacer`参数，那么如果它的某一项是JSON所不支持的，比如`undefined`，那么这个项会被转换为`null`；
- `replacer`是个十分有意思的函数，它接收到的第一个键会是空字符串`''`，而接收到的第一个值会是`value`。之后如果`value`是对象或者数组，它才开始在`value`中进行迭代。

---
有时候`JSON.stringify()`还是不能满足对某些对象进行自定义序列化的需求，这种情况下，就可以在对象中定义`toJson()`方法。**当`JSON.stringify()`接收的第一个参数包含`toJson()`方法时，会先调用其`toJson()`方法，然后再对该方法的返回值进行序列化，即实际上的操作是`JSON.stringify(value.toJson())`**。
{% endnote %}

下面的例子来自[**MDN**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

```js
JSON.stringify({})                    // '{}'
JSON.stringify(true)                  // 'true'
JSON.stringify('foo')                 // '"foo"'
JSON.stringify([1, 'false', false])   // '[1,"false",false]'
JSON.stringify([NaN, null, Infinity]) // '[null,null,null]'
JSON.stringify({ x: 5 })              // '{"x":5}'

JSON.stringify(new Date(2006, 0, 2, 15, 4, 5)) 
// '"2006-01-02T15:04:05.000Z"'

JSON.stringify({ x: 5, y: 6 })
// '{"x":5,"y":6}'
JSON.stringify([new Number(3), new String('false'), new Boolean(false)])
// '[3,"false",false]'

// String-keyed array elements are not enumerable and make no sense in JSON
let a = ['foo', 'bar']
a['baz'] = 'quux'      // a: [ 0: 'foo', 1: 'bar', baz: 'quux' ]
JSON.stringify(a) 
// '["foo","bar"]'

JSON.stringify({ x: [10, undefined, function(){}, Symbol('')] }) 
// '{"x":[10,null,null,null]}' 

// Standard data structures
JSON.stringify([new Set([1]), new Map([[1, 2]]), new WeakSet([{a: 1}]), new WeakMap([[{a: 1}, 2]])])
// '[{},{},{},{}]'

// TypedArray
JSON.stringify([new Int8Array([1]), new Int16Array([1]), new Int32Array([1])])
// '[{"0":1},{"0":1},{"0":1}]'
JSON.stringify([new Uint8Array([1]), new Uint8ClampedArray([1]), new Uint16Array([1]), new Uint32Array([1])])
// '[{"0":1},{"0":1},{"0":1},{"0":1}]'
JSON.stringify([new Float32Array([1]), new Float64Array([1])])
// '[{"0":1},{"0":1}]'
 
// toJSON()
JSON.stringify({ x: 5, y: 6, toJSON(){ return this.x + this.y } })
// '11'

// Symbols:
JSON.stringify({ x: undefined, y: Object, z: Symbol('') })
// '{}'
JSON.stringify({ [Symbol('foo')]: 'foo' })
// '{}'
JSON.stringify({ [Symbol.for('foo')]: 'foo' }, [Symbol.for('foo')])
// '{}'
JSON.stringify({ [Symbol.for('foo')]: 'foo' }, function(k, v) {
    if (typeof k === 'symbol') {
        return 'a symbol'
    }
})
// undefined

// Non-enumerable properties:
JSON.stringify( Object.create(null, { x: { value: 'x', enumerable: false }, y: { value: 'y', enumerable: true } }) )
// '{"y":"y"}'

JSON.stringify({"1": 1, "2": 2, "3": {"4": 4, "5": {"6": 6}}}, (key, value) => {
    console.log(key)
    console.log(value)
    console.log('========')
    return value
})
// 这里是空字符串''
// { '1': 1, '2': 2, '3': { '4': 4, '5': { '6': 6 } } }
// ========
// 1
// 1
// ========
// 2
// 2
// ========
// 3
// { '4': 4, '5': { '6': 6 } }
// ========
// 4
// 4
// ========
// 5
// { '6': 6 }
// ========
// 6
// 6
// ========
```

# JSON.parse()

{% note info %}
`JSON.parse()`接受两个参数：
- **text**：需要被转换为JSON字符串的值。
- **receiver（可选）**：该参数是一个函数，这个函数也接受键值对，并且返回一个值。如果返回值是`undefined`，则表示要从结果中删除相应的键。关于这个函数是怎么迭代的，请看下面的例子。
{% endnote %}

例子来自[**MDN**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)

```js
JSON.parse('{}')              // {}
JSON.parse('true')            // true
JSON.parse('"foo"')           // "foo"
JSON.parse('[1, 5, "false"]') // [1, 5, "false"]
JSON.parse('null')            // null

JSON.parse('{"p": 5}', (key, value) =>
    typeof value === 'number'
        ? value * 2     // return value * 2 for numbers
        : value         // return everything else unchanged
)
// { p: 10 }

JSON.parse('{"1": 1, "2": 2, "3": {"4": 4, "5": {"6": 6}}}', (key, value) => {
    console.log(key)
    console.log(value)
    console.log('========')
    return value
})
// 1
// 1
// ========
// 2
// 2
// ========
// 4
// 4
// ========
// 6
// 6
// ========
// 5
// { '6': 6 }
// ========
// 3
// { '4': 4, '5': { '6': 6 } }
// ========
// 这里是空字符串''
// { '1': 1, '2': 2, '3': { '4': 4, '5': { '6': 6 } } }
// ========
```