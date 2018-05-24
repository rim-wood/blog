---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/designpattern/builder/builder.png
title: 建造者模式（builder）
date: 2018-3-14 16:30:00
toc: true
tags: 
- 建造者模式
categories:
- 设计模式
---
建造者模式属于设计模式中的构造性模式，也就是说该模式基本上是用在构建对象的时候。它的宗旨就是将一个复杂的事物，进行一步一步的构建，需要什么就拼接成什么。
<!--more-->

# 源代码地址
[源码连接](https://github.com/rim-wood/design-patterns/tree/master/builder)

# UML
类图如下：
![](http://icepear.oss-cn-shenzhen.aliyuncs.com/designpattern/builder/builder.png)

**Derictor**: 指挥人，哔哩吧啦说你这玩意儿要怎么建
**builder**：抽象建造者，怎么建的一系列方法
**concreteBuilder**：具体的劳动力，实现建造的方法
**product**：具体的产品类

#源码分析
给大家讲一个生动形象的例子，小刚喜欢大保健，没事就去洗洗脚按按摩。从这句话里面，我们就可以分析出一个模式。
**大保健**：具体的产品
**小刚**：享受服务的指挥人，要什么服务都由他决定
**老板**：大保健服务的建造者

传统模式下，小刚要一个大保健服务是不是说：老板，做个大保健（new dabaojian()）服务就出来了。但是存在一个问题，这样叫出来的是个大保健，但是
具体哪一些服务没有指定，老板肯定问你：都想做什么服务；那服务种类很多，不可能每次都是
``` java
new Dabaojian("洗脚");
new Dabaojian("洗脚","按摩");
new Dabaojian("洗脚","按摩","papapa");
``` 
服务千姿百态这样就会存在弊端。老板也想到这个问题了
所以老板就会拼接各种服务给客人做成一个大保健
具体见代码分析
```java
public class Bighealthcare {
    private String xijiao;
    private String anmo;
    private String papapa;

    /**
     * 传统享受大保健服务就洗洗脚
     * @param xijiao
     */
    public Bighealthcare(String xijiao){
        this.xijiao = xijiao;
    }
    /**
     * 舒服一点就加个按摩
     * @param xijiao
     */
    public Bighealthcare(String xijiao,String anmo){
        this.xijiao = xijiao;
        this.anmo = anmo;
    }

    /**
     * 想更舒服就搞下特殊服务
     * @param xijiao
     * @param anmo
     * @param papapa
     */
    public Bighealthcare(String xijiao,String anmo,String papapa){
        this.xijiao = xijiao;
        this.anmo = anmo;
        this.papapa = papapa;
    }
    /**
     * 建造者模式的大保健服务
     */
    public Bighealthcare(Builder builder){
        this.xijiao = builder.xijiao;
        this.anmo = builder.anmo;
        this.papapa = builder.papapa;
    }
    protected static class Builder{
        protected String xijiao;
        protected String anmo;
        protected String papapa;

        protected Builder xijiao(String xijiao){
            this.xijiao = xijiao;
            return this;
        }

        protected Builder anmo(String anmo){
            this.anmo = anmo;
            return this;
        }

        protected Builder papapa(String papapa){
            this.papapa = papapa;
            return this;
        }

        protected Bighealthcare build(){
            return new Bighealthcare(this);
        }
    }

    public void service(){
        System.out.print(this.toString());
    }
```
小刚要做大保健了
``` java
public class Test {
    public static void main(String args[]){
        //传统模式下的大保健服务
        Bighealthcare tradition = new Bighealthcare("洗个脚","按个摩","ppp");
        tradition.service();

        //建造器模式下的大保健服务，先构造一个大保健服务，具体要哪些项目可以一个个拼装，扩展方便
        //运用《effective java》中说的，当构造方法过多时就应该要考虑使用构造器，其实就是建造者模式
        //简单大保健
        Bighealthcare simpleBighealthcare = new Bighealthcare.Builder().xijiao("洗个脚").build();
        simpleBighealthcare.service();
        //一般大保健
        Bighealthcare generalBighealthcare = new Bighealthcare.Builder().xijiao("洗个脚").anmo("按个摩").build();
        generalBighealthcare.service();
        //高级大保健
        Bighealthcare specialBighealthcare = new Bighealthcare.Builder().xijiao("洗个脚").anmo("按个摩").papapa("ppp").build();
        specialBighealthcare.service();
    }
}
```