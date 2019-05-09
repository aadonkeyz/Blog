---
title: Windows下Node安装及配置
categories:
    - Node.js
abbrlink: ee28a0bf
date: 2019-02-28 10:23:13
---

# Node的安装

[首先附上Node的下载链接](https://nodejs.org/en/)

安装过程中除了对安装位置进行修改外，其他的都选择默认项就可以，我的安装位置选择为D:\nodejs\，安装之后在命令行中分别输入`node -v`和`npm -v`会显示出对应的node及npm版本号。

![Node安装成功](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/node%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.png)

刚刚安装完成之后，nodejs的目录如下所示。

![Node安装后目录](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/node%E5%AE%89%E8%A3%85%E5%90%8E%E7%9B%AE%E5%BD%95.png)

# Node路径修改

接下来要修改npm安装全局模块的路径和缓存cache的路径，这样做是因为默认路径都在C盘，而我们并不想过多的占用C盘空间。

在修改路径前先在D:\nodejs\下创建node_global和node_cache文件夹。

![Node下新建文件夹](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/nodejs%E4%B8%8B%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9.png)

然后在命令行中输入以下指令：

```md
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
```

上面的指令为修改路径，现在让我们验证下路径修改是否成功：首先通过npm全局安装一个模块，如果这个模块被下载到我们指定的文件夹中，则说明修改成功。

```md
npm install express -g
```

![模块全局安装成功](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/%E6%A8%A1%E5%9D%97%E5%85%A8%E5%B1%80%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.png)

检查D:\nodejs\node_global文件夹，在里面的node_modules中发现刚刚全局安装的express模块，说明上述对路径的修改已经成功。

![express安装在指定位置](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/express%E5%AE%89%E8%A3%85%E5%9C%A8%E6%8C%87%E5%AE%9A%E4%BD%8D%E7%BD%AE.png)

# Node环境变量修改

因为我们刚刚修改了模块的全局安装路径，所以需要修改相应的环境变量以保证在Node使用过程中，可以找到正确的位置来引用模块，操作如下：

{% note info %}
我的电脑 - 右键 - 属性 - 高级系统设置 - 高级 - 环境变量
{% endnote %}

![进入环境变量](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/%E8%BF%9B%E5%85%A5%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E7%9A%84.png)

{% note info %}
在“系统变量”下新建`NODE_PATH`,输入：`D:\nodejs\node_global\node_modules`。
{% endnote %}

![系统变量修改](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/%E7%B3%BB%E7%BB%9F%E5%8F%98%E9%87%8F%E4%BF%AE%E6%94%B9.png)

{% note info %}
在“用户变量”下的`Path`中用`D:\nodejs\node_global`替换`C:\Users\17590\AppData\Roaming\npm`。
{% endnote %}

![用户变量修改](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/%E7%94%A8%E6%88%B7%E5%8F%98%E9%87%8F%E4%BF%AE%E6%94%B9.png)

环境变量修改完成之后，打开一个新的命令行窗口，先输入`node`，按回车进入nodejs的交互式命令控制台，然后输入``require('express')``,如果能够正常显示express模块的信息，则说明环境变量的修改也成功了。

![环境变量修改成功](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BNode%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E4%BF%AE%E6%94%B9%E6%88%90%E5%8A%9F.png)

# 使用淘宝镜像

接下来修改npm配置，使用国内的淘宝镜像以保证模块下载的稳定性和速度，下列两种方式二选一即可。

```python
# 安装cnpm模块
# 以后使用cnpm替代npm
npm install -g cnpm --registry=https://registry.npm.taobao.org

# 修改源地址为淘宝镜像
npm config set registry https://registry.npm.taobao.org
```