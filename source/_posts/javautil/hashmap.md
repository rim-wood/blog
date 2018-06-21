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
![类图](http://icepear.oss-cn-shenzhen.aliyuncs.com/javautil/map/Map.png)
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

![结构](http://icepear.oss-cn-shenzhen.aliyuncs.com/javautil/map/hashmap.png)

HashMap在1.7中只用到了数组和链表，代码也只有一千多行。上图展示的是1.8的存储结构，在1.8中加入了红黑树，在链表大于8的时候转换为红黑树；扩容后导致红黑树节点在小于6时，又会转换成链表。代码量虽然翻倍了，带来的确实性能的提升。
HashMap 1.8结构代码如下

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16 默认hashmap的容量
static final int MAXIMUM_CAPACITY = 1 << 30; //hashmap最大的容量
static final float DEFAULT_LOAD_FACTOR = 0.75f; // 负载因子，跟扩容有关，后面会提到
static final int TREEIFY_THRESHOLD = 8; //转换成红黑树的阀值
static final int UNTREEIFY_THRESHOLD = 6;//红黑树转成链表的阀值
static final int MIN_TREEIFY_CAPACITY = 64;//转换成红黑树最小需要hash数组中的数量大于64

// Node 节点
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash; //hash值
        final K key; // 键
        V value; // 值
        Node<K,V> next; // 链表下一个节点
        ··· //一些默认函数省略掉
    }

transient Node<K,V>[] table; // 数组
transient Set<Map.Entry<K,V>> entrySet; //缓存了hashmap中的node
transient int size; //存储的node数量
transient int modCount; //结构修改次数，跟Fail-Fast机制有关
int threshold; // 能负载的node数量（Capacity*loadFactor）初识值 16*0.75 =12
final float loadFactor; //负载因子
//构造函数，程序员传初始化容量和负载因子进来（一般不用）因为0.75这个值是经过大量的统计计算得出来的结论，一般不更改
//也就是说容量到达Capacity的0.75时，进行扩容操作。不至于loadFactor过大，导致hash碰撞过多，太小，扩容次数太多影响性能
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                            initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
// 构造函数，初始化容量（最好也是2的N次幂）这个会影响hash定位
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
//最常用的构造函数，使用默认的0.75作为负载因子
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; 
}
// 构造函数，传一个map进来，复制到本hashmap
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);//这个函数将用到m这个map中的entrySet，目的是将m中的node通过put函数复制到本hashmap中
}
//树节点
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // 根节点
    TreeNode<K,V> left;    // 左节点
    TreeNode<K,V> right;   // 右节点
    TreeNode<K,V> prev;    // 删除后需要取消链接
    boolean red;
    //一些红黑树操作的函数
    ···
}
```

# 实现说明
主要从hashmap的主要三个步骤进行说明，**hash定位**，**插入**，**扩容**

## hash定位

hash定位是HashMap比较核心的方法了，上面我们了解到HashMap的结构为数组，既然是数组，就会有下标，那么这个hash值就是数组的下标，也正是HashMap可以根据key快速查找定位到Value的原因
下面看下1.7中hash的源码

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
static int indexFor(int h, int length) {
     return h & (length-1);
}
```

在1.8中做了改进

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

1.8中虽然取消了indexFor函数，但是在put和get的时候都通过了 **tab[i = (n - 1) & hash]** 来定位，原理跟1.7是一样的
可以看出，不管是哪个版本，算法大致分为 **取key的hashcode**，**高位运算** **取模运算**
为了让hash值均匀分布，会采用高位运算，让小的值的高位也参与运算;然后拿运算后key的hashcode对数组的长度进行模运算定位数组中的位置，但是细心一点就会发现，取模运算并没有使用%运算，
因为模运算是很耗费性能的，所以采用与运算，可以说这个与运算设计的是相当精巧了。这也就是为什么数组的长度一定要是2的N次幂长度的原因，因为当length等于2的n次幂时，h&(length-1)就等于h%length

## put实现

我们知道HashMap的时间复杂度为O(1)，但是当Hash碰撞率过高时hashmap就会遍历链表，导致某些情况时间复杂度提高至O(n)；所以好的hash算法以及扩容机制是相当重要的，下面就讲讲hashmap插入值的原理
单纯的代码加文字，表现力可能没那么强，所以采用文字加流程图的方式进行说明：
流程图：

![put](http://icepear.oss-cn-shenzhen.aliyuncs.com/javautil/map/hashmap_put.png)

源码解释:

 ```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 第1步 判断table是否为null，如果为空进行扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 第2步 判断table中hash值对应的位置是否有值，就是table[i],如果没值，就new一个Node节点存入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //第3步 
        else {
            Node<K,V> e; K k;
            //第3步 如果table[i]有值，就判断table[i]的key是否和要插入的值的key相同，相同则令节点e等于table[i]
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //第4步 不相同则判断table[i]是否为树节点，为树节点则将新值插入红黑树,如果红黑树里面存在这个值，也令e等于这个树节点，否则e等于null
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //第5步 key不相同也不是树节点，则按链表的方式处理，判断链表中是否存在这个值，存在则e等于这个节点，否则将这个值插入到链表，e等于null；插入后判断链表大小是否大于8，大于则转换为红黑树
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //第6步 判断e是否为空，不为空说明有相同key的节点，需要进行覆盖，并返回old值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //第7步 如果是old值覆盖，则modCount不变，modCount只有在插入节点才会变化；最后判断table大小是否大于负载值threshold，大于则进行扩容
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
 ```

相比较于1.7的插入操作，1.8的优化是引入红黑树，不至于在hash碰撞频繁的情况下，导致链表过长查询速度变慢的问题。
在插入操作里最后就是扩容函数，想必很想知道hashmap是怎么扩容的，下面详细讲讲扩容原理

## 扩容

resize就是更换容器，小桶放不下了得换个大桶。前面我们了解到，table是个数组，我们也知道数组是有大小的，不能动态的扩张，但是HashMap对象却可以不停的添加元素，这也真是resize帮我们做的，
方法就是使用一个新的数组代替已有的容量小的数组。下面我们分析下resize的源码。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //如果hashmap存在值
    if (oldCap > 0) {
        // 超过最大值就不再扩充
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //否则负载数量左移一位，翻倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    //负载量大于0，则用负载量替换容量
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    //都为0则初始化容量和负载量
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //计算新的负载量上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //将原来的数据放入新的数组中
    if (oldTab != null) {
        //遍历老数组
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //oldTab[j]存在数据
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                //不是链表，直接定位值并插入
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //树节点操作，里面实现不细讲（实际上有点复杂）
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                //数组节点操作（非常精辟的一段操作，简直牛逼）
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```