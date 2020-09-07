---
title: nodejs生成脚手架
date: 2019-08-21 23:36:58
categories:
  - node
tags:
  - node
---

## 资料

- 代码示例仓库：https://github.com/ddzyan/chain-cli
- 参考文章
  - https://github.com/lin-xin/blog/issues/27
  - https://github.com/ykfe/koa-generator#readme

## 本文目的

- 了解脚手架是什么，有什么作用
- 如何使用 nodejs 创建一个脚手架模块

### 脚手架

脚手架是创建基础代码框架的工具。

脚手架可以通过交互模式（需要开发支持）快速生成基础代码模板，避免从零开始导致的时间浪费，并且利于团队使用。

### 代码分析

涉及的代码，请到示例仓库中获取

#### 依赖

- commander：解析用户输入的指令，并且执行相关操作
- download-git-repo：下载和提取 github 代码，并且放到指定文件夹内
- handlebars：替换模板内的指定内容，用于创建指定文件
- ora：下载进度动画，避免长时间拉取代码无反馈
- chalk：使控制台输出的内容，可以设置颜色，方便识别
- inquirer：交互式获取用户输入信息，用户创建指定文件
- log-symbols：可以在终端上显示出 √ 或 × 等的图标

<!--more-->

#### 备注

需要在入口文件上输入以下内容，使系统指定要用什么程序去启动，与 python 代码类似，否则可能出现如下报错
![image](/images/脚手架报错linux.png)
![image](/images/脚手架报错windows.png)

```js
#!/usr/bin/env node
```
