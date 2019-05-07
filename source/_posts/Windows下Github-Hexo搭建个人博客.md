---
title: Windows下Github+Hexo搭建个人博客
categories:
    - Blog
abbrlink: d4a0c543
date: 2019-02-28 16:11:21
---

# 准备工作

{% note info %}
- [Node安装及配置](https://aadonkeyz.com/posts/ee28a0bf/#more)
- [Git安装及配置](https://aadonkeyz.com/posts/6749cdaf/#more)
- [阅读Hexo的文档](https://hexo.io/zh-cn/docs)
{% endnote %}

# Hexo安装及博客初始化

在Node安装成功之后，在命令行中使用npm命令安装Hexo，输入：`npm install -g hexo-cli`。

安装完成后我们自己创建一个空文件夹Blog作为博客的根目录，双击进入文件夹，之后按住shift键并点击鼠标右键，选择菜单中的“在此处打开Powershell窗口”，这样所有的命令行操作都是以Blog为根目录进行的，**后文的所有命令行均是以Blog为根目录！**

在新打开的命令行窗口中依次输入：
```md
hexo init
hexo g
hexo s
```

完成后就可以在浏览器中输入[http://localhost:4000](http://localhost:4000)来查看我们的本地博客了。

![本地博客初始化](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E6%9C%AC%E5%9C%B0%E5%8D%9A%E5%AE%A2%E5%88%9D%E5%A7%8B%E5%8C%96.png)

# 主题更换

我选择的是评价较好的Next主题，首先将主题下载到Blog/themes中，在Blog根目录启动命令行，输入：

```md
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

现在先简单的介绍下Blog下的themes文件夹和_config.yml文件，themes下保存的是博客的主题，每一种主题为一个文件夹，例如目前为止我们有两种主题landscape（自带的）和next。下面请仔细注意，每个主题文件夹下都有一个_config.yml文件，我们称之为主题配置文件，而Blog文件夹下的_config.yml称之为站点配置文件。

![主题配置与站点配置](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E4%B8%8E%E7%AB%99%E7%82%B9%E9%85%8D%E7%BD%AE.png)

接下来打开站点配置文件，修改主题为next，保存。

```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

也可以修改next的主题样式，当前版本的next有四种样式，我选择的是最后一个Gemini。

```yml
# Schemes
# scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```

除此之外有关于博客样式和功能菜单的配置，大部分都可以在这两个配置文件中发现，[Next主题也提供了一份配置方面的文档](https://theme-next.org/docs/theme-settings/)。

都保存之后，连续执行三个命令：

```md
hexo clean
hexo g
hexo s
```

接着刷新浏览器就可以观察到页面样式的变化。

# 博客的备份及部署

到目前为止，已经成功用Hexo建立本地博客，接下来要先将本地博客文件备份至github中，然后再部署博客。

在开始前先介绍一下这部分的安排，打算创建两个分支master和root。

{% note info %}
- **master**：负责博客的部署，所以其中存放的是部署后生成的文件；
- **root**：负责存放博客的原始文件，并且要注意将root设置为默认分支。
{% endnote %}

首先在github上创建个人仓库，然后复制仓库的SSH，格式为`git@github.com:username/repository_name.git`，将username和repository_name分别换为你的github用户名和仓库名。

![仓库的SSH](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E4%BB%93%E5%BA%93%E7%9A%84ssh.png)

接着修改github账号的Setting中的Emails设置，取消下图中红线标记的功能。

![修改github的emails设置](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E4%BF%AE%E6%94%B9github%E7%9A%84emails%E8%AE%BE%E7%BD%AE.png)

然后通过下列指令将本地博客文件备份至github的root分支上：

```python
git init
# 注意add后面有个“空格”和“.”
git add .
```

![子仓库报错](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E5%AD%90%E4%BB%93%E5%BA%93%E6%8A%A5%E9%94%99.png)

备份过程中会遇到一个警告提示，原因是我们前面使用的主题Next也是一个Git仓库，所以在将Blog文件下所有文件加入Git缓存中时会警告我们子仓库不会被添加，解决方式为将这个Git子仓库以我们的本地文件夹形式加入到Git缓存中。

```python
git rm --cached themes/next -f
# next后面的“/”表示添加的是文件夹，一定要加上
git add themes/next/
git commit -m "first commit to branch root"
# 将本地master分支重命名为root
git branch -m master root
git remote add origin git@github.com:aadonkeyz/Blog.git
git push -u origin root
```

最后一个指令输完可能会要求我们输入SSH秘钥的密码，如果要求验证，输入密码即可。

上面的操作已经成功将本地博客文件添加到仓库的root分支，接着打开站点配置文件进行如下修改：
```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:aadonkeyz/Blog.git
  branch: master
```

保存后开始通过命令行下载并保存一个名为hexo-deployer-git的插件，然后就可以将博客部署到github上了。

```md
npm install hexo-deployer-git --save
hexo clean
hexo g
hexo d
```

目前已经将博客部署到master分支了，但是还要进行一些修改才可以，进入仓库的Setting中，找到Github Pages，将Source选择为master branch。

![修改Github Pages的设置](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E4%BF%AE%E6%94%B9Github%20Pages%E7%9A%84%E8%AE%BE%E7%BD%AE.png)

完成后会发现这部分会有所变化，多出一个链接，格式为`https://username.github.io/仓库名/`。

![站点链接](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/Windows%E4%B8%8BGithub%2BHexo%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/%E7%AB%99%E7%82%B9%E9%93%BE%E6%8E%A5.png)

回到站点配置文件中将url改为刚刚的链接，root改为`/仓库名/`。

```yml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://aadonkeyz.github.io/Blog/
root: /Blog/
```

之后将新的修改备份至github并重新部署博客。

```md
git add .
git commit –m “change URL and Deploy of site config”
git push -u origin root
hexo clean
hexo g
hexo d
```

大功告成，等待几秒后（github需要时间来完成更新），在浏览器中输入刚刚的url链接就可以查看我们部署在github上的博客啦！