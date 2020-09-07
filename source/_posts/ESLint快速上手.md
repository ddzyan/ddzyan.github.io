---
title: ESLint快速上手
date: 2018-08-25 00:05:46
categories: 
- nodejs
tags:
- nodejs
- node_module
---
## 简介
可以统一配置工程代码规范标准，并且可以进行自动修复。

eslint 规则参考文档：https://eslint.org/docs/rules/

git demo地址：https://github.com/ddzyan/node-module-example/tree/master/eslint

### 安装
全局安装
```bash
//全局安装 eslint
npm install -g eslint

//初始化配置文件
eslint --init

//检测某文件
eslint yourfile.js

//自动修复某文件
eslint yourfile.js --fix
```

<!--more-->

本地安装
```bash
npm install eslint --save-dev

./node_modules/.bin/eslint --init

./node_modules/.bin/eslint yourfile.js
./node_modules/.bin/eslint yourfile.js --fix
```

参考文档：http://eslint.cn/docs/user-guide/getting-started

### 基础模板
```js
//.eslintrc.js
module.exports = {
    "extends": "airbnb-base",
    "env": {
        "node": true
    },
    "rules": {
        "linebreak-style": [0, "error", "windows"],
        "no-console": "off",
        "quotes": ["error", "single"],
        "no-plusplus": "off"
    }
};
```
### 使用
```bash
//使用前先安装必要模块，这里采用的是全局安装
npm install -g eslint-config-airbnb-base

npm install -g eslint-plugin-import

//检测
eslint index.js

//自动修复
eslint index.js --fix
```
继承于 eslint-config-airbnb 规则，规则具体内容参考 https://github.com/yuche/javascript

在继承规则后，再修改属性将覆盖继承的规则属性。

### vsCode 中使用
下载安装ESLint插件，可以在命令面板中输入命令操作：
- Create '.eslintrc.json' file：创建一个新.eslintrc.json文件。
- Fix all auto-fixable problems：将ESLint自动修复解决方案应用于所有可修复的问题。
- Disable ESLint for this Workspace：禁用此工作空间的ESLint扩展。
- Enable ESLint for this Workspace：为此工作空间启用ESLint扩展。

在项目中启动后，会实时检测代码，不需要再执行ESLint指令检测。