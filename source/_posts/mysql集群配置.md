---
title: mysql集群配置
date: 2020-03-07 15:19:38
categories: 
- mysql
tags:
- 集群
---
## 简介
mysql集群配置，采用的是主从模式。可以将一个数据库的数据复制到另外一个数据库中，其中主数据库称为master,从数据库称为slave,关系为1对多。复制操作为异步，从数据库不需要一直连接着主数据库。

### 常见错误解决方案
1. 文档：https://blog.csdn.net/mbytes/article/details/86711508

### 优点
1. 实现高负载：采用读写分离，将一个数据库的压力进行分担
2. 提高安全性：多个从数据库对主数据库的数据进行备份，防止主数据库数据丢失

<!--more-->

## 实现原理图
![image](https://upload-images.jianshu.io/upload_images/4053714-bf6f27fecff034bc.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

### 简述
1. 开启master binary log，数据库的修改操作都将记录到log中（二进制日志事件）
2. slave 的 I/O 线程会读取matser binary log ，并且写入到 relay log 中（中继日志事件）
3. slave 的 sql 线程会读取 relay log 中的操作，并且写入到数据库中

### 主从同步延迟原因
1. 主库执行大批量的未有索引表的修改操作
2. 从数据库配置不足
3. 在mysql5.7之前，从数据库 sql thread 单线程进行同步
4. 同步为异步串行，只有在主数据库提交事务之后，从数据库才开始进行同步，并且同步的过程也需要消耗时间

### 主从同步延迟解决办法
1. 提升从数据库配置
2. mysql 5.7 开启并行复制
3. 业务层在进行操作的时候，先进行查询再进行修改，避免sql执行失败

## 配置
### 准备工作
1. 准备2台服务器，并且数据库服务启动正常
    1. 主(windoes)ip:192.168.100.107
    2. 从(centos)ip:192.168.100.117
2. 关闭2台服务器防火墙
3. 2台服务器网络连接正常

### 主数据库配置
数据库配置文件修改 my.ini,添加如下配置内容
```
#id编号
server_id=1
#减少日志记录数量，只记录受影响的列
binlog_row_image = minimal
# 二进制日志存放路径
log-bin="D:\Program Files\mysql-5.7.28-winx64\data\mysql-bin"
# 错误记录日志文件
log-error="D:\Program Files\mysql-5.7.28-winx64\data\mysql-error"
#(可选)主从同步忽略的数据库
#binlog-ignore-db=mysql
#(可选)指定同步时只同步的数据库
binlog-do-db=demo
#binlog记录内容的方式，记录被操作的每一行
binlog_format=ROW
#日志的缓存时间
expire_logs_days=10                    			
#日志的最大大小
max_binlog_size=200M                        
```

修改完毕后，重启mysql服务使配置生效，启动完毕后，需要观察二进制日志存放路径，是否产生对应文件
```
net stop mysql
net start mysql
```



数据库权限配置
```sql
/*创建远程连接账号*/
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' IDENTIFIED BY '123456';

flush privileges;

/*查看主数据库信息*/
show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 | demo         |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

### 从数据库配置
数据库配置文件修改,添加如下配置内容
```
vim /etc/my.cnf

[mysqld]
# 配置slave id
server-id=117
# 选择同步的数据库
replicate_do_db=demo
# 日志过期时间
expire_logs_days=10
# 二进制日志最大大小
max_binlog_size=200M
```

重启MySQL服务
```shell
$ service mysqld restart

# 进入数据库
$ mysql -u root -p

# 配置同步信息，其中master_log_file，master_log_pos 根据上文master status输出内容决定
mysql> change master to master_host='192.168.100.107',master_port=3306,master_user='test',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=154;

# 启动同步
mysql> start slave;
# 查看同步状态
mysql> show slave status\G
Slave_IO_Running: Yes
Slave_SQL_Running: No
Replicate_Do_DB: demo
Last_Errno: 0
Last_Error: 
Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
```
当如下参数不正确时候，需要查看Last_Errno，Last_Error，Slave_SQL_Running_State输出的信息：
1. Slave_IO_Running: Connecting
2. Slave_SQL_Running: YES
3. Replicate_Do_DB: demo

此时的状态为正在等待获取更多的的操作信息，说明matser为进行任何操作。

### 同步测试
#### 主数据库
```sql
/*创建数据库*/
create database demo;

use demo;

create table book(author varchar(11));

insert into book values("ddz");
```

#### 从数据库
```sql
show databases;

use demo;

select * from book;
+--------+
| author |
+--------+
| ddz    |
+--------+
```

数据同步完成