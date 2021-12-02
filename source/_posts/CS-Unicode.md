---
title: Unicode
categories:
  - CS
abbrlink: 1c0c8dfc
date: 2021-11-28 21:56:38
---

# 字符和码位

{% note info %}

- 字符：又叫抽象字符，是用于组织、控制 或 表示 文本数据 的 信息单元。
- 码位：是分配给单个字符的一个数字。`U+<hex>` 是码位的格式，其中 `U+` 代表 Unicode 的前缀，而 `<hex>` 表示十六进制数字。

---

每一个字符都有对应的码位，如 `A` 对应 `U+0041`。但是并非所有码位都有关联字符。
{% endnote %}

# Unicode 平面

{% note info %}
平面是从 `U+n0000` 到 `U+nFFFF` 的 `65536` 个连续的码位，`n` 的取值范围从 `0` 到 `16`。平面将全部的 Unicode 码位分成了 `17` 个均等的组。
{% endnote %}

基本多文种平面为 `U+0000` 到 `U+FFFF`，简称 BMP（Basic Multilingual Plane）。其余 `16` 个平面称为星光平面或辅助平面。

# 码元

{% note info %}
码元是用于以给定编码格式，对每个字符编码的一个二进制位序列。在不同的编码格式中，一个码元有多少字节是不同的。
UTF-8 中一个码元可能有 `1 - 4` 个字节。UTF-16 中一个码元有 `2` 个字节。UTF-32 中一个码元有 `4` 个字节。
{% endnote %}

# 代理对

{% note info %}
在 UTF-16 中代理对是对那些由 `2` 个 `16` 位长的码元所组成序列的单个抽象字符的表示方式。代理对中的首个值为高位代理码元，而第二个值为低位代理码元。高位代理码元从 `0xD800` 到 `0xDBFF` 取值。低位代理码元从 `0xDC00` 到 `0xDFFF` 取值。
{% endnote %}

在 UTF-16 中，对于基本多文种平面，一个 `16` 位长的码元就可以解决了。但是对星光平面内的码元，如 `U+1F600`，需要两个 `16` 位长的码元才能将其表示，这就是代理对。`U+1F600` 对应的代理对是 `0xD83D 0xDE00`。转换方法如下所示。

```js
function getSurrogatePair(astralCodePoint) {
  let highSurrogate = Math.floor((astralCodePoint - 0x10000) / 0x400) + 0xd800;
  let lowSurrogate = ((astralCodePoint - 0x10000) % 0x400) + 0xdc00;
  return [highSurrogate, lowSurrogate];
}
getSurrogatePair(0x1f600); // => [0xD83D, 0xDE00]

function getAstralCodePoint(highSurrogate, lowSurrogate) {
  return (highSurrogate - 0xd800) * 0x400 + lowSurrogate - 0xdc00 + 0x10000;
}
getAstralCodePoint(0xd83d, 0xde00); // => 0x1F600
```

# 组合符号

{% note info %}
字位又称形素、字素、或符号，对一些书写系统来说是最小的构成单位。
{% endnote %}

在绝大多数情况下，单个 Unicode 字符表示单个的字位。但也存在单个字位包含一系列字符的情况。例如 å 在丹麦书写系统中是一个原子性的字位，展示它需要使用 `U+0061` 和 `U+030A` 组合在一起。

组合字符会被用户认为是单个字符，但开发者必须使用 `2` 个码位来表示它。

# 参考文献

[每个 JavaScript 开发者都应该了解的 Unicode](https://mp.weixin.qq.com/s/YIJzT7ymxbxNxXYsV8zpVg)
[What every JavaScript developer should know about Unicode](https://dmitripavlutin.com/what-every-javascript-developer-should-know-about-unicode)
