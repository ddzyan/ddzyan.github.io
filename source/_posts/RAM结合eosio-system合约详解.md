---
title: RAM结合eosio.system合约详解
date: 2018-08-09 03:06:52
categories: 
- eos
tags: 
- 区块链
- eos
---
### 源码分析
#### eosio.system.cpp
```
// native.hpp (newaccount definition is actually in eosio.system.cpp)
(newaccount)(updateauth)(deleteauth)(linkauth)(unlinkauth)(canceldelay)(onerror)
// eosio.system.cpp
(setram)(setparams)(setpriv)(rmvproducer)(bidname)
// delegate_bandwidth.cpp
(buyrambytes)(buyram)(sellram)(delegatebw)(undelegatebw)(refund)
// voting.cpp
(regproducer)(unregprod)(voteproducer)(regproxy)
// producer_pay.cpp
(onblock)(claimrewards)
```
ram,cpu和net操作相关方法的都定义在 **delegate_bandwidth.cpp** ，其中和 RAM 相关的是 buyrambytes (通过指定字节数购买ram), buyram (通过指定货币购买ram)和 sellram (销售ram)。

#### delegate_bandwidth.cpp
```
//根据当前市场的份额，将需要购买的字节数转化为指定的EOS进行购买
void system_contract::buyrambytes( account_name payer, account_name receiver, uint32_t bytes ) {
      //在数据库中查询RAMCORE发行量，默认为100000000000000
      auto itr = _rammarket.find(S(4,RAMCORE));
      auto tmp = *itr;
      auto eosout = tmp.convert( asset(bytes,S(0,RAM)), CORE_SYMBOL );
      //通过转化后，调用buyram使用EOS购买
      buyram( payer, receiver, eosout );
   }
```
##### 解析
RAM的交易机制采用Bancor算法，使每字节的价格保持不变，通过中间代币(RAMCORE)来保证EOS和RAM之间的交易流通性。从上源码看首先获得RAMCORE的发行量，再通过tmp.convert方法RAM->RAMCORE，RAMCORE->EOS(CORE_SYMBOL)再调用 buyram 进行购买。这里的CORE_SYMBOL不一定是指EOS，查看core_symbol.hpp，发现源码内定义为SYS，也就是说在没有修改的前提下，需要提前发行SYS代币，才能进行RAM购买。

<!--more-->

#### core_symbol.hpp
```
#define CORE_SYMBOL S(4,SYS)
```
#### delegate_bandwidth.cpp
```
void system_contract::buyram(account_name payer, account_name receiver, asset quant)
{
      //验证权限
      require_auth(payer);
      //不能为0
      eosio_assert(quant.amount > 0, "must purchase a positive amount");

      auto fee = quant;
      //手续费为0.5%，如果amoun为1则手续费为1，如果小于1，则手续费在amoun<free<1 quant.amount.
      //扣除手续费后的金额，也就是实际用于购买RAM的金额。如果扣除手续费后，金额为0，则会引起下面的操作失败。
      auto quant_after_fee = quant;
      quant_after_fee.amount -= fee.amount; the next inline transfer will fail causing the buyram action to fail.

      INLINE_ACTION_SENDER(eosio::token, transfer)
      (N(eosio.token), {payer, N(active)},
       {payer, N(eosio.ram), quant_after_fee, std::string("buy ram")});

      //如果手续费大于0，则将手续费使用eosio.token合约中的transfer，将金额转移给eosio.ramfee，备注为ram fee
      if (fee.amount > 0)
      {
            INLINE_ACTION_SENDER(eosio::token, transfer)
            (N(eosio.token), {payer, N(active)},
             {payer, N(eosio.ramfee), fee, std::string("ram fee")});
      }

      int64_t bytes_out;

      //根据ram市场里的EOS和RAM实时汇率计算出能够购买的RAM总量
      const auto &market = _rammarket.get(S(4, RAMCORE), "ram market does not exist");
      _rammarket.modify(market, 0, [&](auto &es) {
            //转化方法请参考下半部分
            bytes_out = es.convert(quant_after_fee, S(0, RAM)).amount;
      });

      //剩余总量大于0判断
      eosio_assert(bytes_out > 0, "must reserve a positive amount");

      //更新全局变量，总共可以使用的内存大小
      _gstate.total_ram_bytes_reserved += uint64_t(bytes_out);
      //更新全局变量，购买RAM冻结总金额
      _gstate.total_ram_stake += quant_after_fee.amount;

      user_resources_table userres(_self, receiver);
      auto res_itr = userres.find(receiver);
      if (res_itr == userres.end())
      {
            //在userres表中添加ram相关记录
            res_itr = userres.emplace(receiver, [&](auto &res) {
                  res.owner = receiver;
                  res.ram_bytes = bytes_out;
            });
      }
      else
      {
            //在userres表中修改ram相关记录
            userres.modify(res_itr, receiver, [&](auto &res) {
                  res.ram_bytes += bytes_out;
            });
      }
      //更新账号的RAM拥有量
      set_resource_limits(res_itr->owner, res_itr->ram_bytes, res_itr->net_weight.amount, res_itr->cpu_weight.amount);
}
```
##### 解析
```c++
INLINE_ACTION_SENDER(eosio::token, transfer)
            (N(eosio.token), {payer, N(active)},
             {payer, N(eosio.ramfee), fee, std::string("ram fee")});
```
INLINE_ACTION_SENDER为内联调用其他合约交易的方法，这里 transfer 参数对应为：合约名称，交易行为，合约账号，授权账号，支付账号，接收账号，金额，备注。具体的参数根据调用的合约交易方法而定。

