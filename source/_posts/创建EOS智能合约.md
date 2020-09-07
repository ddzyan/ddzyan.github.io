---
title: 创建EOS智能合约
date: 2018-08-09 05:55:52
categories: 
- eos
tags:
- 区块链
- eos
---
## 前提
首先请先了解智能合约相关概念，智能合约介绍。

在开始之前以下信息。
1. eos-version:v.1.5
1. os:centos 7
2. 必须在build文件夹中执行sudo make isntall，否在在执行编译wast文件时，将找不到相关模块。
3. 参考文档：https://developers.eos.io/eosio-cpp/docs/hello-world

## 创建智能合约
### 初始化合约结构
这里采用最简单的方式，用eosiocpp命令创建合约基础架构。

```
eosiocpp -n hello
```

此时在eos/build/tools/文件夹下将产生一个hello文件夹，文件夹内包含hello.cpp  hello.hpp两个最基础的智能合约文件。

<!--more-->

#### 查看hello.cpp

```
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>
using namespace eosio;

class hello : public eosio::contract {
  public:
      using contract::contract;

      /// @abi action 
      void hi( account_name user ) {
         print( "Hello, ", name{user} );
      }
};

EOSIO_ABI( hello, (hi) )
```

可以看出合约中定义了一个hi的方法，并且包含了一个user参数。方法在执行之后将输出Hello ${user}。

### 编译wast文件

```
eosiocpp -o hello.wast hello.cpp
```
任何要部署到EOSIO区块链的程序都必须编译成wast格式。这是区块链接受的唯一格式。

### 编译abi文件

```
eosiocpp -o hello.abi hello.hpp
```

abi文件定义接口数据结构，使开发人员和用户将能够通过JSON无缝地与智能合约进行交互。

### 部署合约
#### 创建部署的合约账户

```
cleos create account eosio hello.code EOS69bTRDpiVr9wymPY3uKK9K28N1kh7kxR1wn6ynhbpWAasJKW1K EOS69bTRDpiVr9wymPY3uKK9K28N1kh7kxR1wn6ynhbpWAasJKW1K
```
#### 部署智能合约

```
cleos set contract hello.code ../hello -p trustwallets
```
### 调试合约

```
cleos push action hello.code hi '["user"]' -p  trustwallets
```

将在nodeos中输出Hello user。
由于合约中并未添加身份验证，此时合同允许任何人授权。

```
cleos push action hello.code hi '["user"]' -p  eosio
```
可在hi函数中添加验证require_auth()方法。

```
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>
using namespace eosio;

class hello : public eosio::contract {
  public:
      using contract::contract;

      /// @abi action 
      void hi( account_name user ) {
         require_auth(user)
         print( "Hello, ", name{user} );
      }
};

EOSIO_ABI( hello, (hi) )
```
重复新编译wast文件并生成abi，然后再次部署更新合同。此时只有trustwallets授权才能进行访问此方法。

## 备注
EOS 合约是由一个EOS账户来部署合约，之后该EOS账户名就是合约的标识，并且EOS账户名和合约是一一对应的。如果EOS账户重新部署了一个新的合约，则会导致旧合约失效，并且从主网消失。
此方法可以用来删除旧智能合约，部署新合约。