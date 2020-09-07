---
title: pm2快速上手
date: 2018-08-30 01:36:36
categories: 
- nodejs
tags:
- node_module
- nodejs
---
参考文档：https://www.npmjs.com/package/pm2

测试demo: https://github.com/ddzyan/node-module-example/tree/master/pm2

### 简介
pm2 是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等

### 安装
```bash
//全局安装
npm i pm2 -g

//更新
pm2 update
```

<!--more-->

### 常用命令
#### 管理应用程序
```bash
//查看所有服务信息
pm2 list

//监控所有服务详细信息包括日志
pm2 monit

pm2 start    <file.js|json_conf>
pm2 [stop/restart/reload/delete]  <app_name|id|'all'|json_conf>

pm2 [start/stop/restart/reload/delete] ecosystem.config.js --only worker-app
```
##### 参数
- file.js - 服务启动文件
- json_conf - pm2 配置文件
- app_name - 指定 pm2 服务名称
- id - 指定 pm2 id
- all 全部服务
- json_conf 配置文件中的全部服务
- --only - 只操作配置文件中的指定服务

#### 添加自启
```
//添加系统启动
pm2 startup centos

//保存
pm2 save
```

#### 查看日志
```
//查看全部服务
pm2 logs 

//查看指定 APP-NAME  服务日志
pm2 logs APP-NAME 

//重新加载日志
pm2 reloadLogs

//清空全部日志
pm2 flush
```

----

### 配置文件模版
参考文档：http://pm2.keymetrics.io/docs/usage/cluster-mode/

在项目根目录下，初始化配置文件
```bash
pm2 init
```

ecosystem.config.js
```js
module.exports = {
  apps: [{
    name: 'test',//进程名称
    script: 'index.js',//启动脚本
    watch: false, // 默认关闭watch 可替换为 ['src']
    log_date_format: 'YYYY-MM-DD HH:mm Z',//日志格式
    ignore_watch: ['node_modules', 'public', 'logs'],
    out_file: './pm2-server-out.log', // 日志输出地址
    error_file: './pm2-server-error.log', // 错误日志地址
    max_memory_restart: '1G', // 超过多大内存自动重启，仅防止内存泄露有意义，需要根据自己的业务设置
    env: {
      NODE_ENV: 'production'//环境变量,在服务中通过 process.env.NODE_ENV 获取
    },
    exec_mode: 'fork', // 开启多线程模式，用于负载均衡
    instances: '1', // 启用多少个实例，可用于负载均衡
    autorestart: true // 程序崩溃后自动重启
  }],
  deploy: {
    production: {
      user: 'node',
      host: '212.83.163.1',
      ref: 'origin/master',
      repo: 'git@github.com:repo.git',
      path: '/var/www/production',
      'post-deploy': 'npm install && pm2 reload ecosystem.config.js --env production'
    }
  }
};
```
#### 参数
绝大部分参数已经在模版中注释

- apps - 服务配置信息
- deploy - 同步信息，可以将pm2配置文件同步到指定服务器

日志文件监控，根据文件大小或者日期进行切割或者回滚，请参考：http://www.zmscode.cn/2018/08/17/pm2日志管理pm2-logrotat/