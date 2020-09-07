---
title: yarn快速上手
date: 2018-09-13 00:43:02
categories: 
- nodejs
tags:
- nodejs
- node_module
---
### 简介
与 npm 相同的依赖包管理工具

官网：https://yarnpkg.com/zh-Hans/

#### 特点
- 离线模式：如果你之前安装过某个包，你就可以在没有网络连接的情况下再次安装它。
- 确定性：不管是什么顺序，在不同的机器上的依赖会以同一方式安装。
- 相同软件包：从 npm 安装软件包并使用相同的包管理流程。
- 网络性能：Yarn可以高效地队列化请求并且避免请求瀑布化，使网络利用率最大化。

#### 安装
参考网址：https://yarnpkg.com/zh-Hans/docs/install#windows-stable

用 Chocolatey 安装
```bash
choco install yarn
```

##### 检测
```bash
yarn --version
```

<!--more-->

### CLI
#### 基础
```bash
//初始化项目
yarn init

//添加依赖包
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]

//将依赖项添加到不同依赖项类别
yarn add [package] --dev //开发环境
yarn add [package] --peer
yarn add [package] --optional

//升级依赖包
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]

//移除
yarn remove [package]

//安装全部
yarn
yarn install

//全局安装
yarn global <add/bin/list/remove/upgrade> [--prefix]

//显示安装包列表
yarn list

//显示全局安装列表
yarn global list
```

#### yarn run
```bash
//运行用户自定义脚本
yarn run [script] [<args>]

//显示所有可运行的脚本
yarn run

//执行该命令将会会列出脚本运行时可用的环境变量
yarn run env

//快捷指令
yarn [script] [<args>]
```

### 更换镜像源
```bash
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```

### 管理镜像源工具
yarm的注册管理平台

yrm可以帮助您轻松快速地在不同的npm注册表之间切换，现在包括：npm，cnpm，taobao，nj（nodejitsu），rednpm，yarn。

github地址：https://github.com/i5ting/yrm