---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/javautil/map/hashmap.png
title: HashMap源码记录
date: 2017-06-15 18:30:00
toc: true
tags: 
- HashMap Map
categories:
- java.util
---
hashmap在java里是出现频率较高的类，不管是工作还是面试，掌握hashmap的原理是很重要的，本文也将从整体到细节介绍hashmap
<!--more-->

# 摘要
hashmap在java里是出现频率较高的类，不管是工作还是面试，掌握hashmap的原理是很重要。随着JDK版本的更新，HashMap底层的实现进行了优化，例如引入红黑树的数据结构和扩容的优化等。本文结合JDK1.7和JDK1.8的区别，深入探讨HashMap的结构实现和功能原理。

# 简介
java中Map数据结构定义了一个主要的接口：java.util.Map。主要实现这个接口的类是：HashMap、HashTable、LinkedHashMap、TreeMap。关系如下
![](http://icepear.oss-cn-shenzhen.aliyuncs.com/javautil/map/Map.png)
下面针对各个实现类的特点做一些说明：

(1) HashMap：
1. 访问速度快hashcode直接定位，但遍历顺序却是不确定的。 
2. 最多只允许一条记录的键为null，允许多条记录的值为null。
3. 非线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

(2) Hashtable：**没什么卵用**，基本上与HashMap类似、虽然线程安全，但介于HashMap与ConcurrentHashMap之间，不上不下。

(3) LinkedHashMap：LinkedHashMap是HashMap的一个子类，保存了记录的插入顺序。

(4) TreeMap：
1. 它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。
2. 如果使用排序的映射，建议使用TreeMap。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。

本文主要讲HashMap的实现原理，结合1.7和1.8主要从存储结构，常用方法，定位，扩容等方面展开

# 存储结构
存储结构如图：
![](http://icepear.oss-cn-shenzhen.aliyuncs.com/javautil/map/hashmap.png)
HashMap在1.7中只用到了数组和链表，代码也只有一千多行。上图展示的是1.8的存储结构，在1.8中加入了红黑树，在链表大于8的时候转换为红黑树；扩容后导致红黑树节点在小于6时，又会转换成链表。代码量虽然翻倍了，带来的确实性能的提升。