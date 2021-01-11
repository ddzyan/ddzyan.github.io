---
title: consul的负载均衡fabio
date: 2020-10-12 15:49:38
categories:
  - 微服务
tag:
  - 服务注册和发现
  - consul
  - fabio
---
## 简介

fabio 是 ebay 团队用 golang 开发的一个快速、简单零配置，能够让 consul 部署的应用快速支持 http(s) 的负载均衡路由器。

因为 consul 支持服务注册与健康检查，所以 fabio 能够零配置提供负载，升级部署从未如此简单。

根据项目的介绍，fabio 能提供每秒15000次请求。



fabio 的工作就很简单了！ 就是直接从consul 注册表里面取出健康的服务，根据服务注册时候的 tags 配置自动创建自己的路由表，然后当一个 http 请求过来的时候自动去做负载均衡



资料：

- [官网](https://fabiolb.net/feature/docker/)
- [配置](https://fabiolb.net/ref/)



## 工作原理

```
======    服务注册     =========         =========
  A服务   <------>    consul集群  ---->  健康的 A/不健康的 A 集群
======    健康检查     =========         =========
                          ^
                          | 加入/移出路由表
                          |
                       ========
                       fabio 集群
                       ========
                          |
                          | A服务   如果找到则成功路由，否则返回错误
                          V
                        http 请求
```

<!--more-->

## 部署

创建配置文件 fabio.properties，更多配置文件信息请参考

```
# 配置 consul 服务地址
registry.consul.addr = 10.10.0.12:8500
registry.consul.register.addr = 10.10.0.12:9998
metrics.target = stdout
```



docker 部署

```shell
$ docker run -d --name fabio -p 9999:9999 -p 9998:9998 -v $PWD/fabio/fabio.properties:/etc/fabio/fabio.properties fabiolb/fabio
```



docker-compose 部署 docker-compose.yml

```yml
version: '3.6'

services:
  fabio:
  image: "magiconair/fabio"
  ports:
  - "9998:9998"
  - "9999:9999"
  volumes:
  - ./fabio.properties:/etc/fabio/fabio.properties
```



部署完成后打开ui管理界面 `127.0.0.1:9998` 如下，这里将显示所有注册完成的路有列表

![image-20201010144659162](https://tva1.sinaimg.cn/large/007S8ZIlly1gjk8ksputsj31y90dbdhk.jpg)



## 客户端配置

nodejs 代码仓库: https://github.com/ddzyan/node-consul.git

核心逻辑：在 client 注册时需要添加上指定 tag 标签，类似于`urlprefix-`，如下

```js
consul.agent.service.register({
	name: 'cmdWork', // 服务名称可以不唯一，可以通过 name 获取批量服务信息
  id: `cmdWork-${number}`, // 服务Id必须唯一
  tags: ['urlprefix-a.com'], // 标签信息
  meta: {
    describe: 'commP',
  },
  address: '172.16.0.39', // 服务地址
  port: 3001, // 服务端口号
  check: {
    http: 'http://172.16.0.39:3001/health', // 服务地址
    interval: '5s', // 健康检测轮询时间
    timeout: '10s', // 超时时间
    status: 'critical', // 初始化服务状态
  },
});
```



添加后在打开 fabio ui 管理界面，会发现路有已经完成注册，并且会根据服务数量自动进行轮询权限调整，可以通过一下命令测试请求请求转发

```shell
$ curl -H "Host:a.com" http://127.0.0.1:9999/health
```

