---
title: nodejs之buffer与stream
date: 2020-03-19 15:10:35
categories: 
- nodejs
tag: 
- nodejs
---
### buffer
buffer 使用从磁盘读取保存的临时空间，用于文件操作，二进制数据处理。

buffer 一个堆外内存，不通过v8引擎进行分配，而是由node的c++层面提供，node层面进行操作。内存申请操作如下：
1. 当申请的内存小于8kb的时候，会与现实申请一个固定存储(8kb)区域，用于保存数据。
2. 当下一个申请的内存也小于8kb的时候，会进行判断上一个申请的固定存储区域剩余空间是否足够，不够则会申请一个新的固定存储区域，用于存储，上一个空间的剩余部分则不会被使用，直到存储的数据被释放，固定存储区域则会被释放（小数据内存采用预先申请，事后分配，减少了node于系统之间的申请和调用操作）
3. 当申请的内存大于8kb的时候，会直接申请一个指定大小的存储区域，用于保存数据（大数据由c++层面提供，不需要事后分配）

使用场景：
1. 网络数据传输的时候，采用buffer会比使用string更有效率
2. 文件操作，例如下面要讲到的stream

<!--more-->

### stream
通过以上内容了解到，buffer只能在定义的时候申请到指定大小的存储区域，如果保存的数据太大，则会造成内存溢出。

stream 是i/o，网络操作进行传输的数据。

```js
const http = require('http');
const fs = require('fs');

http.createServer((req,res)=>{
    const readStream = fs.createReadStream('./data.txt',{
        highWaterMark:60*1024
    });
    readStream.pipe(res);
}).lissten(3000);
```
fs.createReadStream的工作方式是在内存中预先创建一个buffer对象(大小由highWaterMark控制)，然后通过fs.read()读取时逐步将磁盘中的字节复制到buffer中。完成一次读取时候，会从这个buffer中通过slice()方法取出部分数据作为一个小buffer对象，再通过data事件传递给调用方。如果buffer用完，则重新申请；如果还有剩余，则继续使用。

每次读取的指定长度由 highWaterMark 设定，所以 highWaterMark 大小会影响触发 data 事件的次数。值越大，读取速度越快。


