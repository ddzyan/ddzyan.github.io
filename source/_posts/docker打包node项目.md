---
title: docker打包node项目
date: 2019-06-11 01:48:41
categories:
  - node
tags:
  - node
  - docker
---

## 简介

### 常规操作的问题

一个项目成功运行需要依赖于：系统版本，代码版本，运行环境，依赖包版本等综合因素控制。如果我们提交代码给运行只提交代码版本，可能会导致因为其他依赖不同，而产生的各种问题。

在 node 中即使使用 package-lock.json，控制依赖包的版本，也无法解决以下的问题：

1. 安装速度慢，如果是集群化部署，影响更大
2. 依赖包所依赖的包版本的管理
3. 回滚的速度慢

### docker 的优势

- 将一个完整可运行的环境（构建期就把依赖打包）的镜像文件，提交给运维进行部署，从而解决依赖包版本不一致问题
- 镜像的启动速度非常的快（几秒以内），也大大加快了部署和回滚的速度，降低了运维操作时间和对线上的影响。

### 上线流程

编写--->测试--->代码评审--->合并分支--->打包（构建期就把依赖打包）---->部署（回滚）

<!--more-->

## 实际操作

### 修改启动脚本（非必须）

#### package.json

目的：将直接在前端运行，输出的日志可以直接在 docker logs 中查看

```json
{
  "scripts": {
    "start": "egg-scripts start"
  }
}
```

### Dockerfile

```
FROM node:10.15.2-alpine

RUN apk --update add tzdata \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apk del tzdata

RUN mkdir -p /usr/src/app

WORKDIR /usr/src/app

# add npm package
COPY package.json /usr/src/app/package.json

RUN npm i --registry=https://registry.npm.taobao.org

# copy code
COPY . /usr/src/app

EXPOSE 7003

CMD npm start

```

## 操作指令

```bash
# 构建镜像
sudo docker build -t iost-gateway  .

# 启动容器
sudo docker run -d --name iost-gateway -p 7007:7007 -v :/root/logs/iost-funds  iost-gateway

# 进入容器
docker exec -it iost-gateway /bin/bash
```
