---
title: axios拦截器源码分析
date: 2021-01-12 02:31:28
categories:
  - nodejs
tags:
  - axios
---

## 简介

本文主要分析 axios 模块中拦截器的使用和源码分析，文章分为如下几个部分：

1. 拦截器的作用以及使用场景
2. 如何使用拦截器
3. 拦截器的源码分析

axios 源码仓库：https://github.com/axios/axios/blob/master/lib/core/Axios.js

### 拦截器的作用

axios 的拦截器可以分为请求拦截器和响应拦截器。请求拦截器可以在发出请求之前，按照拦截器添加顺序执行，再发送请求。而响应拦截器则是在收到响应之后，按照拦截器添加顺序执行后，再返回响应结果。

使用场景：

1. 数据转换
2. 添加额外的数据，例如往 header 添加信息等
3. 日志记录，输出请求响应时间和失败率等

<!--more-->

### 如何使用拦截器

下面简单代码实现请求响应时间

```js
const axios = require('axios');

const reportCgi = (res, config) => {
  const { config: conf } = res || { config };
  console.log('response', conf.url, new Date() - conf.requestTime);
};

axios.interceptors.request.use(config => {
  return { ...config, ...{ requestTime: new Date() } };
});

axios.interceptors.response.use(
  response => {
    reportCgi(response);
    return response;
  },
  error => {
    reportCgi(error.response, error.config);
    return error;
  }
);

const main = async function () {
  const res = await axios.get(
    'https://filfox.info/api/v1/message/bafy2bzacedotalbhs6nlsqdxbyu6yqajizqj3nch25m3svyx6tocd4732vzvi'
  );
  console.log(res);
};

main();
```

### 源码分析

https://github.com/axios/axios/blob/master/lib/core/Axios.js

axios 对象

```js
//...
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  // 构造中实现了2个拦截器，分别为请求拦截器和响应拦截器
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager(),
  };
}

// request 实现方式
Axios.prototype.request = function request(config) {
  //...
  var chain = [dispatchRequest, undefined];

  // 将所有请求拦截器按照添加顺序添加到chain数组之前
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  // 将所有的响应拦截器都按照添加顺序添加到chain之后
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  // 开始遍历chain执行请求，现在的数组结构类似与[request拦截器,请求,response拦截器]
  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }
};

//使用get 方式的请求复用配置文件，并且调用 Axios.prototype.request 方法
utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function (url, config) {
    return this.request(
      mergeConfig(config || {}, {
        method: method,
        url: url,
        data: (config || {}).data,
      })
    );
  };
});

// 使用 post 方式的请求复用配置文件，并且调用 Axios.prototype.request 方法
utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function (url, data, config) {
    return this.request(
      mergeConfig(config || {}, {
        method: method,
        url: url,
        data: data,
      })
    );
  };
});
```

https://github.com/axios/axios/blob/master/lib/core/InterceptorManager.js

拦截器管理对象

```js
function InterceptorManager() {
  // 内部保存一个处理数组
  this.handlers = [];
}

// 拦截器添加方法分别接收2个方法：完成状态结果处理函数和失败状态结果处理函数
InterceptorManager.prototype.use = function use(fulfilled, rejected) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected,
  });
  return this.handlers.length - 1;
};
```

简单总结：

在内部实现了一个拦截器管理器，可以通过 use 往管理器中添加处理函数（包含一个成功状态处理函数和一个失败状态处理函数）

1. axios 对象内部实例了 2 个拦截器管理器，分别为：请求拦截器和响应拦截器。

2. 在调用 request 发送请求时会创建一个数组并且添加请求处理函数
3. 首先遍历请求拦截器按照顺序将处理函数添加到数组前部，
4. 然后遍历响应拦截器按照顺序将处理函数添加到数组末尾
5. 最后遍历数组开始执行
