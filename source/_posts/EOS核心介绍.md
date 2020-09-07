---
title: EOS核心介绍
date: 2018-08-04 02:03:20
categories: 
- eos
tag: 
- 区块链
- eos

---
## account
### 账号规则
[账号命名规则](https://developers.eos.io/eosio-nodeos/docs/learn-about-wallets-keys-and-accounts-with-cleos)
1. Must be less than 13 characters
2. Can only contain the following symbols: .12345abcdefghijklmnopqrstuvwxyz

<!--more-->

## 可执行文件
### 路径
eos/build/programs

### nodeos
- nodeos是EOS的核心守护进程，可以通过它运行一个节点；
- nodeos常见用途是生产区块、作为API端点、本地开发等；
- nodeos可以理解为EOS区块链的服务端，可以通过添加插件（plugin）的方式为客户端提供API。

### cleos
- cleos是“client eos”的缩写，可以理解为访问EOS区块链的客户端；
- cleos访问nodeos暴露的API，需要nodeos的IP地址和端口号；可以通过修改config.ini文件中http-server-address属性修改默认连接服务地址。

### keosd
- keosd用于存储交易签名的私钥;
- keosd在本地节点上运行，并将私钥保存在本地节点上，是一个eos钱包守护进程；
- keosd位于 eos/build/programs/keosd 路径下;
- 与keosd交互使用的工具是cleos。

#### 注意
除了使用keosd来管理你的钱包外，还能使用nodeos来管理钱包。不建议同时使用keosd和nodeos来管理钱包，虽然不会出现什么问题，但很容易引起混淆。

使用方法，在config.ini文件中添加插件，重启nodeos。
```
plugin = eosio::wallet_api_plugin

```
注意：当使用nodeos来管理钱包时，如果nodeos关闭，钱包将会被加锁。重新启动nodeos后，需要使用unlock命令解锁钱包。
如果同时运行nodeos和keosd，cleos会优先访问nodeos。

## 基础智能合约
### eosio.bios
- 检测、设置指定账户的权限；
- 限制指定账户或全局的资源使用；
- 设置区块生产者。

#### 部署方法
```
./cleos.sh set contract eosio  ../eos/build/contracts/eosio.bios/ -p eosio
```

### eosio.token
- 创建代币
- 发布代币
- 转账

#### 部署
```
./cleos.sh set contract eosio.token  ../eos/build/contracts/eosio.token/ -p eosio.token 
```

### eosio.system
创建账号，更新权限，删除权限，关联权限，取消关联权限，canceldelay，onerror
设置内存大小，setparams，setpriv，rmvproducer，bidname
购买指定字节内存，用EOS购买内存，销售内存，抵押，取消抵押，refund
注册生产者，取消注册生产者，投票，设置成为投票代理
更新指定生产者的区块信息，（生产者）获取回报

#### 部署
请参考eosio.system部署

### eosio.msig
msig”是“multiple signature”（多重签名）的简写，顾名思义，就是让多个账户对一起事务进行签名。可以异步提出、批准、发布经过多方同意的事务。
#### 部署
```
cleos set contract eosio.msig eosio.msig -p eosio.msig

```