```c++
user_resources_table  userres( _self, receiver );
```
user_resources_table 为 typedef eosio::multi_index< N(userres), user_resources> 的重命名，这个方法是引用 multi_index.hpp ,这里定义对DB的所有操作，后续会专门分析。通过查询获取账号在DB中是否有值，进行操作。

```c++
 set_resource_limits( res_itr->owner, res_itr->ram_bytes, res_itr->net_weight.amount, res_itr->cpu_weight.amount );
```
更新账号的限制资源，此方法最终执行的为到 pending_limits 表中一条购买的资源数据。

参数：
- ram_bytes - ram limit
- net_weight - net limit
- cpu_weight - cput limit

相关注释已经在代码中，这里还会用到一个比较重要的内容，那就是代币转化为RAM的公式，此方法请参考下面。
#### exchange_state.cpp
```
asset exchange_state::convert_to_exchange( connector& c, asset in ) {
      real_type R(supply.amount);   //RAM已经售出的总量
      real_type C(c.balance.amount+in.amount);  //RAM总购买金额+本次购买的量
      real_type F(c.weight/1000.0);
      real_type T(in.amount);
      real_type ONE(1.0);

      real_type E = -R * (ONE - std::pow( ONE + T / C, F) );//换算出EOS对应的RAM量
      //print( "E: ", E, "\n");
      int64_t issued = int64_t(E);

      supply.amount += issued;//更新RAM已经售出的总量
      c.balance.amount += in.amount;//更新RAM总购买金额

      return asset( issued, supply.symbol );
   }
   
asset exchange_state::convert( asset from, symbol_type to ) {
      auto sell_symbol  = from.symbol;  
      auto ex_symbol    = supply.symbol; 
      auto base_symbol  = base.balance.symbol;  
      auto quote_symbol = quote.balance.symbol;
      
     //根据币种转化可以购买的RAM量
      if( sell_symbol != ex_symbol ) {
         if( sell_symbol == base_symbol ) {
            from = convert_to_exchange( base, from );
         } else if( sell_symbol == quote_symbol ) {
            from = convert_to_exchange( quote, from );
         } else { 
            eosio_assert( false, "invalid sell" );
         }
      } else {
         if( to == base_symbol ) {
            from = convert_from_exchange( base, from ); 
         } else if( to == quote_symbol ) {
            from = convert_from_exchange( quote, from ); 
         } else {
            eosio_assert( false, "invalid conversion" );
         }
      }

      if( to != from.symbol )
         return convert( from, to );

      return from;
   }
```

convert_to_exchange的转化公式如下：
![image|432x98](/images/ram-gs.jpg)

ram购买的量是一个绝对值，是根据购买EOS的金额和当前市场内ram的数量计算出来的。一般来说在ram总量不增加的情况下，一样金额的EOS，所能获得的ram会越来越少。所以如果早期你购买了ram，然后过段时间后通过sellram卖掉ram可能还能挣钱。

---
### 总结
#### ram购买流程图
![image|690x488](/images/ram-step.png)

购买RAM最终将以添加到数据库里的值为准，也就是说明每次的查询可用量都将根据本地数据库同步到的数据为准。接下来将主要分析multi_index请参考：http://111.231.215.55/posts/eos_multi-index.html