---
title: EOS资源系统
date: 2018-08-09 05:49:08
categories: 
- eos
tags:
- 区块链
- eos
---
### EOS资源系统
1. 一类是RAM购买型；另一类是CPU和网络带宽抵押型。
1. 把EOS理解为区块链操作系统，CPU和NET带宽是跟时间相关。一定时间内你可使用的CPU和NET带宽是有限的，可使用量和你抵押的EOS数量相关。随着时间的流逝，CPU和NET带宽会慢慢恢复，而抵押的EOS在不用时可以全部退回。
1. ram是EOS唯一消耗性资源，用完需要再购买，设计目的为减少无意义的交易，消耗整个的ram资源。

<!--more-->

#### 注意
一个EOS账号创建的时候可以通过购买和抵押来获取资源，费用由主账号支付。如果创建账号时并未购买资源，则需要通过其他账号购买或者进行抵押，因为购买和抵押本身就是一种交易行为需要消耗资源。

#### RAM
RAM是运行时的内存。在EOSIO系统中，数据存储在区块链中要消耗该资源，是DApp开发时必须的资源。

##### RAM的交易方式
1. RAM 的买卖，实质上是抵押 eos 到系统账户，而不是买方和卖方直接的交易。
1. 不论是购买ram(即抵押eos，获取ram)，还是卖出ram(即取回抵押的eos，释放ram)，都是参与者与系统账户之间的交互，该过程将会收取0.5%的手续费。
1. 买入RAM有两种计价方式： 买多少字节的RAM；买多少EOS的RAM。
1. 卖出RAM只有一种方式：多少字节的RAM。

##### RAM相关网站
1. RMA价格实时查询：https://bloks.io/
1. RAM购买：https://eostoolkit.io/account/undelegate

#### NET 带宽
网络带宽以过去3天的平均消耗量为单位进行测量，单位是字节如KB。每次发送操作或事务时都会暂时消耗网络带宽，但随着时间的推移会减少到0。抵押的EOS越多，网络带宽可以使用得越多。 可以随时赎回EOS，但是有三天左右的赎回期。

#### CPU 带宽
CPU带宽以过去3天的平均消耗（以微秒ms为单位）来衡量。 当您发送操作或事务时，CPU带宽会暂时消耗，但随着时间的推移会减少到0。事务运行时间越长，它将消耗的CPU带宽就越多。 可以随时赎回EOS，但是有三天左右的赎回期。

#### 三者区别
1. RAM是自由市场买卖模式，由市场价格来决定。CPU、NET是抵押模式，抵押多少取消多少。
1. RAM是随时可以交易，但CPU、NET有三天等待期。
1. CPU和NET可用于出租给其他账户，取消抵押后，EOS可以回到自己的账户。RAM可帮助其他账户购买，但卖出时的EOS归其他账户所有。

资源操作

```
//账号资源查看
cleos get account ${account}

//${account1}为支付账号，${account2}获得账号
//ram购买
cleos system buyram ${account1} ${account2}  "0.0001 EOS" 
//ram销售，最多能售出的数量limit - used
cleos system sellram ${account1} 68718 -p ${account1}

//抵押EOS，获得CPU带宽和NET带宽
cleos system delegatebw ${account1} ${account2} '0.1000 EOS'  '0.1000 EOS'  -p ${account1}
//取消抵押
cleos system undelegatebw ${account1} ${account2} '1 EOS' '1 EOS' -p ${account1}
```
