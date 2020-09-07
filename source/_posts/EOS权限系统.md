---
title: EOS权限系统
date: 2018-08-09 05:56:53
categories: 
- eos
tags:
- 区块链
- eos
---
## 概念
参考资料：https://hiblock.net/topics/222

### 权限作用
1. 向EOS区块链发起一些事务，比如说转账，需要得到 **账户** 的授权。
2. 账号交易可以通过权限转移做到。
3. 新权限和action的绑定关系，可增加了eos网络权限的灵活性，通过单个权限的绑定，可将一个账户的权限分层管理，甚至一个公司的所有人都可以使用一个EOS账户来进行权限分离。

**账户** 的授权是如何授权的？账户的权限里 有一个阈值属性，当你的解锁状态的钱包中的有一把私钥能对应到那个权限所绑定的公钥上，而且权重刚好大于等于阈值时，那么就能成功签名，向区块链发送事务。

<!--more-->

### 权限配置
首先当一个账户创建的时候，具备了两种基本权限。每个权限绑定到一个公钥上（单签名账户）或多个公钥上（多签名账户），除了绑定公钥也可以绑定到另一个有效的账户上：

- **owner** ： 账号主权限, 声明的这个账号的归属。只有极少数事务需要使用到 owner权限。 建议把拥有这个权限的私钥进行**冷存储**。不要分享给任何人。 owner 可以用来恢复其他权限。
- **active**： 活动权限，顾名思议，这个权限 ，一般用来做一些转账、投票、发起事务等常规操作。

除了以上两种默认权限，还可以对账户自定义新的权限（计划在未来的账户管理软件中加入支持）。 这种权限的可扩展性，非常灵活，给软件开发者提供了很多可能的使用场景。

### 权限的权重（weight）与阈值（threshold）
#### 单签名账户 （默认权限配置的账户）

owner 和 active 权限分别有一个值为1的阈值。

owner 和 active 所绑定的公钥,则分别有一个值为1的权重。

#### 怎么理解阈值和权重
举个例子来讲吧，把 **owner** 这个权限比作一扇门，打开这扇门需要一把正确的钥匙。 而 **owner** 所绑定的那个公钥 对应的那把 **私钥** 就是正确的钥匙。

因此 **单签名账户** 就是 **权限的阈值** 和 **钥匙的权重**  都为1的一种账户类型。使用某个权限，只需要一把对应的私钥就行了

#### 多重签名账户

顾名思义，就是一个权限绑定了多个账户或公钥，权重之和大于等于owner权限的阈值，才能使用这个权限。

使用一个权限，可能需要不只一个公钥的签名了，也可能是两个、三个、五个。


## 命令操作
EOS版本号：v1.0.6

环境：私有链

操作系统：centos 7 

参考资料：https://developers.eos.io/eosio-cleos/reference#cleos-set-account


#### 查看账号权限
```
cleos get account eosioddztest
permissions: 
     owner     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
        active     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
memory: 
     quota:       unlimited  used:      2.66 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```

#### 设置权限
```
//修改权限
cleos set account permission $ACCOUNT_NAME $PERMISSION_NAME $PUBKEY_OR_ACCOUNT_NAME active -p sandwichfarm@active

```

##### 参数
- account TEXT - 为其设置/删除权限授权的帐户
- permission TEXT - 设置/删除权限的权限名称
- authority TEXT - [delete] NULL，[create / update]公钥，JSON字符串或定义权限的文件名
- parent TEXT - [创建]此父母权限的权限名称（默认为："active"）

##### 选项
- -h,--help 打印此帮助信息并退出
- -x,--expiration TEXT - 设置交易到期之前的秒数，默认为30秒
- -s,--skip-sign 指定是否应使用解锁的钱包密钥来签署交易
- -d,--dont-broadcast - 不要向网络广播交易（只需打印到标准输出）
- -p,--permission TEXT - 要授权的帐户和权限级别，如'account @ permission'（默认为'account @ active'）
- --max-cpu-usage-ms UINT - 设置CPU使用预算的毫秒数上限，用于执行交易（默认为0，表示无限制）
- --max-net-usage UINT - 为交易设置净使用预算的上限（以字节为单位）（默认为0，表示无限制）

##### 用法
要修改帐户的权限，您必须拥有该帐户的权限和您正在修改的权限。


```
//example 
//修改权限
cleos set account permission eosioddztest active EOS69bTRDpiVr9wymPY3uKK9K28N1kh7kxR1wn6ynhbpWAasJKW1K owner -p eosioddztest@active

//再次查看eosioddztest权限，发现active已经改变
cleos get account eosioddztest
permissions: 
     owner     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
        active     1:    1 EOS69bTRDpiVr9wymPY3uKK9K28N1kh7kxR1wn6ynhbpWAasJKW1K
memory: 
     quota:       unlimited  used:      2.66 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited

```

##### 权重和阈值模式
```
//修改权限 
cleos set account permission eosioddztest active '{"threshold" : 1, "keys" : [], "accounts" : [{"permission":{"actor":"trustwallets","permission":"active"},"weight":1}]}' owner -p eosioddztest@active
```

```
查看权限，发现action已经改变为权重模式，如看详情可以-j，使用json方式
cleos get account eosioddztest
permissions: 
     owner     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
        active     1:    1 trustwallets@active, 
memory: 
     quota:       unlimited  used:     2.643 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited

```

###### 参数
accounts

```
{
  "threshold"       : 100,    /*签名需要权重值达到多少的一个整数*/
  "keys"            : [],     /*包含eos风格的公约和权重*/
  "accounts"        : []      /*包含eos账号和权限，权重*/
}
```

keys

```
{
  "permission" : {
    "actor"       : "sandwich",
    "permission"  : "active"
  },
  "weight"      : 75
}
```

##### 添加权限

```
cleos set account permission eosioddztest test '{"threshold":1,"keys":[{"key":"EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9","weight":1}],"accounts":[]}' active
```

```
查看权限多了一个test
cleos get account eosioddztest
permissions: 
     owner     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
        active     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
           test     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
memory: 
     quota:       unlimited  used:      2.99 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited

```

##### 删除权限

```
//删除权限
cleos set account permission eosioddztest test 'NULL' active
```

```
cleos get account eosioddztest
permissions: 
     owner     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
        active     1:    1 EOS8Lbyk31K78BaMGuP8WHiC1tL8tphrQjRir8pGccxxbg9zVA4f9
memory: 
     quota:       unlimited  used:      2.99 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```

##### 权限绑定action

```
//将之前创建的test权限绑定交易事件
cleos set action permission eosioddztest eosio.token transfer test

//使用test权限进行交易
cleos push action eosio.token transfer '["eosioddztest","trustwallets","100.0000 EOS","transfer"]' -p eosioddztest@hello
```

