---
title: bignumber.js快速上手
date: 2018-10-25 01:25:45
categories: nodejs
tags:
- nodejs
- node_module
---
## 简介
用于任意精度小数和非小数运算的JavaScript库。

解决某些小数相加出现意外情况
```js
console.log(0.1+0.2); // 0.30000000000000004
```

参考地址：https://www.npmjs.com/package/bignumber.js

### 安装和加载
```bash
npm install --save bignumber.js
const BigNumber = require('bignumber.js');
```

### 使用
```js
x = new BigNumber(123.4567)
y = BigNumber('123456.7e-3')
z = new BigNumber(x)
x.isEqualTo(y) && y.isEqualTo(z) && x.isEqualTo(z) 
```
new BigNumber()参数接收Number，String或BigNumber类型，创建为十进制。可以通过如下设置其他基础进制。
```js
x = new BigNumber(123,2);
```

<!--more-->

#### 数据操作
- minus()：减
- plus()：加
- times()：乘
- dividedBy()：除
- squareRoot(): 平方根
- exponentiatedBy()：指数
- 还有其他请参考官方文档

```js
x.dividedBy(y).plus(z).times(9) // 等于((x/y)+z)*9
```

#### 判断方法
- isFinite():是否超出精度
- isNaN()：是否为NaN

#### 取值方法
- toNumber():获得 Number 类型的结果
- toString([转换的进制]):获得 String 类型的结果
- toFormat([保留的小数位数]):四舍五入保留指定位数小数，并且国际化输出带逗号// 1,234,567.83
- toFixed([保留的小数位数])：返回 string 类型

#### 配置
```js
BigNumber.config({ DECIMAL_PLACES: 10, ROUNDING_MODE: 4 })

// 创建另一个BigNumber构造函数，可以选择传入一个配置对象
BN = BigNumber.clone({ DECIMAL_PLACES: 5 })
```
- DECIMAL_PLACES:保留小数位数
- ROUNDING_MODE：舍弃模式