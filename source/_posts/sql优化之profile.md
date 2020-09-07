---
title: sql优化之profile
date: 2020-03-07 18:31:06
categories: 
- mysql
tags:
- sql分析工具
---
## 简介
mysql profile 工具可以记录执行sql总消耗的时间，并且记录系统cpu和i/o锁消耗的时间。

此工具建议只在开发环境中启动，生产环境应该关闭，减少资源消耗。

<!--more-->

### 使用
```sql
/*查看profile配置*/
show variables like "%profiling%";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | OFF   |
| profiling_history_size | 15    |
+------------------------+-------+

/*启动*/
set profiling = 1;
show variables like "%profiling%";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| have_profiling         | YES   |
| profiling              | ON    |
| profiling_history_size | 15    |
+------------------------+-------+

/*查看分析数据*/
show profile for query 3;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000072 |
| checking permissions | 0.000006 |
| Opening tables       | 0.000016 |
| init                 | 0.000014 |
| System lock          | 0.000008 |
| optimizing           | 0.000004 |
| statistics           | 0.000014 |
| preparing            | 0.000009 |
| executing            | 0.000001 |
| Sending data         | 0.013190 |
| end                  | 0.000011 |
| query end            | 0.000009 |
| closing tables       | 0.000005 |
| freeing items        | 0.000048 |
| cleaning up          | 0.000009 |
+----------------------+----------+

show profile all for query 3;

/*主要查看cpu和I/O消耗时间*/
show profile cpu , block io for query 3;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000072 | 0.000000 |   0.000000 |         NULL |          NULL |
| checking permissions | 0.000006 | 0.000000 |   0.000000 |         NULL |          NULL |
| Opening tables       | 0.000016 | 0.000000 |   0.000000 |         NULL |          NULL |
| init                 | 0.000014 | 0.000000 |   0.000000 |         NULL |          NULL |
| System lock          | 0.000008 | 0.000000 |   0.000000 |         NULL |          NULL |
| optimizing           | 0.000004 | 0.000000 |   0.000000 |         NULL |          NULL |
| statistics           | 0.000014 | 0.000000 |   0.000000 |         NULL |          NULL |
| preparing            | 0.000009 | 0.000000 |   0.000000 |         NULL |          NULL |
| executing            | 0.000001 | 0.000000 |   0.000000 |         NULL |          NULL |
| Sending data         | 0.013190 | 0.015625 |   0.000000 |         NULL |          NULL |
| end                  | 0.000011 | 0.000000 |   0.000000 |         NULL |          NULL |
| query end            | 0.000009 | 0.000000 |   0.000000 |         NULL |          NULL |
| closing tables       | 0.000005 | 0.000000 |   0.000000 |         NULL |          NULL |
| freeing items        | 0.000048 | 0.000000 |   0.000000 |         NULL |          NULL |
| cleaning up          | 0.000009 | 0.000000 |   0.000000 |         NULL |          NULL |
+----------------------+----------+----------+------------+--------------+---------------+

/*关闭profile*/
set profiling = 0; 
```

结果分析：https://www.cnblogs.com/flzs/p/9974822.html