---
title: 搭建一个完整的docker生态
date: 2020-10-08 23:03:20
categories:
  - docker
tags:
  - docker
---

## 概要

通过本文你可以学习到以下内容：

1. 快速安装 docker 和 docker-compose
2. 如何使用 docker 可视化管理平台
3. 如何对 docker 容器进行监控和报警

## 环境安装

### ubuntu

```shell
apt-get update
apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

apt-get update

apt-get -y install docker-ce docker-ce-cli containerd.io
```

### centos

```shell
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install -y docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
```

### 修改镜像源和默认资源存储位置

```shell
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://7a1tnjfc.mirror.aliyuncs.com","http://hub-mirror.c.163.com"],
  "data-root": "/media/nvme/docker_data"
}

# 重启
service docker restart
```

### docker-compose 部署

要安装其他版本的 Compose，请替换 1.24.1。

```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

$ docker-compose --version
```

<!--more-->

## 常用指令

一般的拉取镜像，构建容器，进入容器内部，查看容器日志等指令，这里就不再多提，分享一些不常用但是非常高效的指令

```shell
# 删除那些已停止的容器、dangling 镜像、未被容器引用的 network 和构建过程中的 cache
$ docker system prune

# 删除所有未被引用的镜像文件
$ docker images prune

# 相当在 container 退出的时候运行 docker rm -v，删除容器匿名数据卷
$ docker run --it --rm busybox

# 和宿主机使用同一个 network 和 pid namespace
$ docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter \
  --path.rootfs=/host

# 查看docker和宿主机状态
$ docker info

# 查看容器运行状态和资源消耗
$ docker stats

# 查看docker 资源使用情况
$ docker system df

# 删除未被容器引用的数据卷
$ docker system prune -a -f --volumes

# 容器导出
$ docker save -o myapp.tar codebill/myapp:latest

# 容器导入
$ docker import nginx-test.tar nginx:imp

# 镜像导入
$ docker save -o nginx.tar nginx:latest

# 镜像导出
$ docker load -i nginx.tar
```

## 可视化管理平台搭建

### 部署

```shell
$ docker pull portainer/portainer

$ docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name Prtainer portainer/portainer
```

open url: http://localhost:9000/#/init/admin , 第一次登陆需要进行密码修改，以下为控制面板主要内容介绍，点击可查看详情

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gji37l9gb5j31kg0u0don.jpg" alt="image-20201008181011358" style="zoom:50%;" />

### 操作案例：拉取镜像

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gji3a1bu3lj31ke0u0tij.jpg" alt="image-20201008181232548" style="zoom:50%;" />

### 操作案例：构建容器

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gji3cdyabej31kr0u0qax.jpg" alt="image-20201008181448647" style="zoom:50%;" />

<img src="/Users/bill/Library/Application Support/typora-user-images/image-20201008181827708.png" alt="image-20201008181827708" style="zoom:50%;" />

## 可视化性能监控平台搭建

### prometheus 部署

配置 prometheus 从 cadvisor 抓取数据指标，创建 prometheus.yml

```yml
scrape_configs:
  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
      - targets:
          - cadvisor:8080
```

创建 docker-compose.yml

```yml
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - 9090:9090
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
      - cadvisor
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
      - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
      - 6379:6379
```

此配置指示 Docker Compose 运行三个服务，每个服务对应一个 Docker 容器：

1. prometheus 将使用本地 prometheus.yml configuration file (通过 volumes 参数导入到容器中)
2. cadvisor 服务公开端口 8080（cAdvisor 指标的默认端口），并依赖于各种本地卷（/，/ var / run 等）。
3. Redis 服务是标准的 Redis 服务器。cAdvisor 将自动从该容器收集容器指标，即无需任何进一步配置。

```shell
# 启动
$ docker-compose up -d

$ docker-compose ps
   Name                 Command               State           Ports
----------------------------------------------------------------------------
cadvisor     /usr/bin/cadvisor -logtostderr   Up      0.0.0.0:8080->8080/tcp
prometheus   /bin/prometheus --config.f ...   Up      0.0.0.0:9090->9090/tcp
redis        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
```

cAdvisor web UI : http://localhost:8080

![image-20200820141548791](https://i.loli.net/2020/08/20/N6ri9eabc2Kk8PO.png)

查看指定容器的统计信息和图表 : http://localhost:8080/docker/<containerName> ，例如:

- redis http://10.10.0.11:8080/docker/redis
- prometheus http://10.10.0.11:8080/docker/prometheus

Prometheus web UI: http://10.10.0.11:9090/graph

<img src="/Users/bill/Library/Application Support/typora-user-images/image-20201008182635257.png" alt="image-20201008182635257" style="zoom:50%;" />

使用 PromQL Demo

| Expression                                                | Description                                                                        | For                   |
| :-------------------------------------------------------- | :--------------------------------------------------------------------------------- | :-------------------- |
| rate(container_cpu_usage_seconds_total{name="redis"}[1m]) | The [cgroup](https://en.wikipedia.org/wiki/Cgroups)'s CPU usage in the last minute | The `redis` container |
| container_memory_usage_bytes{name="redis"}                | The cgroup's total memory usage (in bytes)                                         | The `redis` container |
| rate(container_network_transmit_bytes_total[1m])          | Bytes transmitted over the network by the container per second in the last minute  | All containers        |
| rate(container_network_receive_bytes_total[1m])           | Bytes received over the network by the container per second in the last minute     | All containers        |

### grafana 部署

```shell
$ docker run -d --name=grafana -p 3000:3000 grafana/grafana

$ docker ps | grep "grafana"
eb5bfe364689        grafana/grafana                                           "/run.sh"                58 seconds ago      Up 56 seconds       0.0.0.0:3000->3000/tcp   grafana
```

open web ui : http://localhost:3000/login 账号 admin 密码 admin，第一次登录需要修改密码

![image-20201008182751973](https://tva1.sinaimg.cn/large/007S8ZIlly1gji3q0ww2tj31cr0saqcj.jpg)

#### 配置数据源

<img src="/Users/bill/Library/Application Support/typora-user-images/image-20201008182938941.png" alt="image-20201008182938941" style="zoom:50%;" />

![image-20201008183002130](https://tva1.sinaimg.cn/large/007S8ZIlly1gji3s8gyf4j31u009ht9r.jpg)

<img src="/Users/bill/Library/Application Support/typora-user-images/image-20201008183109500.png" alt="image-20201008183109500" style="zoom:50%;" />

#### 配置数据面板
