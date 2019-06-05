---
title: 对docker数据持久化的理解
categories:
    - Docker
abbrlink: 9d0d99dd
date: 2019-06-05 17:23:56
---

# 背景

项目需要读取本地的一个文件，采用的方式是挂载本地目录作为`volume`。但是在将这个项目部署到远端服务器时，项目`image`可以直接拉取，而项目所需文件的迁移就成了问题

# 解决方案

{% note info %}
1. 创建一个专门用于`COPY`数据的`image`，起名为`data-image:1.0`
2. 接着就可以远程访问服务器，创建`docker-compose.yml`文件，通过挂载`volume`的方式实现数据共享
3. 到此为止就可以拉取镜像、启动服务了
{% endnote %}

```Dockerfile
# Dockerfile
FROM alpine

COPY ./data /usr/src/data
```

```yml
# docker-compose.yml
version: '3'

services:
    project:
        image: project-image
        volumes:
            - project-data:/usr/src/data

    data-container:
        image: data-image:1.0
        volumes:
            - project-data:/usr/src/data

volumes:
    project-data:
```

# 注意事项

{% note info %}
- 只有`volume`可以做到数据持久化，`image`内的数据是无法被更改的
- 当项目启动后对依赖文件的改动会被记录在`project-data`中，而两个`image`中`/usr/src/data`目录下的内容会一直保持它们最开始的状态
- 启动服务后，`project-data`会被挂载到两个`container`上，实现数据的共享，这个时候两个`container`中`/usr/src/data`目录下的内容与`project-data`同步
- 如果删除了`project-data`，则对依赖文件的改动会丢失
{% endnote %}
