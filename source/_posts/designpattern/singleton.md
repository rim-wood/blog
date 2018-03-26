---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/designpattern/singleton/singleton.png
title: 单例模式（Singleton pattern）
date: 2017-3-15 18:30:00
toc: true
tags: 
- 单例模式
categories:
- 设计模式
---
单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
单例类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
<!--more-->
## 简介
单例模式是常用的设计模式之一，在spring中也大量采用到，单例模式一般分为**饿汉式**和**饱汉式**这两种实现，但除了这两种其实还可以细分到线程安全领域，所以也就还会有线程安全的实现方式

## 基本定律
1. 私有的静态实例变量
2. 私有的构造方法
3. 公共的获取实例的方法
满足这三个条件，基本就可以写出单例了。下面详细分析几种写法

## 单例模式的写法

### 饿汉式，最常见实现（可用）
这种写法比较简单，而且一般来讲也是线程安全的，因为在类装载的时候就完成了实例化，避免同步问题

```java
public class Singleton{
    private final static Singleton instance = new Singleton();
    private Singleton();
    public Singleton getInstance(){
        return  instance;
    }
}
```

### 饿汉式，静态代码块（可用）
和上面基本类似，只是把实例化的过程抽离到了静态代码块中

```java
public class Singleton{
    private static Singleton instance;
    static{
        instance = new Singleton();
    }
    private Singleton();
    public Singleton getInstance(){
        return  instance;
    }
}
```

### 饿汉式，静态内部类，推荐
这种方式跟上两种虽然都采用类加载机制来保证实例化时只有一个线程，但是上两种的做法是只要类被加载就会被实例化，假如这个类没有被用到，就会浪费内存。这种方式的好处就是起到了懒加载的效果，静态内部类在singleton类被加载时并不会立即实例化，而是在需要时实例化，调用getInstance才会去装载内部类

```java
public class Singleton{

    private Singleton();
    
    private static class SingletonInstance{
        private static final Singleton instance = new Singleton();
    } 

    public static Singleton getInstance(){
        return  SingletonInstance.instance;
    }
}
```


### 饱汉式，线程不安全（不可用）
谈完饿汉再谈饱汉，饱汉一般的写法是，getInstance的时候我在实例化，当多个线程同时调用getInstance就会导致线程不安全
```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```


### 饱汉式，线程安全（可用，但效率低）
那怎么可以达到线程安全呢，实现也很简单，不就是getInstance会导致同步问题嘛，那就加一个同步方法，但是也不太推荐这种做法，毕竟每次都要同步效率是很低的
```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```


### 饱汉式，线程安全升级（不可用）
那同步效率低该怎么改进呢，于是想到就是静态代码块咯，效率会高一点吧
```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                instance = new Singleton();
            }
        }
        return instance;
    }
}
```
这样就没问题了吗？仔细想想，假如在判断if (instance == null)时，另外一个线程也放好判断完了，两个线程还是会同时实例化

### 饱汉式，双重检查，线程安全，推荐
这时双重检测是最完美不过的了，首先保证实例的原子性，然后双重判断，这样就不会出现实例化多个的情况了
```java
public class Singleton {
    //volatile 保证变量的原子性，都是从主线程内存中读取
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```


### 枚举，推荐
jdk1.5 引入枚举后，使用枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。
```java
public enum Singleton {
    INSTANCE;
    public void whateverMethod() {

    }
}
```

