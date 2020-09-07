---
title: sql优化之排序
date: 2020-03-06 15:14:41
categories: 
- mysql
tags:
- sql优化
- 索引
- 排序
---
## 简介
本文介绍在sql中进行 order by 操作的时候，如何进行分析和优化sql执行效率。

order by 索引优化总结：
1. 避免出现 using filesort(文件排序)，常见的方法如下：
    1. 单值索引：
    	1. order by列 和 select列一致，可以实现using index(索引覆盖)
    	2. order by 只能使用一个索引，无法同时使用多个索引
    2. 复合索引：
    	1. order by列的顺序需要和复合索引创建顺序一致，不能跨列(最佳左前缀)
    	2. where + order by 列需要和索引创建顺序一致，不能跨列(最佳左前缀)
    	3. 不能对 order by 列同时进行正向和反向排序

<!--more-->

### 预备概念
索引：将表中的一列或者多列按照指定的规则进行排序的数据结果。

索引覆盖：通过遍历索引表，就可以返回满足条件的数据

sql编写顺序：select ... distinct ... join ... on .... where ... group by ... having ... order by limit

sql执行顺序：from ... join ... on ... where ... group by ... having ... select ... distinct ... order by limit

### 预备数据
```sql
create database demo;

use demo

create table book(
	bid bigint(11),
	author varchar(10),
	typeid bigint(11)
)engine=innodb default charset=utf8;

insert into book values (1,'tz',1);
insert into book values (2,'tz',2);
insert into book values (3,'tz',3);
insert into book values (4,'tw',1);
insert into book values (5,'ta',1);
```
### order by 避免use filesort
#### order by列 和 select列 需要为同一个索引
```sql
explain select typeid from book order by author;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | book  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+

/*一致则索引覆盖*/
alter table book add index idx_author(author);
explain select author from book order by author;
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key        | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_author | 33      | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+-------------+
```

#### 每次排序只能使用一个索引，不能同时多个
```sql
alter table book add index idx_typeid(typeid);
/*Using filesort 无法使用多个索引*/
explain select author from book order by author,typeid;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | book  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
```

##### 当需要多列进行排序时，需要创建复合索引
```sql
alter table book add index idx_author_typeid(author,typeid);
explain select author from book order by author,typeid;
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_author_typeid | 42      | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
```

##### order by 多列的使用顺序，需要和复合索引的创建顺序一致
```sql
/*此时复合索引的顺序为author_typeid，不一致则出现use filesort*/
explain select author from book order by typeid,author;
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_author_typeid | 42      | NULL |    5 |   100.00 | Using index; Using filesort |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------------------+
```

##### order by 多列的时候不能同时进行正向和反向排序
```sql
explain select author from book order by author,typeid desc;
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_author_typeid | 42      | NULL |    5 |   100.00 | Using index; Using filesort |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------------------+
```

##### (using index 最优方案) order by 列和 select 列一致，可以实现索引覆盖
```sql
/*此时复合索引的顺序为author_typeid*/
explain select author,typeid from book order by author,typeid;
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_author_typeid | 42      | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+

/*select 列顺序无关*/
explain select typeid,author from book order by author,typeid;
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_author_typeid | 42      | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
```


### wher + order by 优化
首先删除全部索引，避免干扰
```sql
drop index idx_author on book;
drop index idx_typeid on book;
drop index idx_author_typeid on book;
```
#### 单值索引使用中：where ，order by 和 select 需要使用一个索引列
```sql
create index idx_author on book(author);
create index idx_typeid on book(typeid);
explain select author from book where  author='tz' order by typeid;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | book  | NULL       | ALL  | idx_author    | NULL | NULL    | NULL |    5 |    60.00 | Using where; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+

/*在 where 和 order by 为同一单值索引时候，select 也为索引列，则可以索引覆盖*/
explain select author from book where  author='tz' order by author;
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | book  | NULL       | ref  | idx_author    | idx_author | 33      | const |    3 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------------+
```

#### 如果 where 和 order by 不为一个索引，则需要创建复合索引
```sql
/*根据where和order by 顺序，创建复合索引*/
alter table book add index idx_author_typeid(author,typeid);
explain select author from book where  author='tz' order by typeid;
+----+-------------+-------+------------+------+------------------------------+-------------------+---------+-------+------+----------+--------------------------+
| id | select_type | table | partitions | type | possible_keys                | key               | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-------+------------+------+------------------------------+-------------------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | book  | NULL       | ref  | idx_author,idx_author_typeid | idx_author_typeid | 33      | const |    3 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+------+------------------------------+-------------------+---------+-------+------+----------+--------------------------+
```

##### 复合索引的创建顺序和 where+order by 顺序需要一致
```sql
explain select bid from book where  typeid=1 order by author;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | book  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |    20.00 | Using where; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
```
---

### 拓展知识
在mysql底层会使用两种排序方式：
1. index_sort 索引排序，效率高
2. file_sort 文件排序，效率低

#### file_sort
在文件排序中，根据I/O读取次数又分为了：
1. 双路排序(mysql 4.1以前使用，2次I/O操作，效率相对低)：从表中获取符合where条件的order by 列 和行索引，添加到排序 buffer 中进行排序，根据排序后结果的行指针，扫描全表输出结果。
2. 单路排序(mysql 4.1以后使用，1次I/O操作，效率相对高)：从表中获取符合where条件的所有数据(order by 列和select .... distinct 列)，添加到排序buffer中，再给根据order by 列进行排序后输出结果。
		1. 如果获取的数据超过了排序 buffer 长度，则mysql会自动转为为双路排序，进行多次获取和排序。
		2. 可以通过修改 buffer 长度和减少不必要的列添加到buffer中，来减少I/O次数。


```sql
/*修改排序buffer长度*/
set max_length_of_sort_data=1024;
```




