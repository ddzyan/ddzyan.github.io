---
title: EOS正式链部署
date: 2018-08-09 05:44:51
categories: 
- eos
tags:
- 区块链
- eos
---
参考网址：https://github.com/CryptoLions/EOS-MainNet
# 安装Eos
## step1：下载源码进行安装

```
mkdir /home/eos-sources  
cd /home/eos-sources  

git clone https://github.com/EOS-Mainnet/eos.git --recursive    
cd eos  

git checkout mainnet-1.0.6  
git submodule update --init --recursive   

./eosio_build.sh   
```

<!--more-->

## step2：配置node

```
mkdir /opt/EOSmainNet 

cd /opt/EOSmainNet
git clone https://github.com/CryptoLions/EOS-MainNet.git ./ 

chmod -R 777 ./*.sh   
chmod -R 777 ./Wallet/*.sh 

```

## step3：修改config.ini文件
添加：filter-on = * 增加过滤器选项，可以查询账户相关交易。不建议开启，会造成内存溢出。

[查询最新p2p地址](：https：//eosnodes.privex.io/？config=1)，进行添加

# step4：检查genesis.json
initial_key："EOS7EarnUhcyYqmdnPon8rm7mBCTnBoot6o7fE2WzjvEX2TdggbL3"

不相同则无法进行同步区块

## step5：操作指令

```
//第一次运行则运行，删除区块和配置genesis.json
./start.sh --delete-all-blocks --genesis-json genesis.json

//启动节点
./start.sh  

//启动钱包
./Wallet/start_wallet.sh  

```

## step6：验证是否安装成功

```
//其中chain_id等于：aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906
./cleos get info 
{
  "server_version": "c9b7a247",
  "chain_id": "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906",
  "head_block_num": 4113455,
  "last_irreversible_block_num": 4113128,
  "last_irreversible_block_id": "003ec2e8a76b9b219ce53eb854c11024e6d2d17c66c603ecbbdbb4232ed3578e",
  "head_block_id": "003ec42fa45fb82eee5dc532213c396aa5ed8a114e382387a05d79b8e699e7aa",
  "head_block_time": "2018-07-04T11:47:07.000",
  "head_block_producer": "eosnewyorkio",
  "virtual_block_cpu_limit": 200000000,
  "virtual_block_net_limit": 1048576000,
  "block_cpu_limit": 199900,
  "block_net_limit": 1048576
}

```

# 更新EOS

```
cd /opt/EOSmainNet 
./stop.sh
cd /home/eos-sources/eos  
rm -rf build/

//第一种
git checkout mainnet-1.0.6
git submodule update --init --recursive   
./eosio_build.sh 

//第二种下载最新zip安装包
unzip -o eos-1.0.8.zip
./eosio_build.sh 

```