---
title: ionic简介
date: 2018-08-30 00:50:03
categories: ionic
tags:
- ionic
---
#### 前端基础
- Angular4.0
- TypeScript
- HTML
- CSS

#### 简介
ionic主要职责是作为 app 的前端UI框架，提供基本的样式以及各种UI组件。

结合Cordova / PhoneGap插件和TypeScript扩展，支持120多种本机设备功能，如蓝牙，HealthKit，指纹识别等。

##### 原理
用户打开一个ionic应用，首先设备加载cordova应用封装器，然后cordova应用封装器加载webview，webview加载index.html文件，最后angular加载并确定默认视图，ionic渲染ionic组件作为UI。

![image|694x204](/images/ionic-yl.png)

其中cordova的任务是实现浏览器窗口和原生API间的通信。

这个过程是Angular控制器使用Cordova JavaScript API调用Cordova，Cordova使用原生SDK和设备通信。

<!--more-->

###### WebView
WebView(网络视图)能加载显示网页，可以将其视为一个浏览器。它使用了WebKit渲染引擎加载显示网页。

##### 优势
它一次开发，多个平台部署，能够最小化开发成本，它使用web技术开发，又能访问原生API。

##### 劣势
性能上由于依赖于webview所以性能比不上原生应用，原生功能的访问也取决于相应的插件有没有被开发出来或者其他方法。

Cordova 封装的控件UI，需要通过 java 或者 Obj-C进行修改。
**性能：native app >react native app> hybrid app**

---

#### 安装
```
npm install -g cordova ionic

//检测安装版本
ionic -v
3.19.1
```
