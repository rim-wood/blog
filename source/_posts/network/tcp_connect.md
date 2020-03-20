---
title: TCP连接及传输
date: 2018-08-09 19:30:02
toc: true
tags: 
- TCP
- network
categories:
- network
---
OSI七层协议模型主要是：应用层、表示层、会话层、传输层、网络层、数据链路层、物理层。每层都有对应的协议，在传输层主要是TCP、UDP协议，然后通过网络层的IP协议进行网络传输
<!--more-->

# 网络结构

![](http://icepear.oss-cn-shenzhen.aliyuncs.com/network/%E7%BD%91%E7%BB%9C%E7%BB%93%E6%9E%84.png)
![](http://icepear.oss-cn-shenzhen.aliyuncs.com/network/%E7%BB%93%E6%9E%84%E5%8D%8F%E8%AE%AE.png)

# TCP协议报文

![](http://icepear.oss-cn-shenzhen.aliyuncs.com/network/tcp-msg.jpg)

tcp报文格式如上图所示，主要解释一下首部，因为数据部分主要是应用层协议直接包装的，如下图

![](http://icepear.oss-cn-shenzhen.aliyuncs.com/network/xieyibaozhuang.jpg)

|字段|含义|
| - | - |
|URG|紧急指针是否有效。为1，表示某一位需要被优先处理|
|ACK|确认号是否有效，一般置为1。|
|PSH|提示接收端应用程序立即从TCP缓冲区把数据读走。|
|RST|对方要求重新建立连接，复位。|
|SYN|请求建立连接，并在其序列号的字段进行序列号的初始值设定。建立连接，设置为1|
|FIN|希望断开连接。|

 序列号seq：占4个字节，用来标记数据段的顺序，TCP把连接中发送的所有数据字节都编上一个序号，第一个字节的编号由本地随机产生；给字节编上序号后，就给每一个报文段指派一个序号；
 序列号seq就是这个报文段中的第一个字节的数据编号。

 确认号ack：占4个字节，期待收到对方下一个报文段的第一个数据字节的序号；因此当前报文段最后一个字节的编号+1即为确认号。

 确认ACK：占1位，仅当ACK=1时，确认号字段才有效。ACK=0时，确认号无效

 同步SYN：连接建立时用于同步序号。当SYN=1，ACK=0时表示：这是一个连接请求报文段。若同意连接，则在响应报文段中使得SYN=1，ACK=1。因此，SYN=1表示这是一个连接请求，或连接接受报文。SYN这个标志位只有在TCP建产连接时才会被置1，握手完成后SYN标志位被置0。

 终止FIN：用来释放一个连接。FIN=1表示：此报文段的发送方的数据已经发送完毕，并要求释放运输连接
**注意小写的ack为确认序号，大写的ACK为确认符**

正因为有seq和ack的加持，所以tcp传输是可靠的。TCP为了保证不发生丢包，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收。然后接收端实体对已成功收到的包发回一个相应的确认（ACK）；如果发送端实体在合理的往返时延（RTT）内未收到确认，那么对应的数据包就被假设为已丢失将会被进行重传。TCP用一个校验和函数来检验数据是否有错误；在发送和接收时都要计算校验和。

# 握手过程

![](http://icepear.oss-cn-shenzhen.aliyuncs.com/network/tcp-connect.jpg)

第一次握手：建立连接时，客户端发送SYN请求连接包（seq=x,x为客户端随机数）到服务器，并进入SYN_SENT状态，等待服务器确认。

第二次握手：服务器收到SYN包，必须确认客户的SYN（ack=x+1），同时自己也发送一个SYN=1、ACK=1的包（seq=y，y为服务端随机数），即SYN+ACK包，此时服务器进入SYN_RECV状态；
（这里有个特别注意的地方：很多人会问为什么不直接返回ACK包，不就可以少一次握手了吗。因为TCP之可以是可靠的，来源于双向通讯，并且双方来知会对方开始的序列号seq。
第一种情况：已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送ack包。（校注：此时因为client没有发起建立连接请求，所以client处于CLOSED状态，接受到任何包都会丢弃
第二种情况：如果服务器发送对这个延误的旧连接报文的确认的同时，客户端调用connect函数发起了连接，就会使客户端进入SYN_SEND状态，当服务器那个对延误旧连接报文的确认传到客户端时，因为客户端已经处于SYN_SEND状态，所以就会使客户端进入ESTABLISHED状态，此时服务器端反而丢弃了这个重复的通过connect函数发送的SYN包，见第三个图。而连接建立之后，发送包由于SEQ是以被丢弃的SYN包的序号为准，而服务器接收序号是以那个延误旧连接SYN报文序号为准，导致服务器丢弃后续发送的数据包）但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。）

第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=y+1），此包发送完毕，客户端和服务器进入ESTABLISHED（TCP连接成功）状态，完成三次握手。

# 挥手过程

![](http://icepear.oss-cn-shenzhen.aliyuncs.com/network/tcp-close.jpg)

第一次挥手：客户端进程发出连接释放报文，并且停止发送数据。释放数据报文首部，FIN=1，其序列号为seq=u（等于前面已经传送过来的数据的最后一个字节的序号加1），此时，客户端进入FIN-WAIT-1（终止等待1）状态。 TCP规定，FIN报文段即使不携带数据，也要消耗一个序号。

第二次挥手：服务器收到连接释放报文，发出确认报文，ACK=1，ack=u+1，并且带上自己的序列号seq=v，此时，服务端就进入了CLOSE-WAIT（关闭等待）状态。TCP服务器通知高层的应用进程，客户端向服务器的方向就释放了，这时候处于半关闭状态，即客户端已经没有数据要发送了，但是服务器若发送数据，客户端依然要接受。这个状态还要持续一段时间，也就是整个CLOSE-WAIT状态持续的时间。而此时客户端就进入FIN-WAIT-2（终止等待2）状态，等待服务器发送连接释放报文（在这之前还需要接受服务器发送的最后的数据）。

第三次挥手：服务器将最后的数据发送完毕后，服务器再次发送确认关闭请求FIN=1，ACK=1，ack=u+1，假定此时的序列号为seq=w，客户端收到服务器的确认请求后，此时，服务器就进入了LAST-ACK（最后确认）状态，等待客户端的确认。

第四次挥手：客户端收到服务器的连接释放报文后，必须发出确认，ACK=1，ack=w+1，而自己的序列号是seq=u+1，此时，客户端就进入了TIME-WAIT（时间等待）状态。注意此时TCP连接还没有释放，必须经过2∗∗MSL（最长报文段寿命）的时间后，当客户端撤销相应的TCB后，才进入CLOSED状态。服务器只要收到了客户端发出的确认，立即进入CLOSED状态。同样，撤销TCB后，就结束了这次的TCP连接。可以看到，服务器结束TCP连接的时间要比客户端早一些。