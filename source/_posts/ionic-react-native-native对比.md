---
title: 'ionic,react-native,native对比'
date: 2018-09-10 08:15:06
categories: 
- 随笔
tags:
- 随笔
---
### 简介
本文从三者的跨平台性，开发方式，系统功能支持，性能对比，实现原理和优劣势分析三者的差异，并不能说明某个框架有着绝对意义的优势，选择使用哪个框架往往取决于项目的大小，开发人员所具备的技能，开发时间的长短,对性能的要求等。

**umijs为 一个可插拔的企业级 react 应用框架 ，界面渲染使用的是React DOM，暂时不支持react native**

weex 由于使用人员和文档较少，暂不加入对比

#### 跨平台
#####  React Native
由于react-native 模块提供的 android 和 ios UI 组件和系统 API 上的使用方法不同，需要使用 jsx 各自编写 1 套 IOS 和 android 代码

##### Ionic
不涉及到特殊的系统 API 开发(IOS 支持 android 不支持)，一套代码可以支持 IOS 和 android。

绝大多数 UI 组件和系统 API 都适配两个系统。

##### native
用 java 开发android

用 oc 开发 ios

---

<!--more-->

#### 开发方式
##### React Native：jsx + react-native
JS语言中嵌入了HTML和CSS的元素，这种被扩展了的JavaScript语言称为jsx。通过 jsx 调用 react-native 模块内的组件和API，实现页面搭建和系统功能实现。

###### 代码示例
```jsx
import React, { Component } from 'react';
import { Text, View } from 'react-native';

export default class HelloWorldApp extends Component {
  render() {
    return (
        <View>
          <Text>Hello world!</Text>
        </View>
    );
  }
}
```

###### UI
参考地址：https://reactnative.cn/docs/activityindicator/

- react-native提供了三种组件和API：基础组件，IOS特有组件，Android特有组件，绝大多数需要分别调用
- 可用 flexbox 布局



##### Ionic：html + angularjs + css + cordova
css + html 实现页面搭建，angularjs 实现数据动态更新，cordova 提供系统 API 给JS调用

###### 代码示例
```html
<ion-menu [content]="content">

  <ion-header>
    <ion-toolbar>
      <ion-title>Pages</ion-title>
    </ion-toolbar>
  </ion-header>

  <ion-content>
    <ion-list>
      <button ion-item *ngFor="let p of pages" (click)="openPage(p)">
        {{p.title}}
      </button>
    </ion-list>
  </ion-content>

</ion-menu>

<ion-nav [root]="rootPage" #content swipeBackEnabled="false"></ion-nav>

```

###### UI
参考地址：http://www.ionic.wang/css_doc-v3.html

- 提供丰富的 UI 组件，绝大多数适配2种平台
- 可用 flexbox 布局



##### Native：java + oc|swift
iOS android 不同语言开发以及适配

###### UI
UI效果最好

---

#### 系统 API 支持性
##### React Native
可以通过原生代码编写 API 注册到 react-native 中实现全部支持

##### ionic 
可以通过原生代码编写 cordova 实现全部支持

##### native
全部支持

---

#### 性能对比
##### React Native
接近原生性能

##### Ionic
更新到ionic4 版本在 ios 上基本可以达到原生性能，android 添加 crosswalk 插件体验会大幅度提升，但是会增加 APP 安装包大小

##### Native
最好

---

#### 实现原理
##### React Native
eact Native将JavaScript代码转化成Object-C或者Java语言，并且处理好了各种回调问题

##### Ionic
首先设备加载cordova应用封装器，然后cordova应用封装器加载webview，webview加载index.html文件，最后angular加载并确定默认视图，ionic渲染ionic组件作为UI

##### Native
直接调用原生插件

---

#### 优劣势
##### React Native
###### 优势
1. 基本接近原生性能
2. native 混编，可以达到系统 API 全部支持
3. 自带的调试插件，调试容易
4. 使用 jsx 语法编写 ios 和 android 两套代码，支持跨平台
5. 支持热更新
6. 可用 flexbox 布局
7. UI 效果仅次于原生

###### 缺点
1. 需要学习 jsx 语法
2. api无法满足需求时需要使用 native 混编去扩展，增加插件难度较大
3. 发展还不成熟，文档少。目前很多 ui 组件只有ios的实现，android的需要自己实现
4. UI 组件和系统 API 相对较少，使用较难

##### Ionic
###### 优势
1. 内存较高的手机可以达到原始性能
2. 编写 cordova 插件，可以达到系统 API 全部支持
3. UI 组件和系统 API 丰富，可以实现绝大多数效果并且使用容易
4. 可以浏览器调试
5. ios 和 android 基本上可以共用代码，支持跨平台
6. 支持热更新
7. 发展较成熟，已经更新到第四版本，资料较全。
9. 可用 flexbox 布局

###### 缺点
1. 需要学习 angularjs
2. api无法满足需求时需要使用 native 去扩展
3. 原生编写 cordova 插件难度大
3. app包比较大，需要加入 cordova 插件
3. 占用内存高一些(需要加载 webview 和 cordova )
4. web技术无法解决一切问题
5. 增加插件难度较大

##### native
###### 优势
1. 最好的体验以及功能实现
2. 完善成熟的开发文档

###### 缺点
1. 无法跨平台。需要具备iOS,android双平台开发技能
2. 需要通过编译后才能调试，耗时较长
3. 开发成本高，开发周期长