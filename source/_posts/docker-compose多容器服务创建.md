---
title: docker-compose多容器服务创建
date: 2019-08-22 23:16:43
categories:
  - docker
tags:
  - docker
---

# docker-compose 多容器服务

## 本文目的

- 了解如何适用 docker-compose 编排多容器服务
- 在容器中，如何访问其他容器网络

## 服务用例

下载项目：https://github.com/ddzyan/backend_docker-compose.git

### 项目文件夹结构

```shell
$ tree -L 2
.
├── docker-compose.yml
└── mysql
    ├── conf
    │   └── my.cnf
    └── init
        └── init.sql
```

<!--more-->

### 配置文件介绍

#### my.cnf

mysql 数据库配置文件

```
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

#### init.sql

mysql 初始化 sql,目的如下

- 创建远程连接账号并且赋予权限，给 back_end 服务连接使用
- 创建 back_end 必需的数据库和表

```
use mysql;
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';
FLUSH PRIVILEGES;
create database backend;

use backend;

CREATE TABLE `classroom` (
  `id` bigint(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `classname` varchar(13) NOT NULL COMMENT '班级名称',
  `headTeacherId` bigint(10) unsigned DEFAULT NULL COMMENT '班主任ID',
  `createdAt` datetime DEFAULT NULL COMMENT '创建时间',
  `updatedAt` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `classname_index` (`classname`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

CREATE TABLE `user` (
  `id` bigint(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` varchar(13) NOT NULL COMMENT '账号名称',
  `age` int(2) NOT NULL COMMENT '年龄',
  `password` varchar(20) NOT NULL COMMENT '密码',
  `sex` enum('1','0') NOT NULL COMMENT '性别',
  `classroomId` bigint(10) unsigned NOT NULL COMMENT '班级ID',
  `createdAt` datetime DEFAULT NULL COMMENT '创建时间',
  `updatedAt` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `classroomId` (`classroomId`),
  KEY `username_index` (`username`) USING BTREE,
  CONSTRAINT `user_ibfk_1` FOREIGN KEY (`classroomId`) REFERENCES `classroom` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表'
```

#### docker-compose.yml

```yml
version: '3'

services:
  backend:
    container_name: ddz_backend
    image: registry.cn-hangzhou.aliyuncs.com/zmscode/backend:1.0.0
    environment:
      NODE_ENV: production
    ports:
      - 3000:3000
    volumes:
      - './back_end/logs:/usr/src/back_end/logs'
    command: pm2-runtime start ./start.config.js
    depends_on:
      - mysql
      - redis

  mysql:
    container_name: ddz_mysql
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

  redis:
    container_name: ddz_redis
    image: redis:latest
    ports:
      - '6379:6379'
    volumes:
      - './redis/conf/redis.conf:/usr/local/etc/redis/redis.conf'
      - './redis/data:/data'
    restart: always
    command: redis-server /usr/local/etc/redis/redis.conf
```

##### 参数说明

###### back_end

- command 将覆盖 dockerfile 文件中的 CMD 指令
- depends_on 设置 back_end 容器启动在 mysql 和 redis 之后，保证服务启动之前数据库和缓存容器已经创建成功

###### mysql

- environment 系统变量设置，配置 root 账号密码
- volumes 容器卷映射 ，其中 init 文件夹映射的目的，是在 mysql 容器创建成功之后，执行初始化 sql
- restart 重启策略，出问题就重启

### 使用说明

back_end 项目中连接 mysql 和 redis ，将不采用 127.0.0.1 或者其他指定 IP 方式，将采用如下方式

```js
url: 'redis://redis:6379/1';
```

原因是在服务创建的时候，docker-compose 会创建一个默认的网络，配置的所有容器都将在这个网络下创建和互相访问，访问的方式为 容器名称 进行，docker-compose 会进行解析为指定的 ip，用于连接

### 服务操作

```shell
# 构建，启动
docker-compose -f ./docker-compose.yml up

# 关闭，删除容器
docker-compose -f ./docker-compose.yml down

# 查看容器
docker-compose -f ./docker-compose.yml ps

# 查看日志
docker-compose -f ./docker-compose.yml logs -f
```
