---
title: consul的docker自动注册
date: 2020-10-12 15:50:38
categories:
  - 微服务
tag:
  - 服务注册和发现
  - consul
  - Registrator
---
## 简介

Registrator 通过检查容器在线来自动为任何Docker容器注册和注销服务，后端注册中心支持consul、etcd、skydns2、zookeeper等存储



## 部署

必须先在本地部署内容：

- docker
- consul
- fabio (不使用则可以不安装)



需要在每个docker engine上部署一个Registrator实例了，可以通过 docker 部署 registrator 服务，请修改命令中的 consul 地址

```shell
docker run -d \
    --name=registrator \
    --net=host \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
      consul://localhost:8500
```



参数解析：

- --net=host 帮助Registrator获取到主机级别的ip和主机名
- --volume=/var/run/docker.sock:/tmp/docker.sock 允许Registrator访问docker的api接口

<!--more-->

## 测试

### Container 部署

运行一个 web 服务

```shell
docker run -d -p 3002:3000 --name myapp codebill/myapp:v1.0.7 
```



检查服务状态

```shell
curl 127.0.0.1:3002
{"version":"1.0.7","url":"/"}
```



查看consul endpoint 内容

```shell
curl http://127.0.0.1:8500/v1/catalog/services
{"consul":[],"consul-8500":[],"fabio":[],"fabio-9998":[],"fabio-9999":[],"myapp":[],"prometheus":[],"proxy-manager":["proxy-manager"]}
```



查看更详细的 endpoint 信息

```shell
curl http://127.0.0.1:8500/v1/catalog/service/myapp
[{"ID":"b66e8b6c-eb0e-8ac7-ed56-289fca4508cf","Node":"467c201ea1a1","Address":"172.23.0.2","Datacenter":"dc1","TaggedAddresses":{"lan":"172.23.0.2","lan_ipv4":"172.23.0.2","wan":"172.23.0.2","wan_ipv4":"172.23.0.2"},"NodeMeta":{"consul-network-segment":""},"ServiceKind":"","ServiceID":"ks-allinone:myapp:3000","ServiceName":"myapp","ServiceTags":[],"ServiceAddress":"","ServiceWeights":{"Passing":1,"Warning":1},"ServiceMeta":{},"ServicePort":3002,"ServiceEnableTagOverride":false,"ServiceProxy":{"MeshGateway":{},"Expose":{}},"ServiceConnect":{},"CreateIndex":1977,"ModifyIndex":1977}]
```



## 服务模型

container中监听的一个端口就是一个服务，如果监听多个端口就是多个服务。一个服务的创建，包括来自container的信息和用户在container上定义的元数据，这些信息被创建成一个服务对象。这个服务对象随后被传递给后端的注册中心，并尝试放置到一个特定的注册项。



### 容器覆盖

Name、Tags、Attrs、ID都是可以被用户定义的container元数据覆盖掉的，因为元数据被存储为环境变量或者标签，因此容器作者可以在Dockerfile中包含他们自己的元数据定义。

```
SERVICE_NAME=customerdb
SERVICE_80_NAME=api
SERVICE_REGION=us-east
```



### 实战

```
docker run -d -p 3003:3000 --name myapp-2 \
-e "SERVICE_80_NAME=myapp" \
-e "SERVICE_TAGS=proxy-koa" \
-e "SERVICE_REGION=hz" \
codebill/myapp:v1.0.7 
```

查看 consul 发现已经注册完成

![image-20201010180644521](https://tva1.sinaimg.cn/large/007S8ZIlly1gjkecne3vkj31c50e175m.jpg)

### 结合 fabio 实现负载均衡

删除之前创建的容器后，再创建2个新的容器

```
docker run -d -p 3003:3000 --name myapp-2 \
-e "SERVICE_80_NAME=myapp" \
-e "SERVICE_TAGS=urlprefix-a.com" \
-e "SERVICE_REGION=hz" \
codebill/myapp:v1.0.7 

docker run -d -p 3002:3000 --name myapp-1 \
-e "SERVICE_80_NAME=myapp" \
-e "SERVICE_TAGS=urlprefix-a.com" \
-e "SERVICE_REGION=hz" \
codebill/myapp:v1.0.7 
```

查看 consul 检查是否注册成功

![image-20201010181251428](https://tva1.sinaimg.cn/large/007S8ZIlly1gjkeiyxetdj319d0crjsp.jpg)

查看 fabio 查看路由规则是否生成



发现请求检查是否实现代理和负载均衡

```shell
curl -H "Host:a.com" http://127.0.0.1:9999
```

