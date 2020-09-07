---
title: eosio.bios合约分析(一)
date: 2018-08-09 05:51:23
categories: 
- eos
tags:
- 区块链
- eos
---
# 前提
## 目标
从基础的智能合约中分析代码结构，了解调用方式和合约的作用。
## 软件版本
- eos版本：v1.1.1
- 操作系统：centos 7

建议阅读前先了解：https://blockflow.net/t/topic/663

<!--more-->

# 参考文档：
- [C/C++ API参考](https://developers.eos.io/eosio-cpp/reference)
- [智能合约介绍](https://developers.eos.io/eosio-cpp/docs)

EOSIO提供了一组服务和接口，使合同开发人员能够
跨action保持状态。其中包括：
1. 提供在数据库中保持状态的服务
1. 增强查询功能以查找和检索数据库内容 
1. 针对以上功能提供C ++ API，供合同开发人员使用 /eos/contracts/eosiolib/*.hpp,*.cpp 文件
1. 访问核心服务的C API，对库和系统开发人员有用 /eos/contracts/eosiolib/*.h 文件

## 简介
bios全称是：Basic Input/Output System（基本输入/输出系统）。

### 部署方式
```
cloes set contract eosio ./contracts/eosio.bios/ -p eosio
```

### 源码分析
#### 项目结构
``` 
├── CMakeLists.txt 产生Makefile文件，通过make命令编译和链接所需要程序。
├── eosio.bios.abi 数据结构和接口
├── eosio.bios.cpp 核心逻辑代码
└── eosio.bios.hpp 头文件类及其成员声明

```
#### 核心eosio.bios.hpp
```
#pragma once
#include <eosiolib/eosio.hpp>
#include <eosiolib/privileged.hpp>

namespace eosio {

   class bios : public contract {
      public:
         bios( action_name self ):contract(self){}
        
         //设置账号权限
         void setpriv( account_name account, uint8_t ispriv ) {
            require_auth( _self );
            set_privileged( account, ispriv );
         }
         //限制账号资源
         void setalimits( account_name account, int64_t ram_bytes, int64_t net_weight, int64_t cpu_weight ) {
            require_auth( _self );
            set_resource_limits( account, ram_bytes, net_weight, cpu_weight );
         }
         //设置区块链资源,无任何操作，暂时无效
         void setglimits( uint64_t ram, uint64_t net, uint64_t cpu ) {
            (void)ram; (void)net; (void)cpu;
            require_auth( _self );
         }
         //设置生产节点
         void setprods( std::vector<eosio::producer_key> schedule ) {
            (void)schedule; // schedule argument just forces the deserialization of the action data into vector<producer_key> (necessary check)
            require_auth( _self );

            constexpr size_t max_stack_buffer_size = 512;
            size_t size = action_data_size();
            char* buffer = (char*)( max_stack_buffer_size < size ? malloc(size) : alloca(size) );
            read_action_data( buffer, size );
            set_proposed_producers(buffer, size);
         }
         //检查账户权限
         void reqauth( action_name from ) {
            require_auth( from );
         }

      private:
   };

} /// namespace eosio

```
所有方法通过eosio.bios.cpp中的EOSIO_ABI宏进行封装，映射给开发者使用。
require_auth(),通过C/C++ API参考查询得知，此方法定义在action.h中，作用为验证提供的身份验证集中是否存在指定的帐户。

eosio.bios中所有具体的操作，都定义在privileged.h中，在此都为调用。

privileged.h包含的具体方法的作用和参数定义可以结合 https://developers.eos.io/eosio-cpp/reference#privileged 和源码一起看。 
