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
| INSERT(str,x,y,instr) | 将字符串str从第x位开始,y个字符串长度的子串替换成instr |
| REPLACE(str,a,b) | 用字符串b替换字符串str中所有出现的字符串a |
| SUBSTRING(str,x,y) | 截取字符串str从x位置到y位置的子串 |
| LEFT(str,x) | 返回字符串str最左边的x个字符 |
| RIGHT(str,x) | 返回字符串str最右边的x个字符 |
| LOWER(str) | 将字符串str中所有的字符变为小写 |
| UPPER(str) | 将字符串str中所有的字符变为大写 |
| LTRIM(str) | 去掉字符串左侧的空格 |
| RTRIM(str) | 去掉字符串行尾的空格 |
| TRIM(str) | 去掉字符串行头和行尾的空格 |
| LPAD(str,n,pad) | 用字符串pad对str最左边进行填充，直到长度为n个字符长度 |
| RPAD(str,n,pad) | 用字符串pad对str最右边进行填充，直到长度为n个字符长度 |
| REPEAT(str,x) | 返回str重复x次的结果 |
| STRCMP(s1,s2) | 比较字符串s1和s2 |

## 字符串的拼接、替换、裁剪

这些函数是最常见的，务必牢记

1. **CONCAT(s1,s2,...,sn)** 拼接字符串

    下面例子将aa，bb，cc三个字符串拼接，另外任何字符串与NULL连接的结果都将是NULL

    ```sql
    mysql> select concat('aa','bb','cc'),concat('aa',null);
    +-------------------------+----------------------+
    | concat('aa','bb','cc')  | concat('aa',null)    |
    +-------------------------+----------------------+
    | aabbcc                  | NULL                 |
    +-------------------------+----------------------+
    1 row in set(0.00 sec)
    ```
2. **INSERT、REPLACE** 两个函数都是替换字符串
    下面例子将abcd字符串替换成abcc，通过两个方法
    ```sql
    mysql> select insert('abcd',3,2,'cc'),replace('abcd','d','c');
    +-------------------------+-------------------------+
    | insert('abcd',3,2,'cc') | replace('abcd','d','c') |
    +-------------------------+-------------------------+
    | abcc                    | abcc                    |
    +-------------------------+-------------------------+
    1 row in set (0.00 sec)
    ```
3. **SUBSTRING、LEFT、RIGHT** 三个函数都可以对字符串进行裁剪
    substring相对灵活很多，left和right函数比较局限，只能截取左边或右边的子串，看三个例子
    ```sql
    mysql> select substring('abcdefg',2,5), left('abcdefg',4),right('abcdefg',4);
    +--------------------------+-------------------+--------------------+
    | substring('abcdefg',2,5) | left('abcdefg',4) | right('abcdefg',4) |
    +--------------------------+-------------------+--------------------+
    | bcdef                    | abcd              | defg               |
    +--------------------------+-------------------+--------------------+
    1 row in set (0.00 sec)
    ```

## 字符串的字符处理，对比

1. **LOWER、UPPER、LTRIM、RTRIM、TRIM** 等函数可以对字符串进行处理
    LOWER、UPPER 用于大小写转换，LTRIM、RTRIM、TRIM用于对字符串空格处理。看例子
    ```sql
    mysql> select upper('abcd'),lower('ABCD'),ltrim(' $abcd'),rtrim('abcd$ '),trim(' $abcd$ ');
    +---------------+---------------+-----------------+-----------------+------------------+
    | upper('abcd') | lower('ABCD') | ltrim(' $abcd') | rtrim('abcd$ ') | trim(' $abcd$ ') |
    +---------------+---------------+-----------------+-----------------+------------------+
    | ABCD          | abcd          | $abcd           | abcd$           | $abcd$           |
    +---------------+---------------+-----------------+-----------------+------------------+
    1 row in set (0.00 sec)
    ```
2. **LPAD、RPAD** 填充字符串函数
    左右填充字符串，直到长度为n，看例子
    ```sql
    mysql> select lpad('defg',8,'abcd'),rpad('abcd',8,'defg');
    +-----------------------+-----------------------+
    | lpad('defg',8,'abcd') | rpad('abcd',8,'defg') |
    +-----------------------+-----------------------+
    | abcddefg              | abcddefg              |
    +-----------------------+-----------------------+
    1 row in set (0.00 sec)
    ```
3. **REPEAT、STRCMP**两个相对特殊一点的函数
    见例子，strcmp比较的是字符串的ASCII码大小，相等返回0，小返回-1，大返回1
    ```sql
    mysql> select repeat('abc',3),strcmp('a','a'),strcmp('aa','ab'),strcmp('b','a');
    +-----------------+-----------------+-------------------+-----------------+
    | repeat('abc',3) | strcmp('a','a') | strcmp('aa','ab') | strcmp('b','a') |
    +-----------------+-----------------+-------------------+-----------------+
    | abcabcabc       |               0 |                -1 |               1 |
    +-----------------+-----------------+-------------------+-----------------+
    1 row in set (0.00 sec)
    ```

# 数值函数

| 函数 | 功能 |
| - | - |
| ABX(x) | 返回x的绝对值 |
| CEIL(x) | 返回大于x的最小整数值 |
| FLOOR(x) | 返回小于x的最小整数值 |
| MOD(x,y) | 返回x/y的模 |
| RAND() | 返回0~1内的随机数 |
| ROUND(x,y) | 返回参数x的四舍五入的有y位小数的值 |
| TRUNCATE(x,y) | 返回数字x截断为y位小数的结果 |

