---
title: nodejs性能调优--解决CPU占用过高问题
date: 2018-12-10 02:12:29
categories: 
- nodejs
tag: 
- nodejs
---
#### 简介
Nodejs(服务端)的角度来看，JS本身的执行时间对服务的影响至关重要，如果执行时间从30ms降低到3ms，理论上QPS就能提升10倍，换句话说，以前要10台服务器才能扛住的流量现在1台服务器就能扛住，而且响应时间更短.

- QPS：每秒查询率

#### 方法1：Node 自带 profile
##### 以--prof参数启动Node应用
```bash
node --prof [启动脚本]
```
##### 通过压测工具loadtest向服务施压
```bash
loadtest  [http://127.0.0.1:6001] --rps 10
```
##### 处理生成的log文件
```bash
node --prof-process [isolate-0XXXXXXXXXXX-v8-XXXX.log] > profile.txt
```

<!--more-->

##### 分析profile.txt文件
##### 查看样本中运行的代码比例
 [Summary]:
```
[Summary]:
   ticks  total  nonlib   name
     79    0.2%    0.2%  JavaScript
  36703   97.2%   99.2%  C++
      7    0.0%    0.0%  GC
    767    2.0%          Shared libraries
    215    0.6%          Unaccounted
```
这告诉我们收集的所有样本中有97％发生在C ++代码中，当查看处理输出的其他部分时，我们应该最关注用C ++完成的工作（而不是JavaScript）

##### 分析语言中的哪些函数占用最多CPU信息
 [C++]
```
 [C++]:
   ticks  total  nonlib   name
  19557   51.8%   52.9%  node::crypto::PBKDF2(v8::FunctionCallbackInfo<v8::Value> const&)
   4510   11.9%   12.2%  _sha1_block_data_order
   3165    8.4%    8.6%  _malloc_zone_malloc
```
我们看到前三个条目占该程序占用CPU时间的72.1％。从这个输出中，我们立即看到至少51.8％的CPU时间被一个名为PBKDF2的函数占用

##### 分析堆栈，查看主要调用者信息
[Bottom up（heavy）profile]
```bash
ticks parent  name
  19557   51.8%  node::crypto::PBKDF2(v8::FunctionCallbackInfo<v8::Value> const&)
  19557  100.0%    v8::internal::Builtins::~Builtins()
  19557  100.0%      LazyCompile: ~pbkdf2 crypto.js:557:16

   4510   11.9%  _sha1_block_data_order
   4510  100.0%    LazyCompile: *pbkdf2 crypto.js:557:16
   4510  100.0%      LazyCompile: *exports.pbkdf2Sync crypto.js:552:30

   3165    8.4%  _malloc_zone_malloc
   3161   99.9%    LazyCompile: *pbkdf2 crypto.js:557:16
   3161  100.0%      LazyCompile: *exports.pbkdf2Sync crypto.js:552:30
```