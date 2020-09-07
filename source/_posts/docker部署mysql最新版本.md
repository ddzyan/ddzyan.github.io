---
title: docker部署mysql 5.7版本
date: 2018-08-15 02:36:38
abbrlink: docker_mysql_inatll
categories:
  - mysql
tags:
  - mysql
  - docker
---

#### mysql

目录结构

```shell
── mysql
    ├── conf
    │   └── my.cnf
    └── init
        └── init.sql
```

<!--more-->

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
