---
title: docker-compose实战操作
date: 2019-08-22 23:14:17
categories:
  - docker
tags:
  - docker
---

## 参考资料

- https://blog.csdn.net/hjxzb/article/details/84927567#6_131
- https://docs.docker.com/compose/install/#install-compose

### 了解概念

- 作用
  - 快速拉起一个服务，这个服务可以使一个容器或者多个容器组成
  - 编排管理容器
  - 定义和运行复杂 docker 镜像,避免使用 shell 指令创建

<!--more-->

### 安装

step1 确认内核版本

```bash
uname -r
//输出的linux内核版本是3.10以上并且是64位的centos版本，才能支持安装。
```

step2 下载最新版本安装包

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

step3 修改二进制文件权限

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

step3 测试

```bash
docker-compose --version
//输出版本号则正确
```

### 卸载

```shell
//使用curl安装
sudo rm /usr/local/bin/docker-compose

//使用pip安装
sudo pip uninstall docker-compose
```

### 操作

在执行指令的目录下必须存在 docker-compose.yml 文件,指令只会对配置文件内的服务或者容器起效

```shell
# 运行指定文件 默认则为 docker-compose.yml,下面所有指令都适用
docker-compose -f ./docker-compose-debug.yml up -d

# 自动完成包括构建镜像、创建服务、启动服务并关联服务相关容器的一系列操作
docker-compose up -d

# 删除服务内的所有容器和镜像
docker-compose down

# 显示服务内的所有容器
docker-compose ps

# 构建
docker-compose build

# 实时查看日志，加上容器名称则只看该容器日志，否则查看服务内所有容器的日志
docker-compose logs -f

# 暂停/恢复/启动/停止 服务
docker-compose  pause/unpause/start/stop

# 删除前必须停止
docker-compose rm

# 进入容器
docker-compose exec mysql bash
```

### docker-compose.yml 参数

- 卷:volumes
- 端口:ports
- 环境变量:environment
- 容器名称:container_name
- 重启策略:restart
- 镜像:image
- 命令，将覆盖镜像文件内的 CMD:command

### 示例

#### 从 dockerfile 构建容器

docker-compose.yml

```yml
version: '3'

services:
  back_end:
    build:
      context: ./
      dockerfile: DockerFile
    image: back_end
    environment:
      NODE_ENV: production
    ports:
      - 3000:3000
```

#### redis

docker-compose.yml

```
version: "3"

services:
  redis:
    container_name: redis
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - "./conf/redis.conf:/etc/redis/redis.conf"
      - "./data:/data"
    restart: always
    command: redis-server /etc/redis/redis.conf
```

#### mysql

目录结构

```shell
── mysql
    ├── conf
    │   └── my.cnf
    └── init
        └── init.sql
```

##### 创建配置文件

```shell
vim ./mysql/conf/my.cnf
[mysqld]
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
datadir     = /var/lib/mysql
#log-error  = /var/log/mysql/error.log
# By default we only accept connections from localhost
bind-address   = 0.0.0.0
# Disabling symbolic-links is recommended to prevent assorted security risks
#symbolic-links=0
```

初始化 init.sql ，在容器创建完成后将执行

```sql
vim ./mysql/conf/init.sql

use mysql;
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';
FLUSH PRIVILEGES;

```

- 添加远程账号
- 创建基础数据库

docker-compose.yml

```yml
version: '3'

services:
  mysql:
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    image: mysql:5.7
    ports:
      - '3306:3306'
    volumes:
      - './mysql/conf/my.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf'
      - './mysql/logs:/logs'
      - './mysql/data:/var/lib/mysql'
      - './mysql/init:/docker-entrypoint-initdb.d/'
    restart: always
```
