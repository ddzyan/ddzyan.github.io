---
title: 'EOS自动化部署,搭建私有环境'
date: 2018-08-09 05:43:20
categories: 
- eos
tags:
- 区块链
- eos
---
- eosio版本：v1.0.5
- 操作系统：centos 7
- 参考网址为：https://developers.eos.io/eosio-nodeos/docs/local-single-node-testnet

#### step1 编译代码
1. 获取git代码

```
git clone https://github.com/EOSIO/eos --recursive
```
如果未添加 --recursive，则在拉完代码后，运行

<!--more-->

```
cd eos
git submodule update --init --recursive
```

2. 切换分支

```
cd eos 
git checkout v1.0.5

```
3. 编译

```
sudo ./eosio_build.sh -s "EOS" 
```
得到如下结果，则为成功
```
(  ____ \(  ___  )(  ____ \\__   __/(  ___  )
	| (    \/| (   ) || (    \/   ) (   | (   ) |
	| (__    | |   | || (_____    | |   | |   | |
	|  __)   | |   | |(_____  )   | |   | |   | |
	| (      | |   | |      ) |   | |   | |   | |
	| (____/\| (___) |/\____) |___) (___| (___) |
	(_______/(_______)\_______)\_______/(_______)

	EOSIO has been successfully built. 00:08:30

	To verify your installation run the following commands:

	/root/opt/mongodb/bin/mongod -f /root/opt/mongodb/mongod.conf &
	source /opt/rh/python33/enable
	export PATH=${HOME}/opt/mongodb/bin:$PATH
	cd /data/home/admin/eos-private/eos/build; make test

	For more information:
	EOSIO website: https://eos.io
	EOSIO Telegram channel @ https://t.me/EOSProject
	EOSIO resources: https://eos.io/resources/
	EOSIO Stack Exchange: https://eosio.stackexchange.com
	EOSIO wiki: https://github.com/EOSIO/eos/wiki
```
启动节点

```
[项目路径]/eos/build/programs/nodeos/nodeos
```
你也可以设置全局，随时随地启动

```
cd eos/build
sudo make install
```

#### step2 配置环境（本地单节点）
配置文件所在位置

```
~/.local/share/eosio/nodeos/config
```
配置文件修改内容
```
#设置可以访问节点的ip，0.0.0.0为所有IP都可以
http-server-address = 0.0.0.0:8888
#设置是否可以生产块
enable-stale-production = true
#设置生产者名称
producer-name = eosio
#是否启动过滤
filter-on = *
#生产者密钥
private-key = ["EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"]
#插件列表
plugin = eosio::chain_api_plugin
plugin = eosio::chain_plugin
plugin= eosio::producer_plugin
plugin = eosio::http_plugin
plugin = eosio::history_api_plugin
```
重新启动节点nodeos

```
ps -A|grep nodeos

kill [pid]
```

节点配置
```
创建钱包
cleos wallet create -n kgbp
返回钱包密码
PW5Ki8SMwkxfuvtXWy7fgj9FcGWdWfKQ6kfoq9KGHUvPoUfW6M8fx
记下密码，以后解锁要用

1.打开钱包
cleos wallet open -n kgbp

2.解锁钱包
cleos wallet unlock -n kgbp

3.生成密钥对
cleos create key

4.创建eosio.token账号
cleos create account eosio eosio.token PublicKey PublicKey

5.密钥导入钱包
cleos wallet import PrivateKey -n kgbp

6.发布eosio.token合约
cleos set contract eosio.token ./contracts/eosio.token/

7.创建和发布代币
cleos  push action eosio.token create '["eosio","1000000000.0000 EOS",0,0,0]' -p eosio.token
cleos  push action eosio.token issue '["eosio","1000000000.0000 EOS","issue"]' -p eosio

8.在通过3,4,5创建一个自己的账号测试转账

9.进行转账
cleos  push action eosio.token transfer '["eosio","eosioddztest","100.0000 EOS","58tsncxlb7nq0"]' -p eosio

10.查看余额
cleos transfer eosio ddzcash "1.0000 EOS" "issue"
```
