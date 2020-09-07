---
title: ionic项目结构
date: 2018-08-31 00:47:56
categories: ionic
tags:
- ionic
---
### 创建项目
```bash
//创建基础项目
ionic start MyIonicProject tutorial
```

#### 根目录
```bash
.sourcemaps
node_module ---node 依赖包
platforms ---生成android和ios安装包位置
plugins  ---Cordova安装的插件
resources  ---android/ios资源(更换图标和启动动画)
src  ---开发工作目录(最主要)
www ---静态文件
.editorconfig ---ide编码风格配置文件
.gitignore  ---git忽略文件
config.xml  ---配置文件
ionic.config.json
package-lock.json
package.json  ---项目信息
tsconfig.json ---TypeScript 配置信息
tslint.json  ---格式化和校验 TypeScript
```

<!--more-->

 #### src
 ```bash
│  declarations.d.ts
│  index.html ---应用程序的主要入口,加载js,css和引导程序
│  manifest.json ---APP配置文件,如配置启动文件,icons
│  service-worker.js
│
├─app ---应用根目录
│      app.component.ts ---逻辑
│      app.html  ---模版
│      app.module.ts ---应用程序根模块
│      app.scss ---选择器
│      main.ts ---设置根模块
│ 
├─assets ---资源目录（静态文件（图片，js 框架等）
│  ├─icon
│  │      favicon.ico
│  │
│  └─imgs
│          logo.png
│
├─pages ---页面文件，放置编写的页面文件
│  ├─hello-ionic
│  │      hello-ionic.html
│  │      hello-ionic.scss
│  │      hello-ionic.ts
│  │
│  ├─item-details
│  │      item-details.html
│  │      item-details.scss
│  │      item-details.ts
│  │
│  ├─list
│  │      list.html
│  │      list.scss
│  │      list.ts
│  │
│  └─login
│          login.html
│          login.module.ts
│          login.scss
│          login.ts
│
└─theme
        variables.scss ---主题配置文件
 ```