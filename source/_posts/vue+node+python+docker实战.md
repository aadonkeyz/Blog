---
title: vue+node+python+docker实战
categories:
  - Docker
abbrlink: 96fb2f92
date: 2019-05-08 15:09:43
---

# Docker基础概念

[Docker中文文档](https://yeasy.gitbooks.io/docker_practice/introduction/what.html)
[Docker英文文档](https://docs.docker.com/)

# 系统架构

## 注意事项

{% note info %}
系统中各组成部分介绍：
- **vue-client**：使用vue开发的前端界面；
- **node-serve**：使用hapi开发的基础服务器，主要负责项目内数据的增删改查；
- **node-worker**：基于node的消息队列消费者；
- **python-serve**：使用flask开发的算法服务端，主要负责项目内算法的运行；
- **mysql**：数据库；
- **phpmyadmin**：数据库ui；
- **rabbitmq**：消息队列服务器。

---
上面介绍的组成部分，除了`mysql`和`phpmyadmin`不需要利用`Dockerfile`构建镜像外，其余全需要。

马上就会讲解它们的`Dockerfile`，不过需要注意的是，**请将所有镜像全部构建完成后，在利用`docker-compose`来启动所有容器。否则单独启动像`node-worker`这样的容器，是不会成功的。**
{% endnote %}

## vue-client

这个项目使用vue-cli3搭建的，需要在项目根目录下创建`Dockerfile`和`nginx.conf`文件。

```Dockerfile
# Dockerfile
FROM nginx

COPY ./dist /app
COPY ./nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

```python
# nginx.conf
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
        root   /app;
        index  index.html;
        try_files $uri $uri/ /index.html;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
        root   /usr/share/nginx/html;
        }
    }
}
```

{% note info %}
在项目根目录下依次运行如下命令：
1. `npm run build`：这会将项目打包到根目录下的`dist`文件夹；
2. `docker build -t vue-client .`：构建一个名称为`vue-client`的镜像。
{% endnote %}

## node-serve

```Dockerfile

```

## node-worker

```Dockerfile
FROM node:10.15.1-alpine

WORKDIR /app

COPY ./package.json ./package.json

RUN npm install -g cnpm --registry=https://registry.npm.taobao.org && \
    apk add --no-cache tzdata && \
	cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	echo "Asia/Shanghai" > /etc/timezone && \
	apk del tzdata

COPY . .

CMD ["node", "index.js"]
```

## python-serve

```Dockerfile
FROM python:3.7.2

ENV API_SERVER_HOME=/var/www/app

WORKDIR "$API_SERVER_HOME"

COPY ./requirements.txt ./requirements.txt

RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD [ "python", "run.py" ]
```

# docker-compose实例



# volume备份及还原
