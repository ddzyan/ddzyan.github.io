---
title: eventLoop的事件处理机制
date: 2019-08-27 01:27:24
categories:
  - nodejs
tags:
  - nodejs
---

### 浏览器

- 宏任务：js 主线程代码，定时器，I/O 操作
- 微任务：promise ， nodejs 独有的 process.nextTask()

#### 代码

```js
while(true){
    宏任务.shift();
    微任务.(); // 执行全部
}
```

#### 文字

1. 执行一个宏任务
2. 执行全部微任务
3. 如此循环

<!--more-->

### nodejs

此说明只针对 v10 或者以上版本，v11 修改为浏览器一致(日后补充)

```shell
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

#### 代码

```js
while(true){
    loopEvent.foreach(阶段){
        阶段();
        process.nextTask();
        微任务();
    }
    loopEvent.next();
}
```

#### 文字

1. 按照事件循环顺序执行全部宏任务
2. 执行 process.nextTask();
3. 执行所有微任务

#### 事件循环 6 个阶段宏任务顺序

time ---> I/O ---> pool ---> setImmediate ---> check ---> close queue

##### pool

为防止 poll phase 耗尽 event loop，libuv 也有一个最大值（基于系统），会在超过最大值之后停止轮询更多的事件

- 获取新的 I/O ,并且执行 I/O 回调，事件循环将在这里堵塞
- 当设定的 immediate 或者 timers 超过上限，则执行下一个阶段

到了 pool 阶段将执行所有 I/O queue，当任务全部情况或者堵塞时间超过了 timers ，则结束 pool 阶段，出现如下情况

1. setImmediate 不为空，则执行 setImmediate queue，在执行 close queue
2. setImmediate 为空，time 不为空，则重新开始事件循环
3. setImmediate 为空，time 也为空，则不再继续

### 测试用例

#### 第一

```js
setTimeout(() => console.log('setTimeout1'), 0);
setTimeout(() => {
  console.log('setTimeout2');
  Promise.resolve().then(() => {
    console.log('promise2');
    Promise.resolve().then(() => {
      console.log('promise3');
    });
    console.log(5);
  });
  setTimeout(() => console.log('setTimeout4'), 0);
}, 0);
setTimeout(() => console.log('setTimeout3'), 0);
Promise.resolve().then(() => {
  console.log('promise1');
});
```

##### nodejs 结果

1. promise1
2. setTimeout1
3. setTimeout2
4. setTimeout3
5. promise2
6. 5
7. promise3
8. setTimeout4

##### 浏览器结果

1. promise1
2. setTimeout1
3. setTimeout2
4. promise2
5. 5
6. promise3
7. setTimeout3
8. setTimeout4
