---
title: job
abbrlink: fbd8e0f8
categories:
mathjax: true
---

二分查找比较n次后，查找成功的节点数是$2^{n-1}-1$

---
用两个栈模拟实现一个队列，最大容量：[csdn](https://blog.csdn.net/qq_42194192/article/details/82723949)

---
TCP有序，UDP无序

---
js的微任务和宏任务

---
实现一个 EventEmitter 类，要求包括：on\emit\off\once 方法

---
如何提升页面加载速度，并简述原理
回答得分占比：一个点10%，不在此列的，有道理则+10%
合并压缩js、css文件
延迟加载不需要的资源
使用sprites合并细碎的小图片
使用内嵌的base64图片代替url
对静态资源使用CDN
合理配置缓存策略
服务端启用gzip
支持http2
减少阻塞脚本，使用async
ssr后端渲染
减少重定向

---
页面上存在id为jsBlink的下划线闪动节点，请按照如下需求实现 output 函数
1、函数 output 接收一个字符串参数，每隔200毫秒在闪动节点之前逐个显示字符
2、请新建span节点放置每个字符，其中span必须存在class "word"，并随机加上 color0 ~ color23 中的任一个class（请使用系统随机函数）
3、每次输出指定字符串前，请将闪动节点之前的所有其他节点移除
4、不要销毁或者重新创建闪动节点
5、如果输出字符为空格、<、>，请分别对其进行HTML转义，如果是\n请直接输出<br />，其他字符不需要做处理
6、请不要手动调用output函数
7、当前界面为系统执行 output('hello world\n你好世界') 之后，最终的界面，过程请参考以下图片
8、请不要手动修改html和css
9、不要使用第三方插件
10、请使用ES5语法

---
数组是顺序存储结构

---
前中后遍历

---
textarea标签的属性不包括width

---
link是XTHML标签，无兼容问题
link可以使用JavaScript控制DOM改变样式，@import不支持；
link引用css时，页面加载同时加载样式，@import需要页面完全载入以后加载

---
io不是node的内置模块
nw.js不是基于nodejs的

---
html5的mainfest

---
页面加载过程，由于 html 的层次结构已经是树状结构，因此可以实现边加载边生成 DOM 树

---
31
用html+css+js实现一个轮播图，功能如下：
1.    支持自动播放
2.    支持移动端事件
3.    支持无限无缝切换，即滑动到第一张或者最后一张继续滑动则可以继续往前或往后翻页
4.    支持回调事件
5.    按住鼠标/手指按住图片可随鼠标/手指滑动，当移动距离超过图片的1/2时则翻页，不超过则恢复原样
注：是否用第三方js库(或者不用)，请说明原因

1.写出页面的HTML结构(20%)
2.写出页面基本的css样式(20%)
3.实现基本事件绑定，包括mouse 和 touch 事件 (20%)
4.能够实现无限无缝滚动和事件回调 (20%)
5.实现按住后移动鼠标/手指，大于1/2翻页，小于1/2恢复(10%)
6.绑定事件优化(10%)

---
怎样理解JavaScript 的 Event Loop，并且试用代码举例说明setTimeout和Promise差异
1.    阐述明白事件机制包括主进程和异步回调进程，30%
2.    阐述清楚事件中的宏任务和微任务，30%
3.    SetTimeout 和Promise 讲解正确，30%
4.   引申加入 ajax、用户交互事件等细节，10%

---
本题展示了一个简化版的搜索框智能提示功能，请按照如下要求完成suggest函数。
1、当输入框的值发生变化时，系统会调用suggest函数，用于显示/隐藏智能提示数据，参数items为一个字符串数组。
2、当items中的字符串和输入框的值匹配时，将匹配的数据依次渲染在ul下的li节点中，并显示.js-suggest节点，否则移除ul下的所有li节点，并隐藏.js-suggest节点
3、输入框的值需要移除两侧空白再进行匹配
4、输入框的值为空时，按照全部不匹配处理
5、字符串使用模糊匹配，比如"北大"能匹配"北大"和"北京大学"，但不能匹配"大北京"，即按照 /北.*?大.*?/ 这个正则进行匹配
6、通过在.js-suggest节点上添加/移除 hide 这个class来控制该节点的隐藏/显示
7、当前界面是执行 suggest(['不匹配数据', '根据输入框的值', '从给定字符串数组中筛选出匹配的数据，依次显示在li节点中', '如果没有匹配的数据，请移除所有li节点，并隐藏.js-suggest节点']) 后的结果
8、请不要手动修改html和css
9、不要使用第三方插件
10、请使用ES5语法
