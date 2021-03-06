---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/api-security/api-security.jpg
title: 接口安全策略
date: 2017-08-09 19:30:02
toc: true
tags: 
- api
- security
categories:
- security
---
在做接口的时候，我们经常会对安全这方面进行考虑，包括认证授权，接口安全等方面，下面就介绍一种接口安全的设计，如果有什么其他好的设计，都可以在评论区提出交流
<!--more-->
# 背景
为什么要对接口进行安全控制呢，主要是出于这几方面的因素
1. 防伪装攻击
2. 防篡改攻击
3. 防重放攻击
4. 防数据信息泄露

# 解决办法
解决办法有很多，一般常见的做法是token校验然后加上HTTPS。
针对这四种情况，假如接口没做任何安全策略是很容易被攻击的。
对于第一种防伪装攻击，一般的接口都会带有token认证，可以过滤掉；
对于第二种以及第三种，假如别人截获了请求，修改了参数，token机制是没办法做到安全的，所以有效的办法就是加密，加密的办法有很多。
下面以图介绍我用的一种
![](http://icepear.oss-cn-shenzhen.aliyuncs.com/other/api-security/method.png)
这种方法的步骤是

1. 将请求参数（一般是json形式）加上约定好的salt进行AES加密，加密出来的是byte，一般用base64进行转码输出，也可做位偏移，得到一个加密字符串；然后加上后端的token，加上时间戳，进行MD5加密得到一个定长的sign字符串
2. 将sign，toekn，时间戳，参数组成http请求去访问接口
这样做的好处就是，针对第二三种攻击，别人拦击你的请求进行篡改，后端根据参数+约定好的salt进行AES加密再加上+token+时间戳进行MD5加密，与sign比对发现两个MD5不一致，就知道，肯定有人篡改了参数，就可以拒绝这样的请求。
如果你不想参数直接暴露出来也可以进行AES加密传给后端。这样就可以避免第四种情况

# 总结
以上都是自己的愚见，如果有更好的设计方法，欢迎交流学习
