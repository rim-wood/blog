---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/mysql-conf/mysql.png
title: mysql 学习笔记（一）
date: 2018-07-13 19:30:02
toc: true
tags: 
- Mysql
categories:
- mysql
---
mysql 常用函数，使用sql必然避免不了使用函数，函数往往能让你的工作事半功倍，常用的函数包括：字符串的处理，数值的运算，日期函数，流程控制。
<!--more-->

# 字符串函数

| 函数 | 功能 |
| - | - |
| CONCAT(s1,s2,...,sn) | 连接 s1，s2 ...sn 为一个字符串 |
| INSERT(str,x,y,instr) | 将字符串str从第x位开始到y位的子串替换成instr |
| REPLACE(str,a,b) | 用字符串b替换字符串str中所有出现的字符串a |
| SUBSTRING(str,x,y) | 截取字符串str从x位置到y位置的子串 |
| LOWER(str) | 将字符串str中所有的字符变为小写 |
| UPPER(str) | 将字符串str中所有的字符变为大写 |
| LEFT(str,x) | 返回字符串str最左边的x个字符 |
| RIGHT(str,x) | 返回字符串str最右边的x个字符 |
| LPAD(str,n,pad) | 用字符串pad对str最左边进行填充，直到长度为n个字符长度 |
| RPAD(str,n,pad) | 用字符串pad对str最右边进行填充，直到长度为n个字符长度 |
| LTRIM(str) | 去掉字符串左侧的空格 |
| RTRIM(str) | 去掉字符串行尾的空格 |
| TRIM(str) | 去掉字符串行头和行尾的空格 |
| REPEAT(str,x) | 返回str重复x次的结果 |
| STRCMP(s1,s2) | 比较字符串s1和s2 |

## 字符串的拼接、替换、裁剪

这些函数是最常见的，务必牢记

1. **CONCAT(s1,s2,...,sn)** 拼接字符串

    下面例子将aa，bb，cc三个字符串拼接，另外任何字符串与NULL连接的结果都将是NULL

    ```sql
    select concat('aa','bb','cc'),concat('aa',null)

    +-------------------------+----------------------+
    | concat('aa','bb','cc')  | concat('aa',null)    |
    +-------------------------+----------------------+
    | aabbcc                  | NULL                 |
    +-------------------------+----------------------+
    1 row in set(0.05 sec)
    ```
2. **INSERT、REPLACE** 两个函数都是替换字符串