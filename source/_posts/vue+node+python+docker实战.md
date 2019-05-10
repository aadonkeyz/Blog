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
- **node-serve**：使用hapi开发的基础服务器，主要负责系统内数据的增删改查；
- **node-worker**：基于node的消息队列消费者；
- **python-serve**：使用flask开发的算法服务端，主要负责系统内算法的运行；
- **mysql**：数据库；
- **phpmyadmin**：数据库UI；
- **rabbitmq**：消息队列服务器。

---
上面介绍的组成部分，除了`mysql`和`phpmyadmin`不需要利用`Dockerfile`构建镜像外，其余全需要。

马上就会讲解它们的`Dockerfile`，不过需要注意的是，**请将所有镜像全部构建完成后，再利用`docker-compose`来启动所有容器。否则只单独启动像`node-worker`这样的容器，是不会成功的，因为它依赖于`rabbitmq`。**
{% endnote %}

## vue-client

```Dockerfile
# Dockerfile文件
FROM nginx

COPY ./dist /app
COPY ./nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```
```python
# nginx.conf文件
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

        location /api {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://node-serve:8000;
            proxy_connect_timeout 600s;
            proxy_send_timeout 600s;
            proxy_read_timeout 600s;
            send_timeout 600s;
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
2. `docker build -t vue-client:20190509_1 .`：构建一个名称为`vue-client:20190509_1`的镜像。
{% endnote %}

## node-serve

```Dockerfile
# Dockerfile文件
FROM node:10.15.1-alpine

WORKDIR /app

COPY . .

RUN npm install --registry=https://registry.npm.taobao.org && \
    apk add --no-cache tzdata && \
	cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	echo "Asia/Shanghai" > /etc/timezone && \
	apk del tzdata

EXPOSE 8000

CMD ["npm", "start", "-s"]
```
```python
# .dockerignore文件
npm-debug.log
node_modules

.vscode
typings
package-lock.json
```

{% note info %}
在项目根目录下运行`docker build -t node-serve:20190509_1 .`构建一个名称为`node-serve:20190509_1`的镜像。
{% endnote %}

## node-worker

```Dockerfile
# Dockerfile文件
FROM node:10.15.1-alpine

WORKDIR /app

COPY . .

RUN npm install --registry=https://registry.npm.taobao.org && \
	npm install -g pm2 --registry=https://registry.npm.taobao.org && \
    apk add --no-cache tzdata && \
	cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	echo "Asia/Shanghai" > /etc/timezone && \
	apk del tzdata

CMD ["pm2", "start", "ecosystem.config.js", "--no-daemon"]
```
```python
# .dockerignore文件
.vscode/
node_modules/
.git/
.gitignore
```

{% note info %}
在项目根目录下运行`docker build -t node-worker:20190509_1 .`构建一个名称为`node-worker:20190509_1`的镜像。
{% endnote %}

## python-serve

```Dockerfile
# Dockerfile文件
FROM python:3.7.2

WORKDIR /usr/src/app

COPY . .

RUN pip install -U pip && \
    pip install -r requirements.txt

EXPOSE 5000

CMD ["gunicorn", "-c", "gunicorn.conf", "run:app"]
```
```python
# .dockerignore文件
__pycache__
.vscode/
.git/
.gitignore
```

{% note info %}
在项目根目录下运行`docker build -t python-serve:20190509_1 .`构建一个名称为`python-serve:20190509_1`的镜像。
{% endnote %}

## rabbitmq

`rabbitmq`的根目录下只有一个文件，就是`Dockerfile`文件。

```Dockerfile
# Dockerfile文件
FROM rabbitmq:3-management-alpine

RUN rabbitmq-plugins enable rabbitmq_stomp && \
    rabbitmq-plugins enable rabbitmq_web_stomp && \
    rabbitmq-plugins enable --offline rabbitmq_management

EXPOSE 15672
EXPOSE 15674
```

{% note info %}
在项目根目录下运行`docker build -t rabbitmq:stomp .`构建一个名称为`rabbitmq:stomp`的镜像。
{% endnote %}

# docker-compose实例

创建一个名称为`docker-compose`的文件夹，并在其中创建一个`docker-compose.yml`文件。

```yml
version: '3'

services:
    vue-client:
        image: vue-client:20190509_1
        ports:
            - "8888:80"
        depends_on:
            - node-serve
            - node-worker
            - python-serve
        networks:
            - app-net
    
    node-serve:
        image: node-serve:20190509_1
        ports:
            - "8000:8000"
        depends_on:
            - mysql
        networks:
            - app-net
    
    node-worker:
        image: node-worker:20190509_1
        depends_on:
            - python-serve
        networks:
            - app-net

    python-serve:
        image: python-serve:20190509_1
        ports:
            - "5000:5000"
        depends_on:
            - mysql
            - rabbitmq
        networks:
            - app-net

    mysql:
        image: mysql:5.6
        restart: always
        ports:
            - "3306:3306"
        environment:
            MYSQL_ROOT_PASSWORD: yourpassword
        command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
        volumes:
            - mysql-data:/var/lib/mysql
        networks:
            - app-net

    phpmyadmin:
        image: phpmyadmin/phpmyadmin:latest
        restart: always
        ports:
            - "8001:80"
        environment:
            MYSQL_ROOT_PASSWORD: yourpassword
            PMA_HOST: mysql
        depends_on:
            - mysql
        networks:
            - app-net
    
    rabbitmq:
        image: rabbitmq:stomp
        restart: always
        ports:
            - "15672:15672"
            - "15674:15674"
        environment:
            RABBITMQ_DEFAULT_USER: yourname
            RABBITMQ_DEFAULT_PASS: yourpassword
            RABBITMQ_ERLANG_COOKIE: yourpassword
        volumes:
            - rabbitmq-data:/var/lib/rabbitmq
        networks:
            - app-net

networks:
    app-net:

volumes:
    mysql-data:
    rabbitmq-data:
```

{% note info %}
命令介绍：
- `docker-compose build`：构建（重新构建）服务容器；
- `docker-compose up -d`：后台自动（重新）创建服务和关联服务相关容器；
- `docker-compose down`：停止容器、移除容器和网络。

---
遇到的坑：
- **depends_on**：解决容器的依赖、启动先后问题，**但是不会等到所依赖的容器完全启动之后，再启动后续容器**。所以如果在`docker-compose up -d`后，发现`node-worker`部分没有正常工作的话，你在保证`rabbitmq`完全启动后，重启下`node-worker`就可以了。这里有其他的解决办法，比如利用`wait-for-it`脚本等，我简单的试了一下，有些小问题，为了项目进度，暂时搁置，[**相关资料戳这里**](https://docs.docker.com/compose/startup-order/)。
{% endnote %}

# portainer

运行如下命令就可以启动一个docker的可视化管理工具。

```md
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer-data:/data portainer/portainer
```
