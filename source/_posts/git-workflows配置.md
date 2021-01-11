---
title: git-workflows配置
date: 2020-10-17 23:44:12
categories:
  - git
tag:
  - git
  - workflows
---
## 简介

github 在近期提供了一个 workflows 功能，可以轻松实现定制化工作流任务，包括经常使用的包括了 ci和cd ，从此不再需要通过第三方服务来实现。



通过 workflows 我们可以实现以下功能：

1. 构建和测试 Node.js
2. 构建和测试 Python
3. 使用 Maven 构建和测试 Java
4. 使用 Gradle 构建和测试 Java
5. 使用 Ant 构建和测试 Java
6. 发布 Node.js 模块包
7. 使用 Maven 发布 Java 软件包
8. 使用 Gradle 发布 Java 软件包
9. 发布 Docker 镜像
10. 缓存依赖文件



workflows 执行结果配置推送：

1. 个人邮箱
2. 如果您属于组织，则可以选择要将组织活动通知发送到的电子邮件帐户
3. 选择仅接收失败的工作流通知
4. 为移动设备启用推送通知

<!--more-->

参考文件：

- https://docs.github.com/en/free-pro-team@latest/actions/guides/building-and-testing-nodejs

点击 github 仓库 Actions 可以查看更多模板文件

![image-20201017232346339](https://i.loli.net/2020/10/17/JKNk3oU9IPOd7RH.png)

## 操作---node.js ci/cd

首先在项目下创建配置文件`.github/workflows/main.yml`，具体内容将在配置文件中解释

````yml
name: back-end/.TENCETN_CLOUD_HOST
on:
  push:
    branches:
    - master
    - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: 12
          registry-url: https://registry.npm.taobao.org
      - run: npm ci
      - run: npm test
        env: # 注入环境变量，可选
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

  deploy: # 持久化部署方案为，将代码拷贝到服务器，然后进行docker 部署

    runs-on: ubuntu-latest 
    steps:
      # 切换分支，获取源码
    - name: Checkout
      uses: actions/checkout@master
    - name: ssh deploy  
      uses: easingthemes/ssh-deploy@v2.1.4
      env:
    # Private Key  私钥
        SSH_PRIVATE_KEY: ${{secrets.BA_K8S}} # ssh 登录密钥保存位置，可以在仓库 setting secret 中进行配置
    # Remote host   远程主机地址
        REMOTE_HOST: "127.0.0.1" 
    # Remote user  远程主机登录用户
        REMOTE_USER: "root"
    # Remote port  远程主机登录端口
    #   REMOTE_PORT: 22
    # optional, default is 22
    # Source directory  拷贝源文件目录
        SOURCE: "./*"
    # optional, default is 
        ARGS: "-rltgoDzvO --delete"
    # Target directory  目标目录
        TARGET: "/opt/bin/back-end"

    - name: docker build and docker run 
      uses: garygrossgarten/github-action-ssh@release
      with:
        command: cd /opt/bin/back-end && docker rmi -f back-end:1.0.0 && cd /opt/bin/back-end && docker build -t back-end:1.0.0 . &&  docker run -d --name back-end -p 5000:5000 back-end:1.0.0
        host: "127.0.0.1"
        username: "root"
        privateKey: ${{ secrets.BA_K8S }}
````


可以在仓库的 readme.md 中配置 job 运行结果图表，格式如：![](https://github.com/组织或个人/仓库/workflows/执行Action的Name注意转码/badge.svg)



