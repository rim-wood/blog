---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/designpattern/adapter/adapter.png
title: 适配器模式（adapter）
date: 2018-3-18 20:30:00
toc: true
tags: 
- 适配器模式
categories:
- 设计模式
---
适配器模式,连接两个接口的桥梁 ,适配器模式有两种，类适配器和对象适配器.
<!--more-->

# 类适配器

类适配器主要是使用继承的方式连接两个接口,众所周知，苹果电脑连投影仪都是需要转接头转换的，那么我们就用这个做例子。

电脑都可以投影，它又个投影的方法
```java
public interface Computer{
    void projection();
}
```
苹果电脑的实现类
```java
public class Mac implement Computer{
    public void projection(){
            // todo
    }
}
```
投影仪有个展示的方法
```java
public interface Projector{
      void show();
}
```
假如你的工程中有这几个类，然后你发现，show()方法中要写的操作，就是Mac中的操作，如果你的投影仪要想直接支持苹果电脑接口，就必须在show方法中实现放映的方法，但是如果又来一个其他类型的电脑，则必须又要实现另外一个接口，那投影仪商就不要活了，所以，投影仪也比较硬气，老子就一个洞爱上不上，你不习惯，你可以戴螺纹的，或者戴胶点的；所以呢，小杜和小冈就这么发家致富了。

那么现在就来介绍小杜，哦不，介绍适配器
```java
public class ProjectorAdapter extends Mac implement Projector{
    //实现投影仪的展示方法
    public void show(){
        //调用苹果电脑的投影方法
        projection();
    }
}
```

但是呢，此种类适配器不够灵活，java又不支持多继承，所以再来个其他类型的电脑，则又必须开一个适配器，才对得上。

# 对象适配器

那么我们不用继承，可以用什么呢，相必大家都知道，不继承的话，那就直接放里面去new吧。

我们改造一下小杜投影仪适配插头。

```java
public class ProjectorAdapter implement Projector{
    private Mac mac;

    public PlayerAdapter (){
        this.mac = new Mac();
    } 
    //实现投影仪的示方法
    public void show(){
        //调用苹果电脑的投影方法
        mac.projection();
    }
}
```

但是这样好像还不太行，遵循可以给多个电脑插的原则，只允许mac插好像不太合适。

再改造一下，直接传电脑进来不就成了么。
```java
public class ProjectorAdapter implement Projector{
    private Computer computer;

    public PlayerAdapter (Computer computer){
        this.computer = computer;
    } 
    //实现投影仪的示方法
    public void show(){
        //调用电脑的投影方法，至于什么电脑就看外面实例化的是个什么电脑
        computer.projection();
    }
}
```