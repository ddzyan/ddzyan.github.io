---
title: 单表sql优化实战
date: 2020-02-26 16:42:41
categories: 
- mysql
tags:
- sql优化
- 索引
---
## 简介
模拟sql执行，根据返回结果分析和优化，从而提升sql执行效率。

注意的点：
1. 复合索引的创建顺序和使用顺序需要一致
2. 在范围查询中，如果使用了in ，则可能会导致索引失效
3. 根据sql执行顺序，合理创建复合索引，使之满足第一个条件
4. 正常将sql查询类型优化到 type=ref/range 就可以。

<!--more-->

### 预备数据
```sql
create table test(
	a int(2) not null,
	b int(2) not null,
	c int(2) not null,
	d int(2) not null
)engine=innodb default charset=utf8;
```
### 初始sql
```sql
explain select d from test where a in (1,2) and b=2 order by a;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | test  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
```

解析：
1. type=all 为全表查询，效率最低。一般需要优化到 ref/range 阶段
2. key=null 查询未使用索引
3. using filesort 多余了一次查询和排序，性能损耗大

根据 sql 执行顺序 from ... on ... join ... where ...group by ... having ... select dinstinct... order by ... limit ,创建符合执行顺序的复合索引。

### 优化,创建索引
```sql
create index idx_abd on test(a,b,d);
explain select d from test where a in (1,2) and b=2 order by a;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | test  | NULL       | index | idx_abd       | idx_abd | 12      | NULL |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
```

解析：
1. type=index 遍历整个索引表，性能相对all有所提升
2. key=idx_abd 使用到了索引进行查询
3. using index 遍历整个索引表
但是此时，where条件中存在in条件会使索引失效，从而使索引使用顺序与创建顺序不一致，导致复合索引。需要调整where顺序，并且修改复合索引创建顺序。

### 优化,调整查询语句,修改索引
```sql
drop index idx_abd on test;
create index inx_bad on test (b,a,d);
explain select d from test where  b=2 and  a in (1,2)  order by a;
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+--------------------------+
| id | select_type | table | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | test  | NULL       | ref  | inx_bad       | inx_bad | 4       | const |    2 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+--------------------------+
```

解析：
1. type=ref 查询索引表，返回匹配的数据，效率提升
2. key_len=4 这里只匹配到复合索引中b字段，并且b字段为int类型则占用4字节，符合要求
3. Using index 遍历所有表查询
4. using where in 语句使索引失效，所以导致需要进行回表查询