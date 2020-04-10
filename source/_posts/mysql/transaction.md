---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/mysql-conf/mysql.png
title: mysql 事务解析
date: 2019-06-20 20:30:00
toc: true
tags: 
- Mysql
categories:
- transaction
---
最近mysql碰到一个问题，用Navicat插入数据之后，界面上可以看到数据。应用死活查不到数据，期初怀疑是不是锁表了，后面查看锁的情况发现并没有锁表的情况。后面考虑到可能是事务的问题，由于我改过配置文件，我仔细排查一番
果然发现，我把autocommit设置为了0，也就是必须要commit才能提交事务。所以干脆把mysql整个事务情况都记录一下。
<!--more-->
# mysql 事务
事务是数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作；这些操作作为一个整体一起向系统提交，要么都执行、要么都不执行；事务是一组不可再分割的操作集合

# 事务的ACID特性
- **Atomicity 原子性**
   就像事务的定义一样，要么一起执行，要么都不执行，是最小的操作单元；中间任何一个操作出错，之前的操作都会被取消。   
   
- **Consistency 一致性**
   一致性是指事务操作通过AID的特性，保证了事务在执行后，依然满足约束；例如张三账户有90元，要转给李四100元，数据库约束余额不能小于0，所以这个事务必然执行不成功，应为没满足约束。

- **Isolation 隔离性**
   隔离性主要有两个特性
    
    1. 在一个事务执行过程中，数据的中间的(可能不一致)状态不应该被暴露给所有的其他事务。 
　　
    2. 两个并发的事务应该不能操作同一项数据。数据库管理系统通常使用锁来实现这个特征。 

- **Durability 持久性**
    一个被完成的事务的最后结果应该是持久的。

# 隐式事务和显式事务

mysql配置中有**autocommit**这一项，默认为1，开启自动提交，也就是隐式事务

针对SELECT、UPDATE、DELETE、INSERT等DQL及DML语句的执行，mysql会自动提交该事务，如果关闭就需要手动提交或者回滚来完成操作。

显示事务是指 设置**autocommit = 0**，在事务操作中，必须要有明显的开启或结束的标签

```sql
[START TRANSACTION]  # 可选的语句
[DELETE | UPDATE | INSERT | SELECT ]  # DML、DQL操作
[COMMIT | ROLLBACK];  #提交或者回滚
```

在显式事务中还存在回滚点的用法

```sql
START TRANSACTION;
[DELETE | UPDATE | INSERT | SELECT];  #回滚时要执行提交的部分
SAVEPOINT a;  # 设置回滚点，且变量名为a
[DELETE | UPDATE | INSERT | SELECT];  #回滚时不执行提交的部分
ROLLBACK TO a;  # 回滚时与ROLLBACK TO搭配使用
```
回滚点之前的操作会被commit，而回滚点之后的操作会被rollback

# 事务隔离级别

事务的隔离级别分为四种，隔离级别从左至右递增
```
graph LR
READ-UNCOMMITTED-->READ-COMMITTED
READ-COMMITTED-->REPEATABLE-READ默认
REPEATABLE-READ默认-->SERIALIZABLE
```

不同隔离级别所解决的事务并发问题

隔离级别/解决问题 | 脏读 | 	不可重复读 | 	幻读
---|---|---|---
READ UNCOMMITTED | 	1 | 	1 | 	1
READ COMMITTED |	0	 | 1 | 	1
REPEATABLE READ | 	0 | 	0 | 	1
SERIALIZABLE | 	0 | 	0 | 	0

**1. READ UNCOMMITTED  其隔离性最低，会出现脏读、不可重复读、幻读等所有情况。**

**2. READ COMMITTED级别能够避免脏读**

脏读是指对于两个事务T1与T2，T1读取了已经被T2更新但是还没有提交的字段之后，若此时T2回滚，T1读取的内容就是临时并且无效的

**3. REPEATABLE-READ避免不可重复读**

不可重复读是指对于两个事务T1和T2，T1读取了一个字段，然后T2更新了该字段并提交之后，T1再次提取同一个字段，值便不相等了。

**4. SERIALIZABLE避免幻读**

幻读是指对于两个事务T1和T2，T1读取了一个字段，然后T2插入了新字段并提交之后，T1再次提取，结果不一致了。

*注：不可重复读跟幻读最主要的区别是，不可重复读针对的是某一条记录产生了更新的情况，导致同一记录两次读取结果不相等。而幻读是指有新数据插入，导致两次查询整个结果不一致的情况*


**隔离级别的实现都是基于mysql存储引擎内部的锁实现，目前只有InnoDB支持事务，后期再讲一下InnoDB锁相关的知识**