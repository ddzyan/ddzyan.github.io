---
title: EOS常见问题
date: 2018-08-09 05:47:23
categories: 
- eos
tag: 
- 区块链
- eos
---
##### 运行nodeos时候报错
Q：下边代码是用于异常停止服务的恢复

A：

```
nodeos --data-dir /opt/eosio/bin/data-dir --hard-replay-blockchain --truncate-at-block [有问题的区块编号] -e
```

---

<!--more-->

Q：

```
：std::exception::what: unrecognised option '--resync-blockchain'
```

A：

```
 nodeos --replay-blockchain --hard-replay-blockchain --delete-all-blocks
```

---

Q：

```
std::exception::what: unable to find plugin: eosio::account_history_api_plugin
```
A:

```
plugin = eosio::account_history_api_plugin
替换为
plugin = history_api_plugin
```

---

Q：get_actions和get_transaction接口使用后没有返回值？

A：启动的时候添加--filter-on "*"

---

Q:
```
Error 3090003: provided keys, permissions, and delays do not satisfy declared authorizations
```

A：将私钥导入钱包中或者私钥错误

---

Q：公网同步区块失败，报错如下图

A：修改genesis.json,将这个修改为"initial_key": "EOS7EarnUhcyYqmdnPon8rm7mBCTnBoot6o7fE2WzjvEX2TdggbL3",保证genesis.json文件中的公钥和主网一致。

---

Q：
```
Error 3040005: Expired Transaction
Please increase the expiration time of your transaction!
```
A：注意服务器系统时间是否准确，并且可以修改config.ini中max-transaction-time = 30000 

---
Q：

```
Beginning build version: 1.2
Thu Jul 12 06:33:42 UTC 2018
User: root
git head id: f947a6daa6fac1891bd5bb5b2c88a08025c6e740
Current branch: HEAD

ARCHITECTURE: Linux

OS name: CentOS Linux
OS Version: 7
CPU speed: 2397Mhz
CPU cores: 4
Physical Memory: 16048 Mgb
Disk install: /dev/mapper/rootvg-root
Disk space total: 62G
Disk space available: 57G

Checking Yum installation
Yum installation found at /bin/yum.

Checking installation of Centos Software Collections Repository.
Centos Software Collections Repository found.


Enabling Centos devtoolset-7.

Unable to enable Centos devtoolset-7 at this time.

Exiting now
```

A：
参考网址：https://blog.csdn.net/lizhengjava/article/details/80269484

```
//现在依赖
sudo yum -y --enablerepo=extras install centos-release-scl

sudo yum install -y devtoolset-7

sudo yum install -y python33.x86_64

//配置系统环境
source /opt/rh/devtoolset-7/enable 
```