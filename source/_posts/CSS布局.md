---
title: CSS布局
categories:
    - HTML&CSS
abbrlink: ee1ff2c0
date: 2019-05-13 16:06:51
---

# 浮动

首先介绍引入`float`属性的初衷：为了实现文字环绕图片的效果，就像下面的例子：

```html
<div style="width: 500px; border: 1px solid black;">
    <img src="https://mdn.mozillademos.org/files/13340/butterfly.jpg" alt="A pretty butterfly with red, white, and brown coloring, sitting on a large leaf" style="float: left">

    <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci vel, viverra egestas ligula. Curabitur vehicula tellus neque, ac ornare ex malesuada et. In vitae convallis lacus. Aliquam erat volutpat. Suspendisse ac imperdiet turpis. Aenean finibus sollicitudin eros pharetra congue. Duis ornare egestas augue ut luctus. Proin blandit quam nec lacus varius commodo et a urna. Ut id ornare felis, eget fermentum sapien.</p>
</div>
```



浮动元素必须有宽度
display: inline-block;

# 定位
