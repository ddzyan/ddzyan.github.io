---
title: EOS的Unlinkable_block问题解决
date: 2018-08-09 05:59:22
categories: 
- eos
tags:
- 区块链
- eos
---
eos版本：v1.1.1
操作系统：centos 7 

参考地址：
- https://github.com/EOSIO/eos/issues/4986
- https://github.com/EOSIO/eos/pull/5001

<!--more-->

### 问题
区块同步停止，查看nodeos输出如下
```
2018-08-05T04:07:34.613 thread-0   producer_plugin.cpp:310       on_incoming_block    ] 3030001 unlinkable_block_exception: Unlinkable block
unlinkable block
    {"id":"0088f55a6f0609ced742278dbc1a0c9bd2660d211187384c63d19ea10d3b65b1","previous":"0088f5596c6ce2d0797beb3306969c9276f5b86fd7c785af778d120201d6ae5b"}
    thread-0  fork_database.cpp:151 add
rethrow
    {}
    thread-0  controller.cpp:924 push_block
2018-08-05T04:07:34.614 thread-0   controller.cpp:924            push_block           ] 3030001 unlinkable_block_exception: Unlinkable block
unlinkable block
    {"id":"0088f55b290b9bf807736545a1818989243505db569b5546a7890c27f761aab6","previous":"0088f55a6f0609ced742278dbc1a0c9bd2660d211187384c63d19ea10d3b65b1"}
    thread-0  fork_database.cpp:151 add
2018-08-05T04:07:34.614 thread-0   producer_plugin.cpp:310       on_incoming_block    ] 3030001 unlinkable_block_exception: Unlinkable block
unlinkable block
    {"id":"0088f55b290b9bf807736545a1818989243505db569b5546a7890c27f761aab6","previous":"0088f55a6f0609ced742278dbc1a0c9bd2660d211187384c63d19ea10d3b65b1"}
    thread-0  fork_database.cpp:151 add
rethrow
    {}
    thread-0  controller.cpp:924 push_block
2018-08-05T04:07:34.614 thread-0   controller.cpp:924            push_block           ] 3030001 unlinkable_block_exception: Unlinkable block

```

### 解决
将eosio升级到1.1.2，此版本已解决此问题。

修改内容：
1. transaction_context包含新的布尔字段explicit_billed_cpu_time，如果billed_cpu_time_us显式设置，则为true （在验证块时始终发生）。如果为真，transaction_context将永远不会更改billed_cpu_time_us字段的值explicit_billed_cpu_time。
1. 在soft_fail的情况下，节点现在将始终尝试在执行原始延迟事务的可计费CPU时间之上正确计算错误处理程序执行的可计费CPU时间，该时间已失败并导致错误处理程序运行首先。
1. 在hard_fail的情况下，节点将始终尝试计算适当的金额（到目前为止可用的可计费CPU时间），但正确地受到协议规则允许其充电的最大值的限制。这可以防止控制器add_transaction_usage以不适当的高额调用，这会导致它抛出一个传播的异常，直到它导致块的生成失败。


