---
title: node常用日志模块简介
date: 2018-08-28 00:18:32
categories: 
- nodejs
tags:
- node_module
- nodejs
---
### 简介
本文介绍了在 nodejs 服务中常用的日志模块，方便大家选择和快速上手。

### log4js
下载量：1168793

包含的功能：
- 能在控制台彩色输出stdout和stderr
- file appender 可以根据文件大小或者日期时间来配置日志的回滚
- 自定义日志消息布局/模式
- 日志作为电子邮件发送
- 多台服务器日志设置发送到服务器集中记录
- 可以设置不同级别的日志
- 可以自定义日志文件名称

参考文档地址：https://log4js-node.github.io/log4js-node/

<!--more-->

### winston
下载量：2661692

包含的功能：
- 多台服务器日志设置发送到服务器集中记录
- 自定义日志消息
- 可以自定义日志文件名称
- 可配置插件进行拓展

常用插件：
- winston-mongodb：日志输出到mongodb数据库
- winston-mail：日志作为电子邮件发送
- winston-daily-rotate-file：日志根据日期回滚

快速上手：https://github.com/ddzyan/node-module-example/tree/master/winston

参考文档地址：https://github.com/winstonjs/winston#readme

### pm2
下载量：213473

包含的功能：
- 可以将程序的的 console.error,console.info 输出到指定日志文件中
- 可以自定义日志文件名称
- 可以通过安装：pm2-logrotat 实现根据文件大小或者日期时间回滚日志

参考文档地址：http://pm2.keymetrics.io/

---

### 区别
三者该有的功能的功能都有，可根据具体需求选取。比如对日志文件名称有特殊要求的可以选择log4js，因为log4js过期日期名称可以选择为日期或者其他格式，但是正在写入的文件可以单独命名。

#### 日志回滚
pm2：功能最差，需要启动 pm2-logrotat 服务，根据设置的检查时间，每隔一段时间去检测日志文件信息，达到指定要求则新建文件进行写入。

winston：需要下载安装 winston-daily-rotate-file 插件实现日志回滚。

log4js：无需下载安装插件，自带日志回滚功能。

#### 自定义日志信息
pm2：只能通过 console 方式，使用起来比较麻烦

winston：通过自定义返回值和参数配置组合模式，方便简洁

log4js：采用 layout 模版模式，使用起来也非常简单。
