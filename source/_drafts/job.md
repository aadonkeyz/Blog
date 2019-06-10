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