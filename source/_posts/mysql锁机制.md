---
title: mysql锁机制
date: 2020-03-10 16:18:18
categories: 
- mysql
tags:
- 锁
---
## 简介
通过此文，你将学习mysql锁的相关内容：
1. 什么是锁
2. 锁都有哪些类型，每个类型的区别
3. 分析锁的使用情况和应用场景

### 概念
锁：mysql为了解决资源共享，所造成并发问题的处理机制。

锁根据粒度分为：
1. 表锁(粒度大,myisam引擎默认)
	1. 锁类型：
		1. 锁操作
			1. 加锁:lock table 表名 read/write;
			2. 解锁:unlock table;
			3. 观察表锁:show open tables; 
			4. 分析表锁:show status like "table_lock%";
			    1. Table_locks_immediate 可以立即添加锁的表数量
			    2. Table_locks_waited 需要等待的锁数量
			    3. Table_locks_immediate/Table_locks_waited >5000 建议使用innodb引擎，否则使用myisam
		2. 读锁(共享锁)影响
			1. 当前会话
				1. 可以对锁表进行读，但是写操作会处于等待状态，直到锁被释放
				2. 不可以其他表进行读写
			2. 其他会话
		3. 写锁(排他锁)影响
			1. 当前会话
			    1. 可以对锁表进行读写
			    2. 不可以对其他表进行读写
			2. 其他会话：
			    1. 对锁表进行的读写，会处于等待状态，直到锁被释放
			    2. 可以对其他表进行读写
	2. 优点
	    1. 资源占用少，加锁速度快
	    2. 不会出现死锁
	3. 缺点
		1. 并发度低
		2. 容易出现锁冲突
2. 行锁(粒度小,innodb引擎默认)
	1. 锁类型
	    1. 锁操作
	        1. 加锁
	            1. 写锁:进行写操作(insert,update,delete)时候，innodb会自动添加写锁，直到事务提交(commit)或者回滚(rollback)。在测试的时候，可以通过set autocommit=0;关闭事务自动提交
	            2. 读锁: select * from innodb_lock where id=1 for update;
	        2. 解锁:事务提交或者回滚
	    2. 读锁(共享锁)/写锁(排他锁)影响
			1. 当前会话
				1. 可以多锁定行和其他行进行读写
			2. 其他会话
				1. 可以对锁定行进行读，但是写操作会处于等待状态，直到锁被释放
				2. 可以对其他行进行读写				
	2. 优点
		1. 并发度高
	3. 缺点
		1. 资源占用大，加锁速度慢
		2. 会出现死锁
3. 行锁(粒度介于以上两者之间)

<!--more-->

### 表锁
#### 操作
```sql
/*读锁*/
lock table userinfo read;

/*写锁*/
lock table userinfo read;

/*解锁*/
unlock table userinfo;

/*查看表锁*/
show open tables;

/*查看表锁状态*/
show status like "table_lock%";
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 193   |
| Table_locks_waited    | 0     |
+-----------------------+-------+
```

#### 行锁
##### 操作
```sql
/*读锁*/
select * from userinfo where id = 1 for update;

/*查看行锁状态*/
show status like "innodb_row_lock%";
+-------------------------------+--------+
| Variable_name                 | Value  |
+-------------------------------+--------+
| Innodb_row_lock_current_waits | 0      |
| Innodb_row_lock_time          | 215019 |
| Innodb_row_lock_time_avg      | 43003  |
| Innodb_row_lock_time_max      | 51025  |
| Innodb_row_lock_waits         | 5      |
+-------------------------------+--------+
```
1. Innodb_row_lock_time 服务启动到现在，总共锁定的时间
2. Innodb_row_lock_time_avg 平均等待花费的时间
3. Innodb_row_lock_time_max 服务启动到现在，最大的等待时间
4. Innodb_row_lock_waits 服务启动到现在，总共等待的次数
5. Innodb_row_lock_current_waits 当前处于等待的锁数量

innodb引擎会自动给写(update,insert,delete)操作添加写锁。

通过事务提交或者回滚的时候进行解锁。

#### 注意点
##### 间隙锁
在添加行锁时候，innodb会自动给在条件范围内，但是没有值的字段添加行锁。例如where id >2 and id<9;即使此时没有id=7的字段，innndb也会自动给id=7自动添加锁，这个就是间隙锁。

##### 死锁
1. 原因:2个或者2个以上的进程，为了争夺同一资源，出现互相等待的情况。此时如果没有外力干预，将永远处于等待状况。
2. 如何避免
	1. 尽量避免资源的互相调用
	2. 按照同一顺序进行操作

##### 行锁失效
当索引失效或者没有索引的时候，行锁将自动转换为表锁