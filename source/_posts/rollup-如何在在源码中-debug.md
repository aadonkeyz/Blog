---
title: 如何在在源码中 debug
categories:
  - FE tools
  - rollup 2.69.2
abbrlink: d9c5729d
date: 2022-03-28 21:34:15
---

# 使项目可以本地运行

因为 rollup 是使用 ts 编写的，所以需要下载 ts-node 来支持本地运行。过程中主要遇到两类问题：

- SyntaxError: Cannot use import statement outside a module
- ts 报错

第一个问题通过修改 tsconfig.json 来避免，第二个问题通过 @ts-ignore 来忽略。

# 修改 cli 文件

修改 rollup cli 文件，然后本地运行 cli 文件的，是改动最小的方式。这里的主要改动是让 rollup 去读我们指定的 rollup.config.ts 文件，而不是去项目根目录下寻找。

# 打包

在项目根目录下创建 learn 文件夹，里面保存着我们的 rollup.config.ts 和 build.ts。然后在 package.json 中添加对应的命令，就可以在源码中进行 debug 了！

# 修改详情

具体修改内容，可参照 https://github.com/aadonkeyz/rollup/commit/55f0e18bdd0c3fd0276d53c85df52587c87fcf06
