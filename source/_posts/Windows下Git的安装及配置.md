---
title: Windows下Git的安装及配置
categories:
    - Development Tools
    - Git
abbrlink: 6749cdaf
date: 2019-02-27 22:35:21
---

# Git的安装

[首先附上Git的下载链接](https://git-scm.com/downloads)

在Git下载完成之后，除了对安装位置进行更改之外，其他的按照默认选项来就可以。当安装完成之后，打开命令行，输入`git`,如果看到有关Git的相关信息就说明安装成功了！

![Git安装成功](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGit%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/git%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.png)

# Git的配置

安装成功后要配置Git的用户名与邮箱，相当于自报家门，在进行这一配置时可以通过参数指定配置的作用域。
作用域：`--system > --global > --local`。
优先级：`--local > --global > --system`。

以github和gitlab为例，由于我在两个网站的注册用户名与邮箱均为为aadonkeyz和aadonkeyz@gmail.com，所以我选择进行一次全局配置即可。

全局配置指令：

```md
git config --global user.name "aadonkeyz"
git config --global user.email "aadonkeyz@gmail.com"
```

# SSH 配置

因为我们需要在同一台电脑中同时使用github和gitlab，为了保证两个网站在使用SSH时不产生冲突，需要为两个网站生成两对不同的秘钥，并在SSH的config文件中指定Git在访问不同的网站时要使用相对应的秘钥。

```md
ssh-keygen -t rsa -C aadonkeyz@github
```

{% note info %}
-t = The type of the key to generate
-C = comment to identify the key
{% endnote %}

注释内容一般填写注册邮箱，不过我在两个网站的注册邮箱相同，所以我的注释分为了：

```python
# github
aadonkeyz@github
# gitlab
aadonkeyz@gitlab
```

![为github生成秘钥的命令行](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGit%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/%E4%B8%BAgithub%E7%94%9F%E6%88%90%E7%A7%98%E9%92%A5%E7%9A%84%E5%91%BD%E4%BB%A4%E8%A1%8C.png)

上面是我为github生成秘钥的过程，在过程中我没有修改秘钥的默认保存位置，但是为了对秘钥进行区分修改了秘钥的名称，并设定了密码（passphrase），一旦为秘钥设置了密码，那么以后每次使用SSH时都会要求输入秘钥。请读者自行将生成的秘钥添加到github中去，并在gitlab上进行相同的操作。

接下来开始自定义config文件，首先在保存秘钥的文件夹下创建一个名为config的文件，注意不要任何后缀（可以先创建名为config的文本文档，之后删除“.txt”后缀），然后填写如下内容：

```md
Host github.com
    User git
    IdentityFile ~/.ssh/github_rsa

Host gitlab.com
    User git
    IdentityFile ~/.ssh/gitlab_rsa
```

保存并退出，最后我们来通过指令，`ssh -T github.com`和`ssh -T gitlab.com`测试一下是否成功。

![ssh连接测试](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGit%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE/ssh%E8%BF%9E%E6%8E%A5%E6%B5%8B%E8%AF%95.png)

大功告成！