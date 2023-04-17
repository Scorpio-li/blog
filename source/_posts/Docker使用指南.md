---
title: Docker使用指南
comment: true
date: 2023-04-14 11:52:41
tags: 运维
description:
categories: 
keywords:
---
## 认识Docker

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c09275e60e947e4836c7323809cbbec~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

docker 的架构图如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83f7ef6f44445fbbd960dfd3d656820~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

从图中可以看出几个组成部分

- docker client: 即 docker 命令行工具

- docker host: 宿主机，docker daemon 的运行环境服务器

- docker daemon: docker 的守护进程，docker client 通过命令行与 docker daemon 交互

- image: 镜像，可以理解为一个容器的模板，通过一个镜像可以创建多个容器

- container: 最小型的一个操作系统环境，可以对各种服务以及应用容器化，是镜像的运行实例

- registry: 镜像仓库，存储大量镜像，可以从镜像仓库拉取和推送镜像

Docker 技术的三大核心概念，分别是：镜像 Image、容器 Container、仓库 Repository。

## 安装 Docker

### 软件安装

在本地安装 docker/docker-compose，通过 [Docker Desktop](https://www.docker.com/get-started/)下载 docker 后，双击安装即可。

如果是个人服务器且为 linux，可参考 [安装 docker](https://docs.docker.com/engine/install/centos/) ,它将 docker 与 docker compose 一并安装。

[在CentOS上安装Docker Engine](https://dockerdocs.cn/engine/install/centos/index.html)

### 命令行安装

Homebrew 的 Cask 已经支持 Docker for Mac，因此可以很方便的使用 Homebrew Cask 来进行安装，执行如下命令：

```
brew cask install docker
```

查看版本

```
docker -v
```

## 使用Docker启动一个项目

在项目根目录下执行：touch Dockerfile

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cd273772060479ba041a9ce76f3bd4e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 拉取 Nginx 镜像

首先打开你的Docker，默认会启动。

控制台拉取 Nginx 镜像：

docker pull nginx

出现下面的信息说明拉取Nginx镜像成功

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acd02c73c8bd4140a7000daef4a1b858~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

在根目录创建 Nginx 配置文件：

touch default.conf

```js
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;
    error_log  /var/log/nginx/error.log  error;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 配置镜像

打开Dockerfile文件，写入：

```
FROM nginx  
COPY dist/ /usr/share/nginx/html/  
COPY default.conf /etc/nginx/conf.d/default.conf

```

解释一下代码：

- FROM nginx 指定该镜像是基于 nginx:latest 镜像而构建的；

- COPY dist/ /usr/share/nginx/html/ 命令的意思是将项目根目录下 dist 文件夹中的所有文件复制到镜像中 /usr/share/nginx/html/ 目录下；

- COPY default.conf /etc/nginx/conf.d/default.conf 将 default.conf 复制到 etc/nginx/conf.d/default.conf，用本地的 default.conf 配置来替换 Nginx 镜像里的默认配置。

### 构建镜像

Docker 通过 build 命令来构建镜像：

```
docker build -t <Image> .
```

参数说明：

- -t 参数给镜像命名

- <Image> 镜像名称

- . 是基于当前目录的 Dockerfile 来构建镜像

### 查看镜像

运行docker image ls | grep docker-demo-vue查看镜像

### 运行容器

```
docker run -d -p 3000:80 --name <container> <Image>
```

参数解释：

- -d 设置容器在后台运行

- -p 表示端口映射，把本机的 3000 端口映射到 container 的 80 端口（这样外网就能通过本机的 3000 端口访问了。

- --name 设置容器名 container

- Image 是我们上面构建的镜像名字

### 查看容器

可以运行docker ps 查看容器

### 查看对应的静态文件

curl -v -i localhost:端口号

### 发布镜像

1. 命令行执行 docker login，之后输入我们的账号密码，进行登录；

2. 推送镜像之前，需要打一个 Tag，执行 

```
docker tag <image> <username>/<repository>:<tag>
```

## 镜像仓库

### 镜像仓库与拉取

大部分时候，我们不需要自己构建镜像，我们可以在[官方镜像仓库 Docker Hub](https://hub.docker.com/explore/)拉取镜像。

可以简单使用命令 docker pull 拉取镜像。

拉取镜像后可以使用 docker inspect 查看镜像信息，如配置及环境变量等。

```
# 加入拉取一个 node:alpine 的镜像
$ docker pull node:alpine

# 查看镜像信息
$ docker inspect node:alpine

# 列出所有镜像
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
node                alpine              f20a6d8b6721        13 days ago         105MB
mongo               latest              965553e202a4        2 weeks ago         363MB
centos              latest              9f38484d220f        8 months ago        202MB
```

### 构建镜像与发布

但并不是所有的镜像都可以在镜像仓库中找到，另外我们也需要为我们自己的业务应用去构建镜像。

使用 docker build 构建镜像，docker build 会使用当前目录的 Dockerfile 构建镜像，至于 Dockerfile 的配置，参考下节。

```
# -t node-base:10: 指定镜像以及版本号
# .: 指当前路径
$ docker build -t node-base:10 .
```

当构建镜像成功后可以使用 docker push 推送到镜像仓库。

### Dockerfile

在使用 docker 部署自己应用时，往往需要独立构建镜像。

docker 使用 Dockerfile 作为配置文件构建镜像，简单看一个 node 应用构建的 dockerfile。

```js
// <!-- dockerfile。 -->
FROM node:alpine

ADD package.json package-lock.json /code/
WORKDIR /code

RUN npm install --production

ADD . /code

CMD npm start
```

### FROM

基于一个旧有的基础镜像，格式如下。
```
FROM <image> [AS <name>]

# 在多阶段构建时会用到
FROM <image>[:<tag>] [AS <name>]

FROM node:16-alpine
FROM nginx:alpine
```

### ADD

把宿主机的文件或目录加入到镜像的文件系统中。

```
ADD [--chown=<user>:<group>] <src>... <dest>

ADD . /code
```

### RUN

在镜像中执行命令，由于 ufs 的文件系统，它会在当前镜像的顶层新增一层。

```
RUN <command>

RUN npm run build
```

### 容器

镜像与容器的关系，类似于代码与进程的关系。

```
docker run 创建容器
docker stop 停止容器
docker rm 删除容器
```

### 容器管理

- docker ps 列出所有容器

```
$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                    NAMES
404e88f0d90c        nginx:alpine         "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes        0.0.0.0:8888->80/tcp     nginx
498e7d74fb4f        nginx:alpine         "nginx -g 'daemon of…"   7 minutes ago       Up 7 minutes        80/tcp                   lucid_mirzakhani
2ce10556dc8f        redis:4.0.6-alpine   "docker-entrypoint.s…"   2 months ago        Up 2 months         0.0.0.0:6379->6379/tcp   apolloserverstarter_redis_1
```

- docker port 查看容器端口映射

```
$ docker port nginx
80/tcp -> 0.0.0.0:8888
```

- docker stats 查看容器资源占用

```
$ docker stats nginx
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
404e88f0d90c        nginx               0.00%               1.395MiB / 1.796GiB   0.08%               632B / 1.27kB       0B / 0B             2
```

- docker logs 查看容器日志logs

### 移除

- 停止运行容器并移除容器stop && rm

```
// 停止运行容器 docker stop<容器ID或容器名>
docker stop dooringX-Admin

// 移除容器 docker rm <容器ID或容器名>\
docker rm dooringX-Admin

// 查看当前容器
docker container ls -a 
```

- 移除镜像 image rm

```
docker image rm doorxing-admin
```








