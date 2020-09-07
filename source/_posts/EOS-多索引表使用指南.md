---
title: EOS-多索引表使用指南
date: 2018-08-20 02:26:06
categories: 
- eos
tags:
- 区块链
- eos
---
参考网址：https://developers.eos.io/eosio-cpp/docs/multi-index-table-tutorial

## 简介
- EOSIO Multi-Index API为EOSIO数据库提供C ++接口。
- 源码文件路径：eos/contracts/eosiolib/multi_index.hpp
- 其内部使用的是 C 的 Ddatabase_api 来操作数据库，文件路径：eos/contracts/eosiolib/wasm_interface.cpp
- 表对象的创建和修改都需要花费RAM，删除则会返回RAM到账户中。

### 命名规则
#### 合约帐号名称
- 只能包含字符.abcdefghijklmnopqrstuvwxyz12345。a-z（小写）1-5和.（期间）
- 必须是12个字符

#### 表名
- 最多只能包含12个字母字符

#### 结构
- 最多只能包含12个字母字符

#### 代币符号
- 必须是A和Z之间的大写字母字符
- 必须是7个字符或更少
---

<!--more-->

### 使用教程
#### 定义存储结构
创建一个 EOSIO Multi-Index 表必需要一个uint64_t 主键。为了使表能够检索主键，存储在表中的对象需要具有名为 primary_key() 的 const 成员函数，该函数返回 uint64_t 。EOSIO Multi-Index表还支持最多16个二级索引.索引支持的格式如下：
- uint64_t
- uint128_t
- uint256_t
- double
- long double

#### 创建表
##### 例子
```c++
typedef eosio::multi_index<N(TableName), T,...Indices> table_name(code,scope);
```

##### 参数
- TableName - 表的名称
- T - 表中存储的数据类型
- Indices - 表的二级索引，此处支持最多16个索引
- code - 拥有表的帐户
- scope - 数据所属的账户
- table_name  - 存在在内存中的表名称

scope 负责新对象的存储使用，如果必须创建表（和二级索引表），则 scope 支付表创建的 RAM 开销。

#### 表的操作

##### 按二级索引排序
```c++
auto expidx = orders.get_index<N(byexp)>();

print("Items sorted by expiration:\n");
for( const auto& item : expidx ) {
  print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
}
```

##### 添加多索引表(emplace)
添加一个新的对象到表中

```c++
orders.emplace( payer, [&]( auto& o ) {
  o.id = 1;
  o.expiration = 300;
  o.owner = N(dan);
});
```

###### 参数
- payer - 付款账号消耗RAM
- constructor - 用于对要在表中创建的对象进行就地初始化

###### 返回值
新创建的对象的主键迭代器

###### 重点
- 在Multi-Index表中创建一个新对象，该对象具有唯一的主键（在对象中指定）。该对象被序列化并写入表中。如果该表不存在，则创建该表。
- 付款人负责新对象的存储使用，如果必须创建表（和二级索引表），则为表创建的开销。
- 更新辅助索引以引用新添加的对象。如果辅助索引表不存在，则创建它们。

##### 更新多索引表(modify)
在表中修改一个已经存在的对象

```c++
print("Modifying expiration of order with ID=2 to 400.\n");
orders.modify( order2, payer, [&]( auto& o ) {
  o.expiration = 400;
});
```

###### 参数
- order2 - 指向要更新的对象的迭代器
- payer - 更新行的存储使用情况的付款人的帐户名称
- updater - 更新目标对象的 lambda 函数

###### 重点
- 已修改的对象已序列化，然后替换表中的现有对象。
- 付款人需要为更新对象的存储使用付费。
- 如果付款人与现有付款人相同，则付款人仅支付现有和更新对象之间的使用差异（如果此差异为负，则退款）。
- 如果付款人与现有付款人不同，则现有付款人将退还现有对象的存储使用情况。

##### 返回
无返回值

#### erase(删)
使用主键删除一个表中已经存在的对象

```c++
addresses.erase(itr);
```

##### 参数
- itr - 指向要删除的对象的迭代器

##### 重点
- 该对象将从表中删除，并回收所有关联的存储。
- 对于对象的存储使用的现有支付者，删除的对象的表和二级索引使用，以及如果移除表和索引，相关的开销将退还。
- 在删除所有行后，表将自动删除。

##### 返回
返回指向删除对象后面的对象的指针。

##### find(查)
在表中根据主键查找已经存在的对象

```c++
auto itr = addresses.find(N(dan));
```

###### 参数
- primary -对象的主键值

###### 返回值
返回一个查询到的对象的迭代器；

如果没有查询到指定对象，返回一个end迭代器。

### 以上所有方法的例子
```
##### 例子
```c++
#include <eosiolib/eosio.hpp>
#include <eosiolib/dispatcher.hpp>
#include <eosiolib/multi_index.hpp>

using namespace eosio;

namespace limit_order_table {
    //定义结构体
    struct limit_order {
        uint64_t     id;
        uint128_t    price;
        uint64_t     expiration;
        account_name owner;

        auto primary_key() const { return id; }
        uint64_t get_expiration() const { return expiration; }
        uint128_t get_price() const { return price; }

        EOSLIB_SERIALIZE( limit_order, ( id )( price )( expiration )( owner ) )
    };

    class limit_order_table {
        public:

        ACTION( N( limitorders ), issue_limit_order ) {
            EOSLIB_SERIALIZE( issue_limit_order )
        };

        //定义合同方法
        static void on( const issue_limit_order& ilm ) {
            //获得表的归属者
            auto payer = ilm.get_account();

            print("Creating multi index table 'orders'.\n");

            //实例化表
            eosio::multi_index< N( orders ), limit_order, 
                indexed_by< N( byexp ),   const_mem_fun< limit_order, uint64_t, &limit_order::get_expiration> >,
                indexed_by< N( byprice ), const_mem_fun< limit_order, uint128_t, &limit_order::get_price> >
                > orders( N( limitorders ), N( limitorders ) );

            //添加多索引表
            orders.emplace( payer, [&]( auto& o ) {
                o.id = 1;
                o.expiration = 300;
                o.owner = N(dan);
            });

            auto order2 = orders.emplace( payer, [&]( auto& o ) {
                o.id = 2;
                o.expiration = 200;
                o.owner = N(thomas);
            });

            //主键排序
            print("Items sorted by primary key:\n");
            for( const auto& item : orders ) {
                print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
            }

            auto expidx = orders.get_index<N(byexp)>();
            
            //索引排序
            print("Items sorted by expiration:\n");
            for( const auto& item : expidx ) {
                print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
            }

            auto pridx = orders.get_index<N(byprice)>();

            print("Items sorted by price:\n");
            for( const auto& item : pridx ) {
                print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
            }

            print("Modifying expiration of order with ID=2 to 400.\n");

            //修改
            orders.modify( order2, payer, [&]( auto& o ) {
                o.expiration = 400;
            });

            //获得主键大于100的实例
            auto lower = expidx.lower_bound(100);

            print("First order with an expiration of at least 100 has ID=", lower->id, " and expiration=", lower->get_expiration(), "\n");
   };

} /// limit_order_table

namespace limit_order_table {
   extern "C" {
      /// 执行合同下任何方法都需要先经过此方法
      void apply( uint64_t code, uint64_t action ) {
         require_auth( code );
         eosio_assert( eosio::dispatch< limit_order_table, limit_order_table::issue_limit_order >( code, action ), "Could not dispatch" );
      }
   }
}
```