---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/mysql-conf/mysql.png
title: mysql 配置解析
date: 2018-06-20 19:30:02
toc: true
tags: 
- Mysql
categories:
- mysql
---
最近对mysql的配置进行了了解，通过一些文档，所以把配置的一些参数都罗列了一下
<!--more-->

# 下载链接

文件的下载链接 [配置文件](http://icepear.oss-cn-shenzhen.aliyuncs.com/other/mysql-conf/my.cnf)

# docker 运行

```sh
docker run --name mysql \
 -p 3306:3306 \
 -v $PWD/mysql/conf:/etc/mysql/conf.d \
 -v $PWD/mysql/mnt:/home/mnt \
 -v $PWD/mysql/db:/var/lib/mysql \
 -v $PWD/mysql/logs:/home/log \
 -v $PWD/mysql/audit:/home/audit \
 -e MYSQL_ROOT_PASSWORD=123456 \
 -e TZ=Asia/Shanghai \
 --restart=always -d mysql

```

# 配置内容

```conf

# 自用Docker MySql5.7配置文件my.cnf设置 
# 可参考https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html

[mysqld]
pid-file  = /var/run/mysqld/mysqld.pid
socket    = /var/run/mysqld/mysqld.sock
datadir   = /var/lib/mysql

# 支持符号链接，就是可以通过软连接的方式，管理其他目录的数据库，最好不要开启，当一个磁盘或分区空间不够时，
# 可以开启该参数将数据存储到其他的磁盘或分区。使用命令 ln -s /dr1/databases/test /path/to/datadir
# 参考 https://dev.mysql.com/doc/refman/8.0/en/symbolic-links-to-databases.html
symbolic-links=0

# 此变量用于限制数据导入和导出操作的效果，例如由LOAD DATA和SELECT ... INTO OUTFILE语句和LOAD_FILE（）函数执行的操作。
# 这些操作只允许拥有FILE权限的用户使用。NULL值为禁用导入导出
secure-file-priv= NULL

skip-host-cache

# 只能用IP地址检查客户端的登录，不用主机名
skip-name-resolve

#####################基础设置#################

# Mysql服务的唯一编号 每个mysql服务Id需唯一
server-id = 1

# 服务端口号 默认3306
port = 3306

# 用户名 默认root
# user = mysql

# 默认值为本地地址
# bind_address = 127.0.0.1

# 主要用于MyISAM存储引擎,如果多台服务器连接一个数据库则建议注释下面内容
# skip-external-locking

# 设置autocommit=0，则用户将一直处于某个事务中，直到执行一条commit提交或rollback语句才会结束当前事务重新开始一个新的事务。
# set autocommit=0的好处是在频繁开启事务的场景下，减少一次begin的交互。
autocommit = 0

# utf8mb4编码是utf8编码的超集，兼容utf8，并且能存储4字节的表情字符。 
# 采用utf8mb4编码的好处是：存储与获取数据的时候，不用再考虑表情字符的编码与解码问题。
character_set_server=utf8mb4

# 数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server = utf8mb4_general_ci

# 设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'

# 事务隔离级别，默认为可重复读，mysql默认可重复读级别（此级别下可能参数很多间隙锁，影响性能）
transaction_isolation = READ-COMMITTED

# 是否区分大小写，0区分，1不区分，2，表名存储以规定格式存，但比较还是用小写
lower_case_table_names = 1

# 最大连接数
max_connections = 800

# 在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中
# 官方建议back_log = 50 + (max_connections / 5),封顶数为900
back_log = 210

# 最大错误连接数，对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接。
# 如需对该主机进行解禁，执行：FLUSH HOST。
max_connect_errors = 1000

# TIMESTAMP如果没有显示声明NOT NULL，允许NULL值
explicit_defaults_for_timestamp = true

# SQL数据包发送的大小，如果有BLOB对象建议修改成1G
max_allowed_packet = 128M

# MySQL连接闲置超过一定时间后(单位：秒)将会被强行关闭
# MySQL默认的wait_timeout  值为8个小时, interactive_timeout参数需要同时配置才能生效
interactive_timeout = 1800
wait_timeout = 1800


# 临时目录 比如load data infile会用到
tmpdir  = /tmp

# 内部内存临时表的最大值 ，默认64M，设置成128M。
# 比如大数据量的group by ,order by时可能用到临时表，
# 超过了这个值将写入磁盘，系统IO压力增大
tmp_table_size = 134217728
max_heap_table_size = 134217728

# MySQL读入缓冲区的大小，该参数对应的分配内存是每连接独占（以字节为单位）。
# 如果您执行多次连续扫描，则可能需要增加此值，该值默认为131072.此变量的值应为4KB的倍数。
# 如果设置的值不是4KB的倍数，则其值将舍入到4KB的最接近倍数。现在都是InnerDB存储引擎，设置作用不大
read_buffer_size = 8388608

# MySQL的随机读缓冲区大小，将该变量设置为较大的值可以大大提高ORDER BY的性能。
# 但是，这是为每个客户端分配的缓冲区，因此您不应将全局变量设置为较大的值。
# 相反，只需从需要运行大型查询的客户端中更改会话变量。默认值256kb 所以设值2M
read_rnd_buffer_size = 2097152

# MySQL在完成某些join（连接）需求的时候，为了减少参与join的“被驱动表”的读取次数以提高性能，需要使用到join buffer来协助完成join操作
# 当join buffer 太小，MySQL不会将该buffer存入磁盘文件而是先将join buffer中的结果与需求join的表进行操作，
# 然后清空join buffer中的数据，继续将剩余的结果集写入次buffer中，默认值262144，对应的也是每个独占连接
join_buffer_size = 8388608

# MySQL的顺序读缓冲区大小 order by或group by时用到，建议先用4M试一下，对应的也是每个独占连接
sort_buffer_size = 4194304

#一般数据库中没什么大的事务，设成1~2M，默认32kb
binlog_cache_size = 524288




####################日志设置#########################

# 数据库错误日志文件(针对docker)
log_error = /home/log/error.log

# 慢查询sql日志设置
slow_query_log = 1
slow_query_log_file = /home/log/slow.log

# 检查未使用到索引的sql
log_queries_not_using_indexes = 1

# 针对log_queries_not_using_indexes开启后，记录慢sql的频次、每分钟记录的条数
log_throttle_queries_not_using_indexes = 5

# 作为从库时生效,从库复制中如何有慢sql也将被记录
log_slow_slave_statements = 1

# 慢查询执行的秒数，必须达到此值可被记录
long_query_time = 8

# 检索的行数必须达到此值才可被记为慢查询
min_examined_row_limit = 1000

# mysql binlog日志文件保存的过期时间，过期后自动删除
expire_logs_days = 7




####################主从复制设置#########################
# 参考https://dev.mysql.com/doc/refman/5.7/en/replication-options.html
# 将master.info和relay.info保存在表中
master_info_repository = TABLE
relay_log_info_repository = TABLE

# 开启mysql binlog二进制日志功能
log_bin = /var/lib/mysql/mysql-bin

# 当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。设置为零是让系统自行决定
# 为1时安全性高，只丢失一次事物数据，消耗同样也大
sync_binlog = 5

# 从服务器的更新是否写入二进制日志，从库必须开启了二进制日志功能
log_slave_updates = 1

# 开启全局事务ID，GTID能够保证让一个从服务器到其他的从服务器那里实现数据复制而且能够实现数据整合的
gtid_mode = on

# 开启gtid，必须主从全开
enforce_gtid_consistency = 1

# 开启简单gtid，开启此项会提升mysql执行恢复的性能
binlog_gtid_simple_recovery = 1

# 从服务器的更新是否写入二进制日志
log_slave_updates = 1

# 三种模式 STATEMENT（有可能主从数据不一致，日质量小）版本小于5.7.6的默认值；ROW（产生大量二进制日志）版本大于5.7.7的默认值、MIXED
binlog_format = row

# 对于binlog_format = ROW模式时，减少记录日志的内容，只记录受影响的列
# full（记录所有列）默认
# minimal（仅记录更改的列和标识行所需的列）
# noblob（记录所有列，除了不需要的blob和文本列）
binlog_row_image = minimal

# relay-log日志记录的是从服务器I/O线程将主服务器的二进制日志读取过来记录到从服务器本地文件，然后SQL线程会读取relay-log日志的内容并应用到从服务器
relay_log = /home/log/relay.log

# 作为从库时生效,中继日志relay-log可以自我修复
relay_log_recovery = 1

# 作为从库时生效,主从复制时忽略的错误
slave_skip_errors = ddl_exist_errors




#######################Innodb设置########################
# 参考 https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html

# 这个参数在一开始初始化时就要加入my.cnf里，如果已经创建了表，再修改，启动MySQL会报错。最好为8K
#innodb_page_size = 16384
innodb_page_size = 8192

# 数据缓冲区buffer pool大小，建议使用物理内存的 75%
innodb_buffer_pool_size = 8G

# 缓冲池实例数量，当buffer_pool的值较大的时候为1，较小的设置为8
innodb_buffer_pool_instances = 8

# 运行时load缓冲池，快速预热缓冲池，将buffer pool的内容（文件页的索引）dump到文件中，然后快速load到buffer pool中。
# 避免了数据库的预热过程，提高了应用访问的性能 默认值(>= 5.7.7)	开启;默认值(<= 5.7.6)关闭。
innodb_buffer_pool_load_at_startup = 1

# 运行时dump缓冲池 默认值(>= 5.7.7)	开启;默认值(<= 5.7.6)关闭。
innodb_buffer_pool_dump_at_shutdown = 1

# 在innodb中处理用户查询后，其结果在内存空间的缓冲池已经发生变化，但是还未记录到磁盘。这种页面称为脏页，将脏页记录到磁盘的过程称为刷脏
innodb_lru_scan_depth = 2000

# 参数设置innodb后台任务每秒执行的I / O操作数量的上限，默认值200，好的硬盘可适当调高
innodb_io_capacity = 4000

# 突破innodb_io_capacity后的上限值
innodb_io_capacity_max = 8000

# 事务等待获取资源等待的最长时间，超过这个时间还未分配到资源则会返回应用失败，默认50s
innodb_lock_wait_timeout = 30

# 设置redoLog文件所在目录, redoLog记录事务具体操作内容，默认为data的home目录：
# 系统不会创建该目录，请确保目录已经存在
#innodb_log_group_home_dir = /home/log/redolog
# 设置undoLog文件所在目录, undoLog用于事务回滚操作
#innodb_undo_directory = /home/log/undolog

# 这个参数控制着innodb数据文件及redo log的打开、刷写模式
# fsync InnoDB 使用fsync（）系统调用来刷新数据和日志文件。fsync是默认设置。
# O_DIRECT 不经过系统缓存直接存入磁盘，减少操作系统级别VFS的缓存和Innodb本身的buffer缓存之间的冲突，适用于某些GNU/Linux versions, FreeBSD, and Solaris.
# 根据官方提示还有其他选项好像都不可用，Docker官方镜像不支持O_DIRECT
#innodb_flush_method = O_DIRECT

# 表空间innodb文件格式。支持的文件格式是Antelope和Barracuda
# Antelope是原始的innodb文件格式，它支持冗余和紧凑的行格式。
# Barracuda是较新的文件格式，它支持压缩和动态行格式。默认值(>= 5.7.7)	Barracuda 默认值(<= 5.7.6)	Antelope
# 但是这两个选项都被弃用了，在后面版本会删除
# innodb_file_format = Barracuda
# innodb_file_format_max = Barracuda

# 当启用innodb_strict_mode时，innodb在某些情况下返回错误而不是警告。默认值(>= 5.7.7) 开启；默认值(<= 5.7.6) 关闭
#innodb_strict_mode = 1

# 当启用innodb_file_per_table（默认值 开启）时，innodb将每个新创建的表的数据和索引存储在单独的.ibd文件中，而不是系统表空间中。
# 当这些表被删除或截断时，这些表的存储将被回收。缺点会导致单个表文件过大
innodb_file_per_table = 1

# undo日志回滚段 默认为128
innodb_undo_logs = 128

# 传统机械硬盘建议使用，而对于固态硬盘可以关闭
#innodb_flush_neighbors = 1

# 日志文件大小1G，默认48M 
innodb_log_file_size = 1073741842
innodb_log_buffer_size = 16777216

# 专用于innodb清除操作的后台线程的数量。最小值为1表示清除操作总是由后台线程执行，而不是主线程的一部分。
# 在一个或多个后台线程中运行清除操作有助于减少innodb内部的争用，提高可伸缩性。将值增加到大于1会创建许多单独的清除线程，这可以提高在多个表上执行dml操作的系统的效率。
# (>= 5.7.8)默认值是4，(<= 5.7.7)默认值是1，最大值是32。
innodb_purge_threads = 4

# 改为ON时，允许单列索引最大达到3072。否则最大为767，也已经弃用
#innodb_large_prefix = 1

# innodb中同时保持操作系统线程的数量小于或等于此变量（innodb使用操作系统线程处理用户事务）给出的限制。
# 一旦线程数量达到此限制，额外的线程就会进入“先进先出”（fifo）队列中的等待状态以供执行。等待锁的线程数不计入并发执行线程数。
# 值范围0-1000,0表示无限并发。如果工作负载的并发用户线程数小于64，建议用默认值0，否则需要视情况进行调整
innodb_thread_concurrency = 0

# 开启后会将所有的死锁记录到error_log中
innodb_print_all_deadlocks = 1
# 用于在创建innodb索引期间对数据进行排序的排序缓冲区的大小，仅用于索引创建期间的合并排序，而不用于以后的索引维护操作。
innodb_sort_buffer_size = 8338608 
```