1. **ABX(x)** 绝对值
    ```sql
    mysql> select abs(-2),abs(2),abs(-5);
    +---------+--------+---------+
    | abs(-2) | abs(2) | abs(-5) |
    +---------+--------+---------+
    |       2 |      2 |       5 |
    +---------+--------+---------+
    1 row in set (0.00 sec)
    ```
2. **CEIL(x)、FLOOR(x)** 最小整数值
    ```sql
    mysql> select ceil(2.5),floor(2.5);
    +-----------+------------+
    | ceil(2.5) | floor(2.5) |
    +-----------+------------+
    |         3 |          2 |
    +-----------+------------+
    1 row in set (0.01 sec)
    ```
3. **MOD(x,y)** 取模
    ```sql
    mysql> select mod(9,2),mod(17,5);
    +----------+-----------+
    | mod(9,2) | mod(17,5) |
    +----------+-----------+
    |        1 |         2 |
    +----------+-----------+
    1 row in set (0.00 sec)
    ```
4. **RAND()** 0~1随机数
    可以通过组合方式，产生0~9的随机数，或者更大的随机数
    ```sql
    mysql> select rand(),floor(rand()*10);
    +--------------------+------------------+
    | rand()             | floor(rand()*10) |
    +--------------------+------------------+
    | 0.9055393793671759 |                7 |
    +--------------------+------------------+
    1 row in set (0.00 sec)
    ```
5. **ROUND(x,y)、TRUNCATE(x,y)** 四舍五入和截取
    ```sql
    mysql> select round(4.32354,3),round(4.32354,2),truncate(4.32354,3);
    +------------------+------------------+---------------------+
    | round(4.32354,3) | round(4.32354,2) | truncate(4.32354,3) |
    +------------------+------------------+---------------------+
    |            4.324 |             4.32 |               4.323 |
    +------------------+------------------+---------------------+
    1 row in set (0.00 sec)
    ```

# 日期函数

| 函数 | 功能 |
| - | - |
| CURDATE() | 当前日期 |
| CURTIME() | 当前时间 |
| NOW() | 当前日期和时间 |
| UNIX_TIMESTAMP(date) | 日期date的UNIX时间戳 |
| FROM_UNIXTIME(unixtime) | UNIX时间戳的日期值 |
| WEEK(date) | 日期date为一年中的第几周 |
| YEAR(date) | 日期date的年份 |
| HOUR(time) | time的小时值 |
| MINUTE(time) | time的分钟值 |
| MOONTHNAME(date) | date的月份名 |
| DATE_FORMAT(date,fmt) | 按字符串fmt格式化日期date的值 |
| DATE_ADD(date,INTERVAL exper type) | 一个日期或时间值加上一个时间间隔的时间值 |
| DATEDIEF(expr,expr2) | 其实时间expr和结束时间expr2之间的天数 |

**DATE_ADD(date,INTERVAL exper type)**重点说明一下，
其中 INTERVAL 是间隔类型关键字，expr 是表达式， type是类型，
类型表如下

| type | 解释 |
| - | - |
| MICROSECOND | 间隔单位：毫秒 |
| SECOND | 间隔单位：秒 |
| MINUTE | 间隔单位：分钟 |
| HOUR | 间隔单位：小时 |
| DAY | 间隔单位：天 |
| WEEK | 间隔单位：星期 |
| MONTH | 间隔单位：月 |
| QUARTER | 间隔单位：季度 |
| YEAR | 间隔单位：年 |
| SECOND_MICROSECOND | 复合型，间隔单位：秒、毫秒，expr可以用两个值来分别指定秒和毫秒 |
| MINUTE_MICROSECOND | 复合型，间隔单位：分、毫秒 |
| MINUTE_SECOND | 复合型，间隔单位：分、秒 |
| HOUR_MICROSECOND | 复合型，间隔单位：小时、毫秒 |
| HOUR_SECOND | 复合型，间隔单位：小时、秒 |
| HOUR_MINUTE | 复合型，间隔单位：小时分 |
| DAY_MICROSECOND | 复合型，间隔单位：天、毫秒 |
| DAY_SECOND | 复合型，间隔单位：天、秒 |
| DAY_MINUTE | 复合型，间隔单位：天、分 |
| DAY_HOUR | 复合型，间隔单位：天、小时 |
| YEAR_MONTH | 复合型，间隔单位：年、月 |

例如
    ```sql
    mysql> select date_add('2013-01-18', interval '1 2' YEAR_MONTH);
    +-----------------------------------------------------+
    | date_add('2013-01-18', interval '1 2' YEAR_MONTH) |
    +-----------------------------------------------------+
    | 2014-03-18                                          |
    +-----------------------------------------------------+

    mysql> select date_add('2013-01-18', interval '1-2' YEAR_MONTH);
    +----------------------------------------------------+
    | date_add('2013-01-18', interval '1-2' YEAR_MONTH) |
    +----------------------------------------------------+
    | 2014-03-18                                         |
    +----------------------------------------------------+
    ```

# 其他函数

| 函数 | 功能 |
| - | - |
| DATEBASE() | 数据库名 |
| VERSION() | 版本 |
| USER() | 用户名 |
| MD5(str) | str的MD5加密值 |