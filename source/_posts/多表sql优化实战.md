---
title: 多表sql优化实战
date: 2020-03-02 22:27:17
categories: 
- mysql
tags:
- sql优化
- 索引
---
## 简介
了解在多表查询的时候，如何使用索引来优化sql执行效率

多表查询，索引添加原则：
1. 根据程序循环设计原则：外循环为小循环，内循环为大循环得出--->小表驱动大表
2. 在表中索引建立在经常使用的字段

### 预备数据
```sql
create database demo;

use demo

create table teacher(
    tid bigint(11),
    name varchar(10),
    cid bigint(11)
)engine=innodb default charset=utf8;

create table course(
    cid bigint(11),
    title varchar(10)
)engine=innodb default charset=utf8;

insert into teacher values (1,"tz",1);
insert into teacher values (2,"td",1);
insert into teacher values (3,"zh",1);
insert into course values (1,"mysql");
```

<!--more-->

### 第一版本---基础sql
模拟执行sql，分析执行结果
```sql
/*左连接条件查询*/
explain select * from teacher t left join course c on c.cid=t.cid where c.name = "tz";
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | t     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where                                        |
|  1 | SIMPLE      | c     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
```

#### 分析结果
此时效率为最低，具体分析如下：
1. type=all 全表查询
2. key=null 未使用任何索引
3. extra Using where 全表遍历查询,效率低
4. extra Using join buffer mysql底层优化sql后，添加了连接缓存

此时需要对表添加索引，添加的索引如下：
1.teacher.name：name字段必须为索引，否则需要对teacher进行会表查询
2.原则为小表驱动大表，course 原则上数据会小于teacher数据，索引需要修改 where 条件顺序为 c.cid=t.cid
3.根据索引创建原则，需要将索引添加到经常使用到的字段，则为course.cid

### 第二版本---根据分析结果添加索引
```sql
alter table teacher add index idx_name (name);
alter table course add index idx_cid(cid);

explain select * from teacher t left join course c on  c.cid=t.cid where t.name = "tz";
+----+-------------+-------+------------+------+---------------+----------+---------+------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref        | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+----------+---------+------------+------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_name      | idx_name | 33      | const      |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | c     | NULL       | ref  | idx_cid       | idx_cid  | 9       | demo.t.cid |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+----------+---------+------------+------+----------+-------+
```

#### 分析结果
1. type=ref 遍历索引表，返回满足条件的数据（已经达到优化条件）
2. key=idx_name /key=idx_cid 两次查询都使用到了索引
3. name 最大字节数量 33 = 10(长度) * 3(utf8最大字节) + 3(mysql底层用3字节标识可以为null)，满足预期
4. cid 最大字节数量 9 = 8(bigint存储字节长度) + 3(mysql底层用3字节标识可以为null)，满足预期