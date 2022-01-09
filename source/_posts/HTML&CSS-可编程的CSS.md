---
title: 可编程的CSS
categories:
    - HTML&CSS
abbrlink: 6abf3348
date: 2019-06-12 16:16:18
---

# 自定义属性

{% note info %}
- CSS的自定义属性（也叫做变量）是通过前缀`--`定义的
- 区分大小写
- 同一个变量可以在多个选择器内声明。读取的时候，优先级最高的声明生效，这与CSS的“层叠”规则是一致的
- `var(<custom-property-name>, <declaration-value>?)`用于读取变量。`<custom-property-name>`为变量名称，`<declaration-value>`为默认值
- 声明变量的时候可以使用`var()`
{% endnote %}

```html
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8" />
        <title>CSS自定义属性</title>
        <style type="text/css">
            :root {
                --first-color: red;
                --temporary-color: white;
                --second-color: var(--temporary-color);
            }

            #firstParagraph {
                background-color: var(--first-color);
                color: var(--second-color)
            }

            #secondParagraph {
                background-color: var(--second-color);
                color: var(--first-color)
            }

            #container {
                --first-color: green;
            }

            #thirdParagraph {
                background-color: var(--first-color);
                color: var(--second-color)
            }
        </style>
    </head>
    <body>
        <p id="firstParagraph">This paragraph should have a red background and white text.</p>
        <p id="secondParagraph">This paragraph should have a white background and red text.</p>
        <div id="container">
            <p id="thirdParagraph">This paragraph should have a green background and white text.</p>
        </div>
    </body>
</html>
```

![自定义属性](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/HTML%26CSS/%E5%8F%AF%E7%BC%96%E7%A8%8B%E7%9A%84CSS/%E8%87%AA%E5%AE%9A%E4%B9%89%E5%B1%9E%E6%80%A7.png)

# calc

在CSS中使用`calc()`可是实现加、减、乘、除。

```css
width: calc(100% - 20px);
--custom-height: calc(60% + 20px);
height: calc(var(--custom-height) * 1.5);
```

# 通过JavaScript访问自定义属性

{% note info %}
- 可以通过`getComputedStyle().getPropertyValue()`来访问在CSS中定义的变量
- 变量是可继承的
- 不允许修改变量值，否则会抛出错误
{% endnote %}

```html
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8" />
        <title>CSS自定义属性</title>
        <style type="text/css">
            :root {
                --first-color: red;
                --second-color: white;
            }

            #myDiv {
                --first-color: green;
            }
        </style>
    </head>
    <body>
        <div id="myDiv" style="width: 10px"></div>
        <script type="text/javascript">
            let div = document.getElementById('myDiv')

            let computedStyle = document.defaultView.getComputedStyle(div, null)

            console.log(computedStyle['--first-color'])                     // undefined
            console.log(computedStyle.getPropertyValue('--first-color'))    // green
            console.log(computedStyle.getPropertyValue('--second-color'))   // white
        </script>
    </body>
</html>
```