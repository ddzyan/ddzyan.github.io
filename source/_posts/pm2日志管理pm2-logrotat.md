---
title: pm2日志管理pm2-logrotat
date: 2018-08-17 02:33:14
categories: 
- nodejs
tags:
- node_module
- nodejs
---
参考网址：https://github.com/keymetrics/pm2-logrotate
## 简介
就是开个线程对pm2的日志进行监控和操作。

以下测试版本为:
- pm2:3.0.3
- pm2-logrotate:2.6.0

请确保版本对应，否则不保证操作顺利，欢迎留言解决。

## 安装

```
pm2 install pm2-logrotate

//安装指定版本
pm2 install pm2-logrotate@2.6.0
```

<!--more-->

## 启动

```
//带参数
pm2 set pm2-logrotate:<param> <value>

pm2 set pm2-logrotate:compress true
```

## 参数
- max_size（默认为10M）：日志文件大小，超过指定大小将重头写入
- retain（默认为30个文件日志）：日志文件数量
- compress（默认为false）：是否进行压缩
- dateFormat（默认为YYYY-MM-DD_HH-mm-ss）：文件格式
- rotateModule（默认为true）：重复写日志
- workerInterval（默认为30秒）检查时间间隔
- TZ（默认为系统时间）：记录的时间

## 注意
- 每次只能设置一个选项
- 可以通过pm2 set 修改配置文件达到一次修改多个属性
- 一次设定将对所有pm2监听的服务起效

pm2 配置文件路径 ~/.pm2/module_conf.json
```
{
    "module-db-v2": {
        "pm2-logrotate": {}
    },
    "pm2-logrotate": {
        "workerInterval": "3600",
        "max_size": "100M",
        "retain": "5"
    }
}
```
--- 
# 以下未验证
## 对系统进行统一设置
```bash
sudo pm2 logrotate -u user
```

### 配置文件
文件路径：/etc/logrotate.d/pm2-user

添加如下配置, /home/user/.pm2/pm2.log 为监控的日志路径
```bash
/home/user/.pm2/pm2.log /home/user/.pm2/logs/*.log {
        rotate 10
        daily
        missingok
        notifempty
        compress
        delaycompress
        copytruncate
        create 0640 admin admin
        size 1024M
}
//修改完配置文件后，需要重启对应pm2服务
//实现的功能有
保留10个文件
每日保存一份


空文件不转存
压缩旧日志
转存的日志到下一次转存再压缩
文件超过1024M就备份一份

```
参考网址：http://www.ttlsa.com/linux/logrotate-log-management-tools/

参数：
- compress：通过gzip 压缩转储旧的日志
- nocompress：不需要压缩时，用这个参数
- copytruncate：用于还在打开中的日志文件，把当前日志备份并截断
- nocopytruncate：备份日志文件但是不截断
- create mode owner group：使用指定的文件模式创建新的日志文件
- nocreate：不建立新的日志文件
- delaycompress：和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
- nodelaycompress：覆盖 delaycompress 选项，转储同时压缩。
- errors address：专储时的错误信息发送到指定的Email 地址
- ifempty：即使是空文件也转储，这个是 logrotate 的缺省选项。
- notifempty：如果是空文件的话，不转储
- mail address：把转储的日志文件发送到指定的E-mail 地址
- nomail：转储时不发送日志文件
- olddir directory：转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
- noolddir：转储后的日志文件和当前日志文件放在同一个目录下
- prerotate/endscript：在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
- postrotate/endscript：在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行
- sharedscripts：所有的日志文件都轮转完毕后统一执行一次脚本
- daily：指定转储周期为每天
- weekly：指定转储周期为每周
- monthly：指定转储周期为每月
- rotate count：指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
- size size：当日志文件到达指定的大小时才转储，Size 可以指定 bytes (缺省)以及KB (sizek)或者MB