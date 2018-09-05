---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/sql/join/sql_join.png
title: sql的优化
date: 2017-11-15 18:00:00
toc: true
tags: 
- SQL Mysql
categories:
- mysql
---
在应用开发初期，由于数据量小，开发人员写sql时更加注重功能实现，但当应用上线后，数据量的上升，很多sql的性能问题就逐渐显露出来，成为系统性能的瓶颈，下面介绍一些优化方法
<!--more-->

# 分析

不管面对什么问题第一步肯定是分析，分析到底哪里出问题，然后再想解决方案。sql优化也不例外，所以第一步做的就是分析。

## 通过 show status 命令了解各种sql的执行频率

通过命令 show [global/session] status 命令获取信息。

比较关心的参数如：

- Com_select 执行select操作的次数

- Com_insert 执行insert的次数，批量插入只累加一次

- Com_update 执行update的次数

- Com_delete 执行delete的次数

针对InnoDB存储引擎，累加算法略有不同

- Innodb_rows_read select 查询返回的行数

- Innodb_rows_inserted 执行insert操作插入的行数

- Innodb_rows_ updated 执行update操作更新的行数

- Innodb_rows_delete 执行delete操作删除的行数

还有几个参数也比较重要，事物操作 **Com_commit** **Com_rollback**。
数据库情况：

- Connections 试图连接mysql服务器的次数

- Uptime 服务器工作时间

- Slow_queries 慢查询的次数

## 定位执行效率极低的sql语句

设置mysql慢查询，然后通过日志定位 见 [mysql配置](https://www.icepear.cn/2018/06/20/mysql/mysql-conf/)

## 通过EXPLAIN分析低效SQL的执行计划

通过上面两个步骤查到效率低的sql后，可以通过explain或者desc命令获取select语句信息
信息中的说明

- select_type select的类型，SIMPLE 简单表、PRIMARY 主查询、UNION UNION中的第二个或后面的查询、SUBQUERY 子查询中的第一个select

- table 输出结果的表

- type 表示mysql在表中找到所需行的方式，或者叫访问类型，常见类型
  ALL----index----range----ref----eq_ref----const,system----NULL
  从左到右，性能由差到好

- possible_keys 表示查询时可能使用的索引

- key 表示实际使用的索引

- key_len 使用索引字段的长度

- rows 扫描行的数量

- extra 执行情况的说明和描述

通过explain extended之后再输入show warnings可以看到执行之前优化器做了哪些改动

## 通过 show profile 分析sql

可以先通过select @@profiling查看数据库是否开启profiling

然后通过show profile查看sql执行的时间，以及query_id。然后可以通过show profile for query [query_id]查看执行过程中线程每个状态和消耗时间

# 索引

B-Tree 索引 大部分支持
HASH 索引 只有Memory引擎支持，适用于 key-value查询
R-Tree 索引 MyISAM 特有，用于地理空间数据类型
Full-text 全文索引 MyISAM 特有


