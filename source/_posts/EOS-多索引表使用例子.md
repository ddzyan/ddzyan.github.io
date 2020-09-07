---
title: EOS-多索引表使用例子
date: 2018-08-25 01:13:50
categories: 
- eos
tags:
- 区块链
- eos
---
### 简介
### 简介
阅读本文前，请先阅读：http://111.231.215.55/posts/eos_multi-index.html

将完成一个任务清单计划的智能合约，功能有：
1. 发布任务
2. 完成任务
3. 获得全部任务

所有代码将在博文底部展示

#### 定义表结构
```c++
// @abi table todos i64
struct todo
    {
        uint64_t id;
        std::string description;
        uint64_t completed;

        uint64_t primary_key() const { return id; }
        EOSLIB_SERIALIZE(todo, (id)(description)(completed))
    };
```
##### 注意
// @abi table

编译器eosiocpp 通过识别 //@abi 公开此表，并使其在智能契约之外可见。

结构名称少于12个字符，全部为小写。

<!--more-->

#### typedef多索引表并定义索引
```c++
typedef eosio::multi_index<N(todos), todo> todo_table;
```
定义一个表名是 N(todos) ，结构体是 todo 的多索引表。N(mystruct)执行struct name到uint64_t的编译转换，这个uint64_t用于标识属于多索引表的数据。

#### 创建局部变量
```
todo_table todos;
```
现在已经定义了一个多索引表，名称为 todos ，我们可以在接下来的智能合约中使用它。

#### 定义操作方法
```c++
todo_contract(account_name self) : eosio::contract(self), todos(_self, _self) {}

    void create(account_name author, const uint32_t id, const std::string &description);

    void destroy(account_name author, const uint32_t id);

    void complete(account_name author, const uint32_t id);
```
- create -创建任务表
- destroy -删除任务表
- complete - 更新任务表

#### 全部源码
doit.cpp
```c++
#include "doit.hpp"


void todo_contract::create(account_name author, const uint32_t id, const std::string &description)
{
  todos.emplace(author, [&](auto &new_todo) {
    new_todo.id = id;
    new_todo.description = description;
    new_todo.completed = 0;
  });

  eosio::print("todo#", id, " created");
}

void todo_contract::destroy(account_name author, const uint32_t id)
{
  auto todo_lookup = todos.find(id);
  todos.erase(todo_lookup);

  eosio::print("todo#", id, " destroyed");
}

void todo_contract::complete(account_name author, const uint32_t id)
{
  auto todo_lookup = todos.find(id);
  eosio_assert(todo_lookup != todos.end(), "Todo does not exist");

  todos.modify(todo_lookup, author, [&](auto &modifiable_todo) {
    modifiable_todo.completed = 1;
  });

  eosio::print("todo#", id, " marked as complete");
}
EOSIO_ABI(todo_contract, (create)(complete)(destroy))
```

doit.hpp
```c++
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

class todo_contract : public eosio::contract
{
  public:
    todo_contract(account_name self) : eosio::contract(self), todos(_self, _self) {}

    void create(account_name author, const uint32_t id, const std::string &description);

    void destroy(account_name author, const uint32_t id);

    void complete(account_name author, const uint32_t id);

  private:
    // @abi table todos i64
    struct todo
    {
        uint64_t id;
        std::string description;
        uint64_t completed;

        uint64_t primary_key() const { return id; }
        EOSLIB_SERIALIZE(todo, (id)(description)(completed))
    };

    typedef eosio::multi_index<N(todos), todo> todo_table;
    todo_table todos;
};
```

#### 使用和调试
参考博文：http://111.231.215.55/posts/eos_contract_build.html