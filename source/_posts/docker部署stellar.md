---
title: docker部署stellar
date: 2018-08-27 00:56:12
categories: 
- stellar
tags:
- docker
- 区块链
- stellar
---
参考文档：https://github.com/stellar/docker-stellar-core-horizon

#### 下载镜像，构建容器
```bash
//拉取 images ，根据配置启动 container
docker run --rm -it -p “ 8000：8000 ” -v “ / home / scott / stellar：/ opt / stellar ” - name stellar stellar / quickstart --testnet
```

#### 启动参数
- --pubnet:  正式网络
- --testnet:  测试网络
- --standalone:  私有网络

#### 访问正在运行的stellar容器
```bash
sudo docker exec -it stellar /bin/bash
```

<!--more-->

#### 重启服务
在 container (容器)中，可以使用 supervisord 来管理三个服务。
```bash
//前提已经进入对应的容器中，命令在上面
root@79e50b56739e:/# supervisorctl
horizon                          RUNNING   pid 22, uptime 3:19:32
postgresql                       RUNNING   pid 20, uptime 3:19:32
stellar-core                     RUNNING   pid 751, uptime 3:12:11

//停止
supervisor> stop stellar-core  

//重启
supervisor> restart horizon 

//开始
supervisor> start horizon 
```
#### 查看日志
```bash
//进入对应的容器中，存放在如下路径
/var/log/supervisor/
```
supervisord管理的进程输出的stdout和stderr分别保留中2个文件中。

#### 访问postgreSql数据库
- 账号：stellar
- 密码：第一次运行输入的密码
- 端口：5432

---

### 问题
Q：
```bash
$ docker run -d -v "/str:/opt/stellar" -p "8000:8000" --name stellar stellar/quickstart --pubnet

$ docker container ls -al
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                     PORTS               NAMES
7bda77b07a53        stellar/quickstart   "/init -- /start -..."   5 seconds ago       Exited (0) 4 seconds ago                       stellar

$ docker logs stellar
running `/start --pubnet'
pids are [5]

Starting Stellar Quickstart

mode: persistent
network: pubnet (Public Global Stellar Network ; September 2015)
postgres: config directory exists, skipping copy
supervisor: config directory exists, skipping copy
stellar-core: config directory exists, skipping copy
horizon: config directory exists, skipping copy
postgres user: stellar
exited 5
```

A:

删除启动时候映射的文件夹，再次启动。例如-v "/data/home/admin/stellar:/opt/stellar" 中的/data/home/admin/stellar。

Q：
```bash
winpty docker run --rm -it -p "15432:5432" -p "18000:8000" -p "11626:11626" - "c:/some/local/path:/opt/stellar" --name stellar tellar/quickstart --testnet
running `/start --testnet'
pids are [5]

Starting Stellar Quickstart

mode: persistent
network: testnet (Test SDF Network ; September 2015)
postgres user: stellar
Enter New Postgresql Password:
Confirm:
init-postgres: yes
ok
Waiting for postgres to be available...
Waiting for postgres to be available...
Waiting for postgres to be available...
Waiting for postgres to be available...
```

A:
将docker 升级到最新版本。