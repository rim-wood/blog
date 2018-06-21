---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/redis-conf/redis.png
title: redis 配置解析
date: 2018-06-21 19:30:02
toc: true
tags: 
- Redis
categories:
- other
---
拿着原始的redis.conf进行一一解析，应该算是最全的解析版本了
<!--more-->

# 下载链接
文件的下载链接 [配置文件](http://icepear.oss-cn-shenzhen.aliyuncs.com/other/redis-conf/redis.conf)

# 配置内容
看不完，就先下载下来慢慢看

```
# Redis 配置样例
#
# 为了读取配置文件，Redis必须以文件路径作为第一个参数来启动：
#
# ./redis-server /path/to/redis.conf

# 当需要内存大小时，可以使用1k 5GB 4M等通常的形式指定它，如下所示：
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# 单位不区分大小写，所以1GB 1Gb 1gB都是一样的。

################################## INCLUDES配置文件 ###################################

# 在这里包含一个或多个其他配置文件。 如果您有一个标准模板可用于所有Redis服务器，
# 但也需要自定义几个每服务器设置，这非常有用。 包含文件可以包含其他文件，所以明智地使用它。
# “include”不会被来自admin或Redis Sentinel的命令“CONFIG REWRITE”重写。 
# 由于Redis总是使用最后一条处理过的行作为配置指令的值，因此最好将include包含在此文件的开头，以避免在运行时覆盖配置更改。
#
# include /path/to/local.conf
# include /path/to/other.conf

################################## MODULES模块加载 #####################################

# 启动时加载模块。 如果服务器无法加载模块，则会中止。 可以使用多个loadmodule指令
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

################################## NETWORK网络配置 #####################################

# 默认情况下，如果没有指定“绑定”配置指令，则Redis监听来自服务器上可用的所有网络接口的连接。 
# 可以使用“bind”配置指令监听一个或多个选定的接口，然后使用一个或多个IP地址。
#
# 例子:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ 
# 如果运行Redis的计算机直接暴露在互联网上，绑定到所有接口是危险的，并会将实例暴露给互联网上的每个人。 
# 因此，默认情况下，我们取消注释以下绑定指令，这将强制Redis仅侦听IPv4回溯接口地址
#（这意味着Redis将只能接受与运行在同一台计算机上的客户端的连接）。
#
# 如果您确定您希望您的实例能够聆听所有接口，只需像下面这样即可。
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1

# 保护模式是一层安全保护措施，以避免在互联网上保持打开的Redis实例被访问和利用。
#
# 当保护模式打开时，如果：
#
# 1) 服务器没有明确地使用"bind"绑定到一组地址
# 2) 没有配置密码。
#
# 服务器只接受来自IPv4和IPv6环回地址127.0.0.1和:: 1以及来自Unix域套接字的客户端的连接。
#
# 默认情况下启用保护模式。
protected-mode yes

# 接受指定端口上的连接，默认值为6379如果指定端口0，则Redis不会侦听TCP套接字。
port 6379

# TCP listen() backlog.
#
# 在高请求每秒的环境中，您需要高的backlog以避免缓慢的客户端连接问题。 
# 请注意，Linux内核会将其自动截断为/ proc / sys / net / core / somaxconn的值，
# 因此请确保同时提高somaxconn和tcp_max_syn_backlog的值以获得所需的效果。
tcp-backlog 511

# Unix socket.
#
# 指定将用于侦听传入连接的Unix socket的路径。 没有默认设置，因此Redis在未指定时不会在unix socket上侦听。
#
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# 客户端空闲N秒后关闭连接（0禁用）
timeout 0

# TCP keepalive. 心跳检测
#
# 如果非零，则在没有通信的情况下使用SO_KEEPALIVE向客户端发送TCP ACK。 
# 这是有用的，原因有两个：
#
# 1) 检测死掉的资源共享者
# 2) 从中间网络设备的角度来看，连接是活着的。
#
# 在Linux上，指定的值（以秒为单位）是用于发送ACK的时间段。
# 请注意，要关闭连接，需要两倍的时间。
# 在其他内核上，时间取决于内核配置。
#
# 这个选项的合理值是300秒，这是新的
# Redis默认从Redis 3.2.1开始。
tcp-keepalive 300

################################# 一般配置 #####################################

# 默认情况下，Redis不会作为守护进程运行。 如果您需要，请使用"yes"。
# 请注意，当守护进程时，Redis将在/var/run/redis.pid中写入一个pid文件。
daemonize no

# 如果您从upstart或systemd运行Redis，则Redis可以与您的监督树进行交互。选项:
#   supervised no      - 没有监督互动
#   supervised upstart - 通过将Redis置于SIGSTOP模式来发信号
#   supervised systemd - 通过将READY = 1写入$ NOTIFY_SOCKET来系统化信号
#   supervised auto    - 检测 upstart or systemd 方法基于UPSTART_JOB or NOTIFY_SOCKET 环境变量
# 注意：这些监督方法只会发出“过程已准备就绪”的信号。 他们不能将连续的活跃回馈给你的supervisor。
supervised no

# 如果指定了pid文件，则Redis将其写入启动时指定的位置，并在退出时将其删除。
# 当服务器运行非守护进程时，如果配置中没有指定pid文件，则不会创建pid文件。 
# 当服务器被守护进程时，即使未指定，也会使用pid文件，默认为“/var/run/redis.pid”。
# 如果Redis不能创建它，创建一个pid文件是最好的选择
pidfile /var/run/redis_6379.pid

# 服务器日志级别:
# debug (对开发/测试有用)
# verbose (比debug级别精简)
# notice (适度详细，用在生产中)
# warning (只记录非常重要/关键的消息)
loglevel notice

# 指定日志文件名称 空字符串可用于强制Redis log标准输出。请注意，如果您使用标准输出进行日志记录但守护进程，则日志将被发送到/dev/null
logfile ""

# 要启用logging到系统 logger，只需将'syslog-enabled'设置为yes，并可以选择更新其他syslog参数以满足您的需求。
# syslog-enabled no

# 指定系统日志标识。
# syslog-ident redis

# 指定系统日志功能。 必须是USER或local0至local7。
# syslog-facility local0

# 设置数据库的数量，默认值是16。
# 默认数据库是DB 0，您可以使用SELECT <dbid>在每个连接的基础上切换不同的数据库，
databases 16

# 默认情况下，Redis只有在log是标准输出并且标准输出是TTY时才显示ASCII logo。 基本上这意味着通常只有在交互式会话中才会显示logo。
always-show-logo yes

################################ SNAPSHOTTING快照配置  ################################
#
# 保存DB到磁盘:
# 
#   save <seconds> <changes>
#
#   如果发生针对数据库的给定秒数和给定数量的写操作，将保存数据库。
#
#   例如:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   注意: 您可以通过注释掉所有“保存”行来完全禁用保存。也可以用 save ""

save 900 1
save 300 10
save 60 10000

# 默认情况下，如果启用了RDB快照（至少有一个保存点）并且最新的后台保存失败，Redis将停止接受写入。
# 可以让用户意识到数据不能正确的保存在磁盘上，否则没人会去关注这一错误的发生
#
# 如果后台保存过程将再次开始工作，则Redis将自动再次允许写入。
#
# 但是，如果您已设置了对Redis服务器和持久性的适当监控，则可能需要禁用此功能，以便即使磁盘，权限等问题仍然存在，Redis仍将照常继续工作。
stop-writes-on-bgsave-error yes

# 在转储.rdb数据库时使用LZF压缩字符串对象？对于默认设置为'是'，因为它几乎总是可以的。
# 如果您想要在保存子节点中保存一些CPU，请将其设置为“否”，但如果您具有可压缩值或密钥，则数据集可能会更大。
rdbcompression yes

# 从版本5的RDB开始，CRC64校验和被放置在文件的末尾。
# 这使得该格式更能抵抗腐败，但在保存和加载RDB文件时，性能会受到影响（大约为10％），因此您可以禁用它以获得最佳性能。
rdbchecksum yes

# 转储数据库的文件名
dbfilename dump.rdb

# 存储的文件目录
dir ./

################################# 主从复制 #################################

# 主从复制。 使用slaveof将Redis实例作为另一个Redis服务器的副本
#
# 1) Redis复制是异步的，但是如果它看起来没有连接至少给定数量的slave，您可以配置master来停止接受写入。
# 2) 如果复制链接在相对较短的时间内丢失，则Redis从节点能够与主节点执行部分重新同步。 您可能需要根据您的需要配置合理的值来配置复制积压大小
# 3) 复制是自动的，不需要用户干预。在网络分区后slave自动尝试重新连接到master并与它们重新同步
#
# slaveof <masterip> <masterport>

# 如果主站受密码保护（使用下面的“requirepass”配置指令），可以在开始复制同步过程之前告诉从站进行认证，否则主站将拒绝从站请求。
#
# masterauth <master-password>

# 当slave失去与master的连接时，或者复制仍在进行时，slave可以采取两种不同的方式：
#
# 1) 如果slave-serve-stale-data设置为'yes'（缺省值），则从设备仍然会回应客户端请求，可能会使用过时数据，或者如果这是第一次同步，数据集可能只是空的。
#
# 2) 如果slave-serve-stale-data设置为'no'，则从设备将回复一个错误“SYNC with master in progress”，除了INFO和SLAVEOF之外的所有类型的命令。
#
slave-serve-stale-data yes

# 您可以配置一个从设备实例来接受写入与否。 针对从属实例写入数据可能对于存储一些短暂数据非常有用
#（因为写入从属服务器上的数据在与主服务器重新同步后很容易被删除），但是如果客户端因错误配置而写入数据，也可能会导致问题。
#
# 从Redis 2.6后 默认情况是只读的。
slave-read-only yes

# 复制SYNC策略: disk or socket.
#
# -------------------------------------------------------
# WARNING: DISKLESS 是目前的测试参数
# -------------------------------------------------------
#
# 新的slaves和重新连接的因为接收差异无法继续复制进程,需要做所谓的 "full synchronization".RDB文件从maste 传输到slaves。
# 传输可以以两种不同的方式发生：
#
# 1) Disk-backed: Redis master创建一个将RDB文件写入磁盘的新进程。 之后，文件由父进程传递到从服务器。
# 2) Diskless: Redis master创建一个新的进程，直接将RDB文件写入从套接字，而不用接触磁盘。
#
# 使用 disk-backed 复制, 当生成RDB文件时，只要生成RDB文件的当前子节点完成其工作，就可以将更多的从属节点排队并与RDB文件一起提供服务。
# 使用diskless复制，一旦传输开始，到达的新从站将排队，并且当当前端口终止时将开始新的传输。
# 使用diskless复制，在开始传输之前，主设备等待可配置的时间量（以秒为单位），可以等待多个从设备到达后并行的传输。
#
# 对于慢速磁盘和快速（大带宽）网络 diskless 复制效果更好
repl-diskless-sync no

# 启用 diskless 时, 服务器等待一段时间后才会通过套接字向从站传送RDB文件，这个等待时间是可配置的。 
# 这一点很重要，因为一旦传送开始，就不可能再为一个新到达的从站服务。从站则要排队等待下一次RDB传送。因此服务器等待一段时间以期更多的从站到达。
# 延迟时间以秒为单位，默认为5秒。要关掉这一功能，只需将它设置为0秒，传送会立即启动
repl-diskless-sync-delay 5

# 从站以一个预先设置好的时间间隔向服务器发送PING。这个时间间隔可以通过repl_ping_slave_period选项改变。默认值是10秒。
# repl-ping-slave-period 10

# 该选项为以下内容设置备份的超时时间：
#
# 1) slaves角度，同步批量传输的I/O.
# 2) slave角度，master超时(数据, ping).
# 3) master角度，slave超时 (REPLCONF ACK pings).
#
# 确认这个值比定义的repl-ping-slave-period要大，否则每次主站和从站之间通信低速时都会被检测为超时。
#
# repl-timeout 60

# 同步之后是否禁用slave上的TCP_NODELAY？
#
# 假如设置成yes，则redis会合并小的TCP包从而节省带宽，但会增加同步延迟（40ms），造成master与slave数据不一致
# 假如设置成no，则redis master会立即发送同步数据，没有延迟
#
# 默认情况下，我们针对低延迟进行了优化，但在非常高并发量条件下，或者当主设备和从设备距离很远时，将此设置为“yes”可能是一个好主意。
repl-disable-tcp-nodelay no

# 设置备份的工作储备大小。工作储备是一个缓冲区，当从站断开一段时间的情况时，
# 它替从站接收存储数据，因此当从站重连时，通常不需要完全备份，只需要一个部分同步就可以，即把从站断开时错过的一部分数据接收。 
# 工作储备越大，从站可以断开并稍后执行部分同步的断开时间就越长。 
# 只要有一个从站连接，就会立刻分配一个工作储备。
# repl-backlog-size 1mb

# 主站有一段时间没有与从站连接，对应的工作储备就会自动释放。接下来这个选项用于配置释放前等待的秒数，秒数从断开的那一刻开始计算。 
#
# 请注意，从站永远不会释放积压超时，因为它们可能在稍后被提升为主站，并且应该能够正确地与从站“部分重新同步”：因此它们应该始终积累积压。
#
# 值为0表示不释放。
#
# repl-backlog-ttl 3600

# 从站优先级是可以从redis的INFO命令输出中查到的一个整数。当主站不能正常工作时，redis sentinel使用它来选择一个从站并将它提升为主站。 
# 低优先级的从站被认为更适合于提升，因此如果有三个从站优先级分别是10， 100， 25，sentinel会选择优先级为10的从站，因为它的优先级最低。 
# 然而优先级值为0的从站不能执行主站的角色，因此优先级为0的从站永远不会被redis sentinel提升。 
# 默认优先级是100
slave-priority 100

# 主站可以停止接受写请求，当与它连接的从站少于N个，滞后少于M秒。N个从站必须是在线状态。 
# 延迟的秒数必须<=所定义的值，延迟秒数是从最后一次收到的来自从站的ping开始计算。ping通常是每秒一次。 
# 这一选项并不保证N个备份都会接受写请求，但是会限制在指定秒数内由于从站数量不够导致的写操作丢失的情况。 
# 设置某一个为0表示禁用这一功能。 
# 默认情况下default min-slaves-to-write设置为0（禁用）而min-slaves-max-lag设置为10。
# 如果想要至少3个从站且延迟少于10秒，这样写：
#
# min-slaves-to-write 3
# min-slaves-max-lag 10

# Redis master能够以不同的方式列出所连接slave的地址和端口。 
# 例如，“INFO replication”部分提供此信息，除了其他工具之外，Redis Sentinel还使用该信息来发现slave实例。
# 此信息可用的另一个地方在masterser的“ROLE”命令的输出中。
# 通常由slave报告的列出的IP和地址,通过以下方式获得：
#  IP：通过检查slave与master连接使用的套接字的对等体地址自动检测地址。
#  端口：端口在复制握手期间由slavet通信，并且通常是slave正在使用列出连接的端口。
# 然而，当使用端口转发或网络地址转换（NAT）时，slave实际上可以通过(不同的IP和端口对)来到达。 slave可以使用以下两个选项，以便向master报告一组特定的IP和端口，
# 以便INFO和ROLE将报告这些值。
# 如果你需要仅覆盖端口或IP地址，则没必要使用这两个选项。
# 注意：在Docker默认网络模式下，使用-p参数做端口映射，就需要配置一下从服务器redis.conf中的slave-announce-ip 和 slave-announce-port，对应外网的IP和外网端口。
# Sentinel的配置文件sentinel.conf 也需要配置sentinel announce-ip 和 sentinel announce-port ，对应外网的IP和外网端口。
# 当然，如果Docker配置成host网络模式，就不需要配置了，但建议最好不要用host模式
# slave-announce-ip 5.5.5.5
# slave-announce-port 1234

################################## SECURITY安全性 ###################################

# 密码设置
#
requirepass icepear123456

# 作为服务端的redis-server，我们常常需要禁用以上命令来使服务器更加安全。
#
# 例如:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# 在上面的例子中，CONFIG命令被重命名为一个不可猜测的名字。
# 也可以通过将其重命名为空字符串来完全禁用它（或任何其他命令），如下例所示：
#
# rename-command CONFIG ""

################################### CLIENTS客户端 ####################################

# 设置能连上redis的最大客户端连接数量。默认是10000个客户端连接。由于redis不区分连接是客户端连接还是内部打开文件或者和slave连接等，
# 所以maxclients最小建议设置到32。如果超过了maxclients，redis会给新的连接发送’max number of clients reached’，并关闭连接。
#
# maxclients 10000

############################## MEMORY MANAGEMENT 内存管理 ################################

# 将内存使用限制设置为指定的字节数。 达到内存限制时，Redis将尝试根据所选驱逐策略删除密钥（请参阅maxmemory-policy）。
#
# 如果Redis无法根据策略删除密钥，或者如果策略设置为“noviction”吗，Redis将开始以错误的形式回复将使用更多内存的命令，如SET，LPUSH等，并将继续回复GET等只读命令。
#
# 当使用Redis作为LRU或LFU缓存时，或者为实例设置硬内存限制（使用'noviction'策略）时，此选项通常很有用。
#
# 警告：如果您的从站连接到启用了maxmemory的实例，则从已用存储器计数中减去用于馈送从站的输出缓冲区的大小。
# 因此网络问题/ resyncs不会触发导致密钥被清除的一个循环，并且从属的输出缓冲区充满了被删除的DEL键以及触发删除更多密钥的等等，直到数据库完全清空。
#
# 简而言之，如果你有slave，建议你为maxmemory设置一个下限，以便系统上有一些空闲的RAM用于从属输出缓冲区（但如果策略是'noviction'则不需要）。
# 如果不设置maxmemory或者设置为0，64位系统不限制内存，32位系统最多使用3GB内存。
# maxmemory <bytes>

# 内存删除策略: 当达到maxmemory时，Redis将如何选择要删除的内容。 您可以选择八种行为：
# 从Redis4.0开始，一个新的叫做最少频率使用驱逐模型是可用的。此模型在某些场景下可能会工作的更好（提供一个更好的命中/失误比率），
# 因为使用LFU Redis将试图追踪所访问目标的频率，以便极少使用的驱逐而经常使用的则有更高机会保留在内存中。
#
# volatile-lru -> 在设置了过期时间的键空间中，优先移除最近未使用的key。
# allkeys-lru -> 优先移除最近未使用的key。
# volatile-lfu ->  在设置了过期时间的键空间中，移除使用频次最少的key
# allkeys-lfu -> 移除使用频次最少的key
# volatile-random ->  在设置了过期时间的键空间中，随机移除
# allkeys-random -> 随机删除 
# volatile-ttl -> 在设置了过期时间的键空间中，具有更早过期时间的key优先移除。
# noeviction -> 当内存使用达到阈值的时候，所有引起申请内存的命令会报错
#
# LRU 最近未使用
# LFU 使用频次最少
#
# LRU，LFU和volatile-ttl都是使用近似随机算法实现的。(没有绝对的随机算法，哈哈)
#
# 注意: 默认值是: noeviction，当没有合适的驱逐算法时，Redis将在写入操作时返回错误
#
#       这些命令是: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# maxmemory-policy noeviction

# LRU，LFU和最小TTL算法不是精确算法，而是近似算法（为了节省内存），因此您可以调整它以获得速度或准确性。 对于默认值，
# Redis将检查五个键并选择最近使用的键，您可以使用以下配置指令更改样本大小。
#
# 默认值5会产生足够好的结果。 10非常接近真正的LRU，但成本更高。 3更快但不是很准确。
#
# maxmemory-samples 5

############################# LAZY FREEING 懒释放####################################

# 这是4.0加入的新特性
# Redis有两个原函数来删除键。 一个被称为DEL并且是对象的阻塞删除。就是服务器停止处理新命令用同步方式回收与对象关联的所有内存
# 如果删除的键与一个小对象关联，执行DEL命令所需的时间非常短，并且与Redis中的大多数其他O（1）或O（log_N）命令相当 However if the key is associated with an
# 但是，如果key与包含数百万个元素的聚合值关联，则服务器可能会阻塞很长时间（甚至几秒）以完成操作。
#
# 由于上述原因，Redis还提供非阻塞删除原语，例如UNLINK（非阻塞DEL）和FLUSHALL和FLUSHDB命令的ASYNC选项，以便在后台回收内存。 
# 这些命令在不变的时间内执行。 另一个线程将尽可能快地增量释放背景中的对象。
#
# DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB 选项由用户控制。 这取决于应用程序的设计，当然你得了解什么时候使用。 
# 但是，Redis服务器有时必须删除key或刷新整个数据库，这将对其他操作产生影响
# 具体来说，Redis会在以下情况下自动删除对象：
#
# 1) 在删除时，由于maxmemory和maxmemory策略配置的原因，为了为新数据腾出空间，而不超过指定的内存限制。
#
# 2) 由于过期：必须从存储器中删除具有关联时间的密钥（请参阅EXPIRE命令）
#
# 3) 由于存储数据的命令的副作用可能已经存在。 例如，RENAME命令可能会在旧密钥内容被另一个替换时删除旧密钥内容。 
#    同样，SUNIONSTORE或SORT with STORE选项可能会删除现有密钥。 SET命令本身会删除指定键的所有旧内容，以便将其替换为指定的字符串。
#
# 4) 在复制期间，当从服务器与主服务器执行完全重新同步时，整个数据库的内容将被删除，以便加载刚刚传输的RDB文件。
#
# 在上述所有情况下，默认情况下都是以阻塞的方式删除对象，就像调用DEL一样。 要开启调整为“yes”
# 但是，您可以专门配置每个案例，以便使用以下配置指令以非阻塞方式释放内存，就像调用UNLINK一样：

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no

############################## APPEND ONLY MODE 持久化模式###############################

# 默认情况下，Redis异步转储磁盘上的数据集。 这种模式在许多应用程序中已经足够了，但Redis进程或停电可能会导致几分钟的写入丢失（取决于配置的保存点）。
#
# 仅追加文件是一种替代的持久模式，可提供更好的耐用性。例如，使用默认数据fsync策略（稍后在配置文件中看到）
# 可以同时启用AOF和RDB持久性，是没有问题的。 如果在启动时启用AOF，则Redis将加载AOF，即具有更好的持久性保证的文件。
#
# 查看http://redis.io/topics/persistence 获取更多内容.

appendonly no

# AOF文件的名称

appendfilename "appendonly.aof"

# fsync()函数调用告诉操作系统在磁盘上实际写入数据，而不是等待输出缓冲区中的更多数据。 
# 有些操作系统会真正在磁盘上刷新数据，其他一些操作系统只会尽量做到这一点。
#
# Redis支持三种不同的模式：
#
# no: 不要fsync，只需让操作系统在需要时刷新数据。 更快。
# always: 每次写入追加日志后的fsync。 慢，最安全。
# everysec: 每秒只有一次fsync。 折中。
#
# 默认值是“everysec”，因为这通常是速度和数据安全性之间的折中方案。
# 您应该了解您是否可以将其放宽至“否”，以便操作系统在需要时刷新输出缓冲区以获得更好的性能（但是如果您可以忍受某些数据丢失的想法，请考虑快照的默认持久性模式），
# 或者相反你可以使用 always，慢但是安全
#
# 更多详细信息参考:
# http://antirez.com/post/redis-persistence-demystified.html
#
# 如果不确定, 使用 "everysec".
# appendfsync no
# appendfsync always
appendfsync everysec

# 当AOF fsync策略设置为always或everysec，并且后台保存进程（后台保存或AOF日志后台重写）正在对磁盘执行大量I/O时，
# 在某些Linux配置中，Redis可能会在fsync（）调用上阻塞太久。 请注意，目前没有解决此问题的方法，因为即使在其他线程中执行fsync也会阻止我们的同步写入操作的调用。
#
# 为了减轻这个问题，可以使用下面的选项来防止在BGSAVE或BGREWRITEAOF进程中在主进程中调用fsync()。
#
# 这意味着，当另一个child正在保存时，Redis的持久性与“appendfsync none”相同。 实际上，这意味着在最坏的情况下（使用默认的Linux设置）可能会丢失高达30秒的日志。
#
# 如果您有延迟问题，请将其转为“yes”。 否则，将其视为“no”，从持久性角度来看这是最安全的选择。
# 
# 什么意思呢，同时在执行bgrewriteaof操作和主进程写aof文件的操作，两者都会操作磁盘，而bgrewriteaof往往会涉及大量磁盘操作，
# 这样就会造成主进程在写aof文件的时候出现阻塞的情形，现在no-appendfsync-on-rewrite参数出场了。如果该参数设置为no，是最安全的方式，
# 不会丢失数据，但是要忍受阻塞的问题。如果设置为yes呢？这就相当于将appendfsync设置为no，这说明并没有执行磁盘操作，只是写入了缓冲区，
# 因此这样并不会造成阻塞（因为没有竞争磁盘），但是如果这个时候redis挂掉，就会丢失数据。丢失多少数据呢？在linux的操作系统的默认设置下，最多会丢失30s的数据。
# 因此，如果应用系统无法忍受延迟，而可以容忍少量的数据丢失，则设置为yes。如果应用系统无法忍受数据丢失，则设置为no。

no-appendfsync-on-rewrite no

# 自动重写AOF文件
# 当AOF日志大小按指定的百分比增长时，Redis能够自动重写日志文件，隐式调用BGREWRITEAOF。
#
# 这是如何工作的：Redis记得最近一次重写后AOF文件的大小（如果重启后没有发生重写，则使用启动时AOF的大小）。
#
# 上次AOF的大小与当前大小进行比较。 如果当前的大小大于指定的百分比，则触发重写。 此外，您还需要指定要重写的AOF文件的最小值，
# 就是即使达到了百分比增加但文件仍然很小，这可以避免重写AOF
#
# 指定百分比为零以禁用自动AOF重写功能。

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Redis启动加载aof文件，如果发现末尾命令不完整则自动截掉，成功加载前面正确的数据。
# 如果设置为no，遇到此类情况，Redis启动失败，用redis-check-aof 工具手工修复。
aof-load-truncated yes

# 混合 RDB-AOF 持久化格式 
# Redis 4.0 新增了 RDB-AOF 混合持久化格式， 这是一个可选的功能， 在开启了这个功能之后， AOF 重写产生的文件将同时包含 RDB 格式的内容和 AOF 格式的内容， 
# 其中 RDB 格式的内容用于记录已有的数据， 而 AOF 格式的内存则用于记录最近发生了变化的数据， 
# 这样 Redis 就可以同时兼有 RDB 持久化和 AOF 持久化的优点 —— 既能够快速地生成重写文件， 也能够在出现问题时， 快速地载入数据。
aof-use-rdb-preamble no

################################ LUA SCRIPTING  Lua 脚本###############################

# 一个Lua脚本最长的执行时间，单位为毫秒，如果为0或负数表示无限执行时间，默认为5000
lua-time-limit 5000

################################ REDIS CLUSTER redis集群 ###############################
#
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
# in order to mark it as "mature" we need to wait for a non trivial percentage
# of users to deploy it in production.
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#
# 开启集群模式
# cluster-enabled yes

# 集群节点配置文件
# cluster-config-file nodes-6379.conf

# 集群节点超时时间限制
# cluster-node-timeout 15000

# 在进行故障转移的时候全部slave都会请求申请为master，但是有些slave可能与master断开连接一段时间了导致数据过于陈旧，不应该被提升为master。
# 该参数就是用来判断slave节点与master断线的时间是否过长。判断方法是：
# 比较slave断开连接的时间和
#          (node-timeout * slave-validity-factor)+ repl-ping-slave-period
# 如果节点超时时间为三十秒, 并且slave-validity-factor为10，假设默认的repl-ping-slave-period是10秒，即如果超过310秒slave将不会尝试进行故障转移
#
# cluster-slave-validity-factor 10

# 一个主节点在拥有多少个好的从节点的时候就要割让一个从节点出来。例如这个参数若被设为 2，那么只有当一个主节点拥有 2 个可工作的从节点时，它的一个从节点会尝试迁移。
#
# cluster-migration-barrier 1 

# yes表示当负责一个主节点宕机并且下线没有相应的从库进行故障恢复时，整个集群不可用
# no表示当负责一个插槽的主库下线且没有相应的从库进行故障恢复时，集群仍然可用。
#
# cluster-require-full-coverage yes

# 此选项设置为yes时，可防止从站在主站故障期间尝试故障切换其主站
#
# 这在不同情况下非常有用，特别是在多数据中心操作的情况下，如果不是在发生全面的DC故障的情况下，我们希望一方不会被提升。
#
# cluster-slave-no-failover no

# 文档见   http://redis.io web site.

########################## CLUSTER DOCKER/NAT support  docker集群网络支持 ########################

# 在某些部署中，Redis群集节点地址发现失败，因为地址是NAT或由于端口被转发（典型情况是Docker和其他容器）。
#
# 为了使Redis Cluster在这样的环境中工作，需要每个节点都知道其公共地址的静态配置。提供了以下配置：
#
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-bus-port
#
# 如果以上选项未使用，则将使用正常的Redis集群自动检测。
#
# 请注意，在重新映射时，总线端口可能不在客户端端口+ 10000的固定偏移量处，因此您可以根据它们如何重新映射来指定任何端口和总线端口。 
# 如果未设置总线端口，则通常会使用10000的固定偏移量。
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380

################################## SLOW LOG 慢日志###################################

# Redis Slow Log是一个系统，用于记录超过指定执行时间的查询。 执行时间不包括像客户端连接，发送回复等I / O操作，
# 而只是实际执行命令所需的时间（这是命令执行的唯一阶段，其中线程被阻止并且可以在此期间不提供其他请求）。
#
# 只有query执行时间大于slowlog-log-slower-than的才会定义成慢查询，才会被slowlog进行记录。
# slowlog-log-slower-than设置的单位是微妙，默认是10000微妙，也就是10ms 

# slowlog-max-len表示慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列
slowlog-max-len 128

################################ LATENCY MONITOR 监控##############################

# Redis延迟监视子系统在运行时对不同的操作进行采样，以收集与Redis实例的可能延迟来源有关的数据。
#
# 通过 LATENCY命令 可以打印一些图样和获取一些报告，方便监控
#
# 这个系统仅仅记录那个执行时间大于或等于预定时间（毫秒）的操作,这个预定时间是通过latency-monitor-threshold配置来指定的，
#
# 默认情况下，阈值设置为0，即禁用redis监控。实际上启用该监控功能，对redis所增加的成本很少。不过对于一个运行良好的redis，是没有必要打开该监控功能。
latency-monitor-threshold 0

############################# EVENT NOTIFICATION 事件通知##############################

# Redis可以通知Pub / Sub客户端有关key发生的事件。 此功能记录在http://redis.io/topics/notifications
#
# 例如，如果启用密钥空间事件通知，并且客户机对存储在数据库0中的密钥“foo”执行DEL操作，
# 则将通过Pub / Sub发布两条消息：
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# 可以选择Redis通知的事件类别，每个类别都由一个字符标识：
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
# 你可以像下面一样组合几个字符表示那些事件将被通知
#
#  notify-keyspace-events Elg
#
# 如果你想使用__keyevent@<db>__的前缀，并发布过期的键的事件可以用：
#
#  notify-keyspace-events Ex
#
# 默认情况下，采用空字符串所有通知都被禁用，因为大多数用户不需要此功能，并且该功能有一定的开销。 
# 请注意，如果您未指定K或E中的至少一个，则不会传送任何事件。
notify-keyspace-events ""

############################### ADVANCED CONFIG 高级设置 ###############################

# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
list-max-ziplist-size -2

# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0

# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512

# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# HyperLogLog sparse representation bytes limit. The limit includes the
# 16 bytes header. When an HyperLogLog using the sparse representation crosses
# this limit, it is converted into the dense representation.
#
# A value greater than 16000 is totally useless, since at that point the
# dense representation is more memory efficient.
#
# The suggested value is ~ 3000 in order to have the benefits of
# the space efficient encoding without slowing down too much PFADD,
# which is O(N) with the sparse encoding. The value can be raised to
# ~ 10000 when CPU is not a concern, but space is, and the data set is
# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
hll-sparse-max-bytes 3000

# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
#
# The default is to use this millisecond 10 times every second in order to
# actively rehash the main dictionaries, freeing memory when possible.
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply from time to time
# to queries with 2 milliseconds delay.
#
# use "activerehashing yes" if you don't have such hard requirements but
# want to free memory asap when possible.
activerehashing yes

# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can't consume messages as fast as the
# publisher can produce them).
#
# The limit can be set differently for the three different classes of clients:
#
# normal -> normal clients including MONITOR clients
# slave  -> slave clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#
# The syntax of every client-output-buffer-limit directive is the following:
#
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
#
# By default normal clients are not limited because they don't receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
#
# Instead there is a default limit for pubsub and slave clients, since
# subscribers and slaves receive data in a push fashion.
#
# Both the hard or the soft limit can be disabled by setting them to zero.
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# Client query buffers accumulate new commands. They are limited to a fixed
# amount by default in order to avoid that a protocol desynchronization (for
# instance due to a bug in the client) will lead to unbound memory usage in
# the query buffer. However you can configure it here if you have very special
# needs, such us huge multi/exec requests or alike.
#
# client-query-buffer-limit 1gb

# In the Redis protocol, bulk requests, that are, elements representing single
# strings, are normally limited ot 512 mb. However you can change this limit
# here.
#
# proto-max-bulk-len 512mb

# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform according to the specified "hz" value.
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
hz 10

# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
aof-rewrite-incremental-fsync yes

# Redis LFU eviction (see maxmemory setting) can be tuned. However it is a good
# idea to start with the default settings and only change them after investigating
# how to improve the performances and how the keys LFU change over time, which
# is possible to inspect via the OBJECT FREQ command.
#
# There are two tunable parameters in the Redis LFU implementation: the
# counter logarithm factor and the counter decay time. It is important to
# understand what the two parameters mean before changing them.
#
# The LFU counter is just 8 bits per key, it's maximum value is 255, so Redis
# uses a probabilistic increment with logarithmic behavior. Given the value
# of the old counter, when a key is accessed, the counter is incremented in
# this way:
#
# 1. A random number R between 0 and 1 is extracted.
# 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).
# 3. The counter is incremented only if R < P.
#
# The default lfu-log-factor is 10. This is a table of how the frequency
# counter changes with a different number of accesses with different
# logarithmic factors:
#
# +--------+------------+------------+------------+------------+------------+
# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
# +--------+------------+------------+------------+------------+------------+
# | 0      | 104        | 255        | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 1      | 18         | 49         | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 10     | 10         | 18         | 142        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 100    | 8          | 11         | 49         | 143        | 255        |
# +--------+------------+------------+------------+------------+------------+
#
# NOTE: The above table was obtained by running the following commands:
#
#   redis-benchmark -n 1000000 incr foo
#   redis-cli object freq foo
#
# NOTE 2: The counter initial value is 5 in order to give new objects a chance
# to accumulate hits.
#
# The counter decay time is the time, in minutes, that must elapse in order
# for the key counter to be divided by two (or decremented if it has a value
# less <= 10).
#
# The default value for the lfu-decay-time is 1. A Special value of 0 means to
# decay the counter every time it happens to be scanned.
#
# lfu-log-factor 10
# lfu-decay-time 1

########################### ACTIVE DEFRAGMENTATION 自动碎片整理#######################
#
# WARNING 此功能是实验性的。但是在生产过程中也进行了压力测试，并且由多位工程师手动测试一段时间。
#
# 什么是自动碎片整理?
# -------------------------------
#
# 碎片整理允许Redis服务器压缩存储器中小分配和释放数据之间的空间，从而允许回收内存。
#
# 碎片是每个分配器都会发生的自然过程（但幸运的是，Jemalloc并不如此），以及某些工作负载。 
# 通常情况下，需要重新启动服务器以降低碎片，或至少清除所有数据并重新创建。 
# 然而，由于Oran Agra为Redis 4.0实现的这个特性，这个过程可以在运行时以“热”的方式发生，而服务器并不会停止。
#
# 基本上，当碎片超过特定级别时（请参阅下面的配置选项）
# Redis将开始通过利用某些特定的Jemalloc功能（为了理解分配是否导致分段并将其分配到更好的位置）而在连续内存区域中创建值的新副本，
# 并且同时将释放 旧的数据副本。 此过程对所有键重复递增将导致碎片回落到正常值。
#
# 重要了解的几点：
#
# 1. 此功能在默认情况下处于禁用状态，只有在编译Redis时才能使用我们随Redis源代码一起提供的Jemalloc副本。 这是Linux版本的默认设置。
#
# 2. 如果您没有碎片问题，则永远不需要启用此功能。
#
# 3. 一旦遇到碎片，您可以在需要时使用命令“CONFIG SET activedefrag yes”启用此功能。
#
# 配置参数能够微调碎片整理过程的行为。 如果您不确定它们的含义，最好保持默认值不变。

# 开启碎片整理
# activedefrag yes

# 最小量的碎片浪费来启动主动碎片整理
# active-defrag-ignore-bytes 100mb

# 碎片启动主动碎片整理的最小百分比
# active-defrag-threshold-lower 10

# 我们使用最大努力的最大碎片百分比
# active-defrag-threshold-upper 100

# 在CPU百分比中进行碎片整理的最小努力
# active-defrag-cycle-min 25

# 在CPU百分比中进行碎片整理的最大努力
# active-defrag-cycle-max 75


```