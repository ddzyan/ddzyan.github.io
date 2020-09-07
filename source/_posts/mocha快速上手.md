---
title: mocha快速上手
date: 2018-10-21 01:24:11
categories: nodejs
tags:
- nodejs
- 测试
---
![image](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120301.png)

### 参考资料
- [mocha入门1](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
- [mocha入门2](https://www.jianshu.com/p/9c78548caffa)
- [Chai.js断言库API中文文档](https://www.jianshu.com/p/f200a75a15d2)

### 简介
Mocha（发音"摩卡"）诞生于2011年，是现在最流行的JavaScript测试框架之一，在浏览器和Node环境都可以使用。
- 验证代码的正确性
- 避免修改代码时出错
- 避免其他团队成员修改代码时出错
- 便于自动化测试与部署

#### 插件
##### supertest
支持调试本地 api 接口

### 模块安装
```
//全局安装
npm i -g mocha

//项目内安装
npm i mocha --save-dev
```

<!--more-->

### 测试脚本编写
首先写一个需要测试的脚本
```js
// add.js
function add(x, y) {
  return x + y;
}

module.exports = add;
```

测试代码后缀名为.test.js（表示测试）或者.spec.js（表示规格）
```js
// add.test.js
var add = require('./add.js');
var expect = require('chai').expect;

describe('加法函数的测试', function() {
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
});
```
#### 参数
- describe块称为"测试套件"（test suite），表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称（"加法函数的测试"），第二个参数是一个实际执行的函数。
- it块称为"测试用例"（test case），表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称（"1 加 1 应该等于 2"），第二个参数是一个实际执行的函数。

### 执行测试
```bash
mocha add.test.js

  加法函数的测试
    ✓ 1 加 1 应该等于 2

  1 passing (8ms)
```
上面的运行结果表示，测试脚本通过了测试，一共只有1个测试用例，耗时是8毫秒。

### 测试用例的钩子
Mocha在describe块之中，提供测试用例的四个钩子：before()、after()、beforeEach()和afterEach()。它们会在指定时间执行。

```js
describe('hooks', function() {

  before(function() {
    // 在本区块的所有测试用例之前执行
  });

  after(function() {
    // 在本区块的所有测试用例之后执行
  });

  beforeEach(function() {
    // 在本区块的每个测试用例之前执行
  });

  afterEach(function() {
    // 在本区块的每个测试用例之后执行
  });

  // test cases
});
```

### 命令行参数
#### --reporter, -R
使用mochawesome模块，可以生成漂亮的HTML格式的报告
![image](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120303.png)
```bash
$ npm install --save-dev mochawesome
$ ../node_modules/.bin/mocha --reporter mochawesome
```
测试结果报告就在mochaawesome-reports子目录生成

```
//mocha.opts

--timeout 30000
--slow 1000
--bail
--recursive
--reporter mochawesome
--reporter-options overwrite=true,reportDir=report,inline=true,cdn=true,json=false,reportTitle=x-blockchain
```
##### 测试报告工具详细设置
- --reporter：指定测试报告工具
- --reporter-options(-O)：测试报告工具详细设置

###### mochawesome参数配置
参考地址：https://github.com/adamgruber/mochawesome-report-generator#options

- html：测试输出HTML
- json：测试输出JSON
- reportDir：输出的文件地址
- reportFilename：测试输出的文件名称

---

#### --bail, -b
--bail参数指定只要有一个测试用例没有通过，就停止执行后面的测试用例。这对持续集成很有用。
```basg
mocha --bail
```
####  --grep, -g
--grep参数用于搜索测试用例的名称（即it块的第一个参数），然后只执行匹配的测试用例。
```bash
mocha --grep "111"
```
#### --invert, -i
--invert参数表示只运行不符合条件的测试脚本，必须与--grep参数配合使用。
```bash
mocha --grep "1 加 1" --invert
```
#### --timeout
设置测试超时时间

#### --slow
指定“慢”测试阈值，默认为75毫秒。Mocha用这个去高亮那些耗时过长的测试。

#### --recursive 
包含子目录

---

#### 配置文件mocha.opts
Mocha允许在test目录下面，放置配置文件mocha.opts，把命令行参数写在里面。
```opts
//mocha.opts
--reporter tap
--recursive
--growl
```
然后，执行mocha就能取得与第一行命令一样的效果。