---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/designpattern/proxy.png
title: 代理模式（proxy）
date: 2018-3-21 22:30:00
toc: true
tags: 
- 代理模式
categories:
- 设计模式
---
代理(Proxy)是一种设计模式,提供了间接对目标对象进行访问的方式;即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的功能上,增加额外的功能补充,即扩展目标对象的功能.符合开闭原则，即在对既有代码不改动的情况下，增加代码进行功能的扩展。
<!--more-->

#  背景

歌手与经纪人之间就是被代理和代理的关系,歌手出演活动的时候，歌手就是一个目标对象,他只要负责活动中的节目,而其他琐碎的事情就交给他的经纪人。这就是典型的代理模式

# 分析

代理模式实现分为三种:

- 静态代理
- 动态代理
- cglib代理

## 静态代理

1. 抽象主题类
```java
public interface Subject {
    /**
     * 接口方法
     */
    public void request();
}
```

2. 具体主题类
```java
public class ConcreteSubject implements Subject {
    /**
     * 具体的业务逻辑实现
     */
    @Override
    public void request() {
        //业务处理逻辑
    }
}
```

3. 代理类
```java
public class Proxy implements Subject {

    /**
     * 要代理的实现类
     */
    private Subject subject = null;

    /**
     * 默认代理自己
     */
    public Proxy() {
        this.subject = new Proxy();
    }

    public Proxy(Subject subject) {
        this.subject = subject;
    }

    /**
     * 构造函数，传递委托者
     *
     * @param objects 委托者
     */
    public Proxy(Object... objects) {
    }

    /**
     * 实现接口方法
     */
    @Override
    public void request() {
        this.before();
        this.subject.request();
        this.after();
    }

    /**
     * 预处理
     */
    private void before() {
        //do something
    }

    /**
     * 后处理
     */
    private void after() {
        //do something
    }
}
```

4. 客户端类
```java
public class Client {
    public static void main(String[] args) {
        Subject subject = new ConcreteSubject();
        Proxy proxy = new Proxy(subject);
        proxy.request();
    }
}
```

## 动态代理

静态代理的意思呢，通过上面的案例就可以发现，他只能代理一个对象，可以理解为一个经纪人只能代理一个歌手的活动，这样就会产生很多的代理人。所以我们想办法通过一个代理类完成全部的代理功能，那么我们就需要用动态代理.
动态代理就是在运行时，通过反射机制实现动态代理，并且能够代理各种类型的对象。

**在Java中要想实现动态代理机制，需要java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy类的支持。**

#### java.lang.reflect.InvocationHandler接口的定义

```java
package java.lang.reflect;
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

参数解释：

- Object proxy  被代理对象
- Method method 要调用的方法
- Object[] args 方法调用时所需要的参数

#### java.lang.reflect.Proxy类的定义

```java
public class Proxy implements java.io.Serializable {

    private static final long serialVersionUID = -2222568056686623797L;

    /** parameter types of a proxy class constructor */
    private static final Class<?>[] constructorParams =
        { InvocationHandler.class };

    /**
     * a cache of proxy classes
     */
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

    /**
     * the invocation handler for this proxy instance.
     * @serial
     */
    protected InvocationHandler h;

    /**
     * Prohibits instantiation.
     */
    private Proxy() {
    }
    ········
    //中间其他方法就不介绍了
    //主要是这个创建的方法
    @CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h) throws IllegalArgumentException {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
}
```

参数说明：

- ClassLoader loader 类的加载器
- Class<?>[] interfaces 得到全部的接口
- InvocationHandler h 得到InvocationHandler接口的子类的实例

主要就是利用反射构建出代理类

### 例子

下面就是例子

1. 目标接口类

```java
//目标类接口
interface ISinger{
    void singing();
}
```

2. 目标歌手类

```java
//目标类
class Zxy implements ISinger{

    @Override
    public void singing() {
        System.out.println("张学友唱歌");
    }
}
```

3. 代理功能

```java
class SingerUtils{
    public static void method1() {
        System.out.println("增强方式一");
    }
    
    public static void method2() {
        System.out.println("增强方式二");
    }
}
```

4. 代理人预编译处理

```java
class SingerInvocationHandle implements InvocationHandler{
    private Object target;
    public void setTarget(Object target) {
        this.target = target;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            SingerUtils.method1();
            method.invoke(target, args);
            SingerUtils.method2();
            return null;
    }
}
```

5. 创建代理人

```java
 class SingerProxyFactory{
    public static Object getProxy(Object target) {
        SingerInvocationHandle handle = new SingerInvocationHandle();
        handle.setTarget(target);
        Object proxy = Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), handle);
        return proxy;
    }
 }
```

6. 测试

```java
public class ProxyDemo {
    public static void main(String[] args) {
      //张学友要开演唱会，首先张学友得唱歌
      ISinger zxy = new Zxy();
      //但是呢，唱歌有一些列的事情要准备，要找场地，音响，要卖门票等等事情，这就需要代理人来做，所以就创建一个歌手的代理人来办这些事情
      ISinger proxy =(ISinger) SingerProxyFactory.getProxy(zxy);
      //演唱会开始
      proxy.run();
    }

}
```

## cglib代理

上面的代码中有个相同点就是都要求目标对象是实现一个接口的对象,然而并不是任何对象都会实现一个接口，也存在没有实现任何的接口的对象,这时就可以使用继承目标类以目标对象子类的方式实现代理,这种方法就叫做:Cglib代理，也叫作子类代理。

**cglib 依赖第三方的jar实现**

实现方式
只列举关键代码

```java
    //实例化一个增强器，也就是cglib中的一个class generator
    Enhancer enhancer = new Enhancer();
    //设置目标类
    enhancer.setSuperclass(XXX.class);
    //设置拦截对象，这里直接使用匿名内部类写法
    enhancer.setCallback(new MethodInterceptor() {
        @Override
        public Object intercept(Object object , Method method, Object[] args, MethodProxy proxy) throws Throwable {
            //todo 要做的前置处理

            //使用proxy的invokeSuper方法来调用目标类的方法
            proxy.invokeSuper(object, args);
            // todo 要做的后置处理
            return null;
        }
        
    });


 //然后调用时,用enhancer 创建出代理目标对象

 XXX xxx = (XXX) enhancer.create();
 xxx.run();
```