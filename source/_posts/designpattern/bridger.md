---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/designpattern/bridger.png
title: 桥接模式（bridger）
date: 2018-3-19 21:30:00
toc: true
tags: 
- 桥接模式
categories:
- 设计模式
---
桥接模式,组合两个对象的机器,比如我们拿画笔为例子，笔有铅笔、蜡笔、水彩笔等等，而不同的笔又有不同的颜色，如果我们用七种颜色和三只不同种类的笔的话，按照普通的逻辑，我们就需要创建二十一个类来对应不同的笔，但是有的同学就会讲了可以用工厂模式啊，对的，但是我们看待问题的角度不一样，之所以称工厂模式为创建型模式，就是用这种方法来构建出现实对象。而桥接模式是属于结构型，就是两个对象通过用一种协同的模式，让他们达到想要的逻辑。
<!--more-->
# UML

uml图就是banner图，不再复述

# 案例

还是采用简介上面的例子，用桥接模式来实现它。

1. 首先我们创建图形抽象类，这个抽象类是关键，它里面必须包含我们实现类的接口，也就是说颜色的接口

```java
public abstract class Shape {
    Color color;
 
    public void setColor(Color color) {
        this.color = color;
    }
    
    public abstract void draw();
}
```

2. 然后是三个形状


```java 
public class Circle extends Shape{
 
    public void draw() {
        color.bepaint("正方形");
    }
}

public class Rectangle extends Shape{
 
    public void draw() {
        color.bepaint("长方形");
    }
 
}

public class Square extends Shape{
 
    public void draw() {
        color.bepaint("正方形");
    }
 
}
```

3. 颜色接口 

```java
public interface Color {
    public void bepaint(String shape);
}
```

4. 然后是具体的颜色

```java
public class White implements Color{
 
    public void bepaint(String shape) {
        System.out.println("白色的" + shape);
    }
 
}

public class Gray implements Color{
 
    public void bepaint(String shape) {
        System.out.println("灰色的" + shape);
    }
}

public class Black implements Color{
 
    public void bepaint(String shape) {
        System.out.println("黑色的" + shape);
    }
}
```

5. 最关键就看我们这个调用过程了，我们先创建白色的实现类，然后创建正方形图形，关键是融合的这一点，我们将图形上色，运用java多态的特性，设置白色的实现类，然后绘制图形的方法里再反过来调用颜色的绘制方法就实现了两个独立对象合作完成一件事情，而不是创建出一只白色的笔，再去绘制，这就区分出工厂和桥接的区别了。
```java
public class Client {
    public static void main(String[] args) {
        //白色
        Color white = new White();
        //正方形
        Shape square = new Square();
        //白色的正方形
        square.setColor(white);
        square.draw();
    }
}
```