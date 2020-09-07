---
title: EOS智能合约介绍
date: 2018-08-09 05:54:06
categories: 
- eos
tags:
- 区块链
- eos
---

- 文章参考：https://developers.eos.io/eosio-cpp/docs/introduction
- eosiocpp文件地址：/eos/build/tools/eosiocpp

# 介绍
EOSIO智能合约是在区块链上注册并在EOSIO节点上执行的软件，它实现了“合同”的语义，其合约行动请求的分类账存储在区块链中。智能合约定义了接口（操作，参数，数据结构）和实现接口的代码。代码被编译成规范的字节码格式，节点可以检索和执行。区块链存储合同的交易。每份智能合约都必须附有一份Ricardian Contract，该合同定义了合同中具有法律约束力的条款和条件。

<!--more-->

# 通信模式
EOSIO智能合约由一组action和type定义组成。action定义指定并实现合同的行为,type定义指定所需的内容和结构。EOSIO操作主要在基于消息的通信体系结构中运行。客户端通过发送（推送）消息来调用操作nodeos。这可以使用cleos命令完成。它也可以使用EOSIO send方法之一（例如eosio::action::send）完成。nodeos将操作请求分派给实现合同的WASM代码。该代码完整运行，然后继续处理下一个操作。

EOSIO智能合约可以彼此通信，例如让另一个合同执行一些与当前事务完成相关的操作，或者在当前事务范围之外触发未来的事务。彼此之间以Action和共享内存数据库访问的形式进行通信；

EOSIO支持两种基本的通信模型，内联和延迟。

## Inline（内联）
- 内联通信模型立即执行当前的事务；
- 内联通信模型无论执行成功或失败都不会收到通知；
- 内联通信模型只会在原始事务的权限和操作范围内进行操作。

## Deferred（延期）
- 延期通信模型，根据生产者的决定，将事务延期执行；
- 延期通信模型可以传达通信结果，允许简单的超时；
- 延期通信模型可以延伸到不同的范围。

## Transactions和action
- action表示单个操作，而transactions则由一个或者多个action组成。
- 合约和帐号以actions的方式进行通信。
- action可以单独发送，如果打算把多个action作为一个整体执行，则可以以transactions形式发送。

### Action 名称限制
- action类型实际上是base32编码的64位整数。 表示仅限于字符a-z，1-5和'.' 对于前12个字符。
- 如果有第13个字符，则它被限制为前16个字符（'.'和a-p）。

### Transaction 确认
transaction完成后将返回transactionHash，如果在区块中找到包含此transactionHash的区块，则表示transaction已经被确认。


# 文件结构

后缀 | 用途
---|---
hpp | 类及其成员声明
cpp | 业务逻辑
abi|定义接口数据结构
wast|部署智能合约的指定文件


## hpp
$ {contract} .hpp是头文件，包含.cpp文件引用的变量，常量和函数。

## cpp
${contract}.cpp 实现合约功能的逻辑都写在这里。


```
//创建合约基本架构，里面包含cpp和hpp两个文件
eosiocpp -n ${contract}
```

## abi
- 应用程序二进制接口（ABI）是一个基于JSON的描述，介绍如何在JSON和二进制表示之间转换用户操作。 ABI还描述了如何将数据库状态转换为JSON或从JSON转换。
- 文件格式类似JSON，用来定义智能合约与EOS系统外部交互的数据接口。

```
//eosiocpp工具将cpp编译为abi
eosiocpp -g ${contract}.abi ${contract}.hpp
```

## wast
- 任何要部署到EOSIO区块链的程序都必须编译成WASM格式。这是区块链接受的唯一格式。
- .wast是.wasm的文本格式；

```
//eosiocpp工具将cpp编译为WASM(.wast)
eosiocpp -o ${contract}.wast ${contract}.cpp
```

## CMakeLists.txt
包含了项目所需要的类库的位置，使用的EOS默认安装的位置。
