---
title: nodejs错误处理最佳实现
date: 2018-11-16 00:42:48
categories: 
- nodejs
tag: 
- nodejs
---
- 参考资料：https://segmentfault.com/a/1190000002741935#articleHeader3

### 四种传递错误的方式
1. 作为异常抛出
2. 把错误传给一个callback
3. 在EventEmitter上触发一个Error事件
4. 程序运行过程中抛出的异常，或由底层库抛出，或是运行中发生的一些 SyntaxError 之类

### 错误捕获的几种方式
![image](/images/errorInfo.png)
#### try .. catch
常用的一种捕获错误方式，浏览器 || node 环境均适用

缺点：只针对同步异常有效

#### callback
异步执行的函数通过回调函数，将错误返回给同步函数。

#### EventEmitter
由 Events 模块提供的 EventEmitter 类，基于 Observer 模式做的 publish/subscribe，通过 .on('error', ...) || .addEventlistener('error', ...) 注册 subscriber，.emit() 发布事件，但会有最大的 maxListener 的限制，可更改。

<!--more-->

#### Process
Process 进程对象也是 EventEmitter 的实例，可通过如下两事件监听 error
##### uncaughtException  
监听在程序中未被捕获的异常。
##### unhandledRejection 
监听在程序中未被捕获的promise异常

#### 标准error类的信息
- ==name==：用于在程序里区分众多的错误类型（例如参数非法和连接失败）
- ==message==：一个供人类阅读的错误消息。对可能读到这条消息的人来说这应该已经足够完整。如果你从更底层的地方传递了一个错误，你应该加上一些信息来说明你在做什么。怎么包装异常请往下看。
- ==stack==：一般来讲不要随意扰乱堆栈信息。甚至不要增强它。V8引擎只有在这个属性被读取的时候才会真的去运算，以此大幅提高处理异常时候的性能。如果你读完再去增强它，结果就会多付出代价，哪怕调用者并不需要堆栈信息。

用详细的属性来增强 Error 对象。
举个例子，如果遇到无效参数，把 propertyName 设成参数的名字，把 propertyValue 设成传进来的值。如果无法连到服务器，用 remoteIp 属性指明尝试连接到的 IP。如果发生一个系统错误，在syscal 属性里设置是哪个系统调用，并把错误代码放到errno属性里。具体你可以查看附录，看有哪些样例属性可以用。

#### 总结
- 学习了怎么区分操作失败，即那些可以被预测的哪怕在正确的程序里也无法避免的错误（例如，无法连接到服务器）；而程序的Bug则是程序员失误。
- 操作失败可以被处理,也应当被处理。程序员的失误无法被处理或可靠地恢复（本不应该这么做），尝试这么做只会让问题更难调试。
- 学习了怎么区分操作失败，即那些可以被预测的哪怕在正确的程序里也无法避免的错误（例如，无法连接到服务器）；而程序的Bug则是程序员失误。
- 在写新函数的时候，用文档清楚地记录函数预期的参数，包括它们的类型、是否有其它约束（例如必须是有效的IP地址），可能会发生的合理的操作失败（例如无法解析主机名，连接服务器失败，所有的服务器端错误），错误是怎么传递给调用者的（同步，用throw，还是异步，用 callback 和 EventEmitter）。
- 缺少参数或者参数无效是程序员的失误，一旦发生总是应该抛出异常。函数的作者认为的可接受的参数可能会有一个灰色地带，但是如果传递的是一个文档里写明接收的参数以外的东西，那就是一个程序员失误。
- **传递错误的时候用标准的 Error 类和它标准的属性**。尽可能把额外的有用信息放在对应的属性里。如果有可能，用约定的属性名（如下）。

#### 脚注
1. 异步函数无法使用throw一个异常，被外面的catch代码块捕获。因为异步函数在运行的时候，同步函数try代码块已经退出了。
2. 在JavaScript里，抛出一个不属于Error的参数从技术上是可行的，但是应该被避免。这样的结果使获得调用堆栈没有可能，代码也无法检查name属性，或者其它任何能够说明哪里有问题的属性。

#### 例子
```js
/*
* Make a TCP connection to the given IPv4 address.  Arguments:
*
*    ip4addr        a string representing a valid IPv4 address
*
*    tcpPort        a positive integer representing a valid TCP port
*
*    timeout        a positive integer denoting the number of milliseconds
*                   to wait for a response from the remote server before
*                   considering the connection to have failed.
*
*    callback       invoked when the connection succeeds or fails.  Upon
*                   success, callback is invoked as callback(null, socket),
*                   where `socket` is a Node net.Socket object.  Upon failure,
*                   callback is invoked as callback(err) instead.
*
* This function may fail for several reasons:
*
*    SystemError    For "connection refused" and "host unreachable" and other
*                   errors returned by the connect(2) system call.  For these
*                   errors, err.errno will be set to the actual errno symbolic
*                   name.
*
*    TimeoutError   Emitted if "timeout" milliseconds elapse without
*                   successfully completing the connection.
*
* All errors will have the conventional "remoteIp" and "remotePort" properties.
* After any error, any socket that was created will be closed.
*/
function connect(ip4addr, tcpPort, timeout, callback)
{
assert.equal(typeof (ip4addr), 'string',
    "argument 'ip4addr' must be a string");
assert.ok(net.isIPv4(ip4addr),
    "argument 'ip4addr' must be a valid IPv4 address");
assert.equal(typeof (tcpPort), 'number',
    "argument 'tcpPort' must be a number");
assert.ok(!isNaN(tcpPort) && tcpPort > 0 && tcpPort < 65536,
    "argument 'tcpPort' must be a positive integer between 1 and 65535");
assert.equal(typeof (timeout), 'number',
    "argument 'timeout' must be a number");
assert.ok(!isNaN(timeout) && timeout > 0,
    "argument 'timeout' must be a positive integer");
assert.equal(typeof (callback), 'function');

/* do work */
}
```
- 参数，类型以及其它一些约束被清晰的文档化
- 这个函数对于接受的参数是非常严格的，并且会在得到错误参数的时候抛出异常（程序员的失误）
- 可能出现的操作失败集合被记录了。通过不同的”name“值可以区分不同的异常，而”errno“被用来获得系统错误的详细信息
- 异常被传递的方式也被记录了（通过失败时调用回调函数）
- 返回的错误有”remoteIp“和”remotePort“字段，这样用户就可以定义自己的错误了（比如，一个HTTP客户端的端口号是隐含的）
- 虽然很明显，但是连接失败后的状态也被清晰的记录了：所有被打开的套接字此时已经被关闭