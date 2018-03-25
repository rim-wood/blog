---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/multithread/interview/thread.jpg
title: 多线程高并发面试题（Multithreading & Concurrency）
date: 2017-6-10 19:30:02
toc: true
tags: 
- 多线程
- 高并发
- 面试
categories:
- 多线程
---
线程是java面试问题中的热门话题之一，下面总结了一些Java多线程以及并发访问问题和答案，毕竟多线程和并发性都是并存的。
<!--more-->

# 多线程和并发
## 多线程
### Process和Thread之间的区别
1. 进程是一个独立的运行环境，能够看做是一个程序或者应用，java运行环境运行作为一个简单的包含不同的类和程序的进程集。
2. 而线程可以叫做一个轻量级的进程，线程可以看作是进程中的一个执行任务，线程需要较少的资源来创建并存在于进程中，线程共享进程资源

### 多线程编程的优点
多线程编程可以并发的执行，提高性能，因为某些线程会等待获取某些资源所以cpu不会闲置，多线程共享堆内存，所以创建多线程比创建多进程要好，举个例子就是Servlets的性能要比CGI要好

### 用户线程和守护线程有什么不同
首先都是线程，区别是用户线程dead后，JVM就会退出，不管是否还有守护线程，因为守护线程本来就是守护用户线程的，用户线程都死了，守护线程也没有存在的意义，所以JVM就退出了。还有就是守护线程创建的子线程也是守护线程

### 在java中怎么创建一个线程
1. 通过实现Runnable接口，重写run方法，线程通过New Thread(new 线程类())的方式创建，通过调用start方法启动
2. 通过extend一个Thread类，重写run方法，通过new 线程类()的方式创建，通过调用start方法启动
我们可以直接调用run方法，像普通方法那样，但是这时，线程是没有启动的，只有通过start方法，线程才会启动
如果你的类提供更多的功能，建议实现Runnable接口。毕竟java是多实现，单继承，所以优先Runnable

### 线程的生命周期有哪些
**创建，就绪，运行，阻塞，死亡**这五种方式
1. 当new出线程类时，线程处于创建状态
2. 当调用start方法时，线程处于就绪状态
3. 当run方法执行时，线程处于运行状态
4. 当线程因为某些原因放弃cpu资源时，处于阻塞状态。直到重新进入就绪状态，才有机会再次运行
5. 当线程run方法执行完了，或者运行过程中异常中断了，或者调用了stop方法，就会退出run方法，此时线程就死亡了

### 怎样理解线程的优先级
每个线程都有优先级，通常优先级较高的线程在执行时优先，但这个要取决于操作系统相关的线程调度程序实现。我们可以定义线程的优先级，线程优先级是一个int，从1到10,1的优先级最低，10最高。但不能保证优先级较低的线程之前一定执行优先级较高的线程。

### 什么是线程调度和时间分片
线程调度器是一个操作系统服务，它负责为Runnable状态的线程分配CPU时间。一旦创建一个线程并启动它，它的执行便依赖于线程调度器的实现。时间分片是指将可用的CPU时间分配给可用的Runnable线程的过程。分配CPU时间可以基于线程优先级或者线程等待的时间。线程调度并不受到Java虚拟机控制，所以由应用程序来控制它是更好的选择（也就是说不要让的程序依赖于线程的优先级）

### 如何确保main（）是Java程序中最后完成的线程
我们可以使用Thread join（）方法来确保在完成主函数之前程序创建的所有线程都已经死亡

### 线程间通信方式
主要是通过线程间内存共享，通过类的wait(),notify(),notifyAll()方法进行，这些方法都应该在同步方法或同步块中调用。

### 怎么确保线程安全
1. 同步是最简单也是最广泛的线程安全工具
2. 使用java.util.concurrent.atomic下的Atomic Wrapper类，例如AtomicInteger
3. 使用java.util.concurrent.locks包中的类
4. 使用线程安全的集合类，也在java.util.concurrent中，例如ConcurrentHashMap
5. 将volatile关键字与变量一起使用，使每个线程都从内存中读取数据，而不是从线程缓存中读取数据。

### volatile 关键字
当使用volatile 关键字定义变量时，所有的线程将从主内存中读取而不是从线程的本地缓存读取，这就能确保变量在多线程的情况下也是同步的

### 同步块和同步方法那个更好
更倾向于同步块的写法，因为同步块可以指定minitor对象锁定，可控制粒度更小。而同步方法会锁定整个对象，并且如果类中有多个同步块，即使它们不相关，也会阻止它们执行并将它们置于等待状态

### 创建守护线程的方法
通常创建守护线程用于对系统不重要的功能，例如记录线程或监事线程来捕获系统资源细节和状态，最好避免IO操作的守护线程
可以用Thread.setDaemon(true) 创建

### ThreadLocal是什么
ThreadLocal用于创建线程的局部变量，对象的所有线程共享它的变量，所以这个变量不是线程安全的，可以使用线程同步来达到线程的目的，但是如果想避免同步，就可以使用ThreadLocal变量，每个线程都有自己的ThreadLocal变量互不影响，可以使用get/set方法来设置和获取值

### 什么是死锁，怎么分析和避免死锁
死锁是指两个或以上的线程永远处于阻塞状态。
分析死锁，可以通过查看应用程序的java Thread dump，可以通过jstack工具查看状态为阻塞的线程，然后查看它正在等待的锁定的资源，每个资源都有一个唯一的ID，我们可以使用它找到哪个线程已经在对象上持有锁
有以下准则可以避免死锁
1. 避免嵌套锁定，这是大部分死锁的情况，也就是说尽量不要在锁定资源a的情况下，又去锁定资源b
2. 只锁定需要锁定的东西，你应该只获取必须处理的资源的锁，如果我们只对某一个字段感兴趣，那么我们应该只锁定该特定字段而不是完整对象
3. 避免无限等待，如果线程a必须等待线程b完成，尽量不要用sleep去控制，而是使用threa.join串行执行

### 什么是线程池，如何创建线程池
根据系统自身的环境情况，有效的限制执行线程的数量，使得运行效果达到最佳。线程主要是通过控制执行的线程的数量，超出数量的线程排队等候，等待有任务执行完毕，再从队列最前面取出任务执行。
创建线程池的方式有多种
1. java.util.concurrent.Executors 提供了线程池的静态实现方法,但一般不推荐这种写法
- newFixedThreadPool();创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue；
- newSingleThreadExecutor();将corePoolSize和maximumPoolSize都设置为1，也使用的LinkedBlockingQueue；
- newCachedThreadPool();将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。
- newScheduledThreadPool(); 用于创建一个线程池，线程池中得线程能够周期性地执行给定的任务
2. ThreadPoolExecutor类提供了更完善的线程池创建构造方法
3. ScheduledExecutorService 类提供了定期执行任务线程池

## 并发
### 什么是原子操作，java并发api中的原子类是什么
原子操作在一个任务单元中执行，不受其他操作的干扰。原子操作在多线程环境中是必需的，以避免数据不一致。
比如int++ 就不是一个原子操作，因为再多线程的情况下，某个线程执行了加1，但其他线程可能读的还是旧的值，就会导致错误的结果。
为了解决这个问题，我们必须保证count的增量操作是院子的，我们可以使用同步来达到目的，但java1.5以后在java.util.concurrent.atomic提供了int和long的包装类，可以用来实现这个原子操作，没有使用同步，有兴趣的可以深入了解实现

### java并发api中的Lock接口是什么？与synchronize相比，它有什么好处？
Lock 借口提供了更多广泛的锁定操作比使用synchronized，Lock结构更加灵活，可以有完全不同的属性，并且可以关联多个条件对象
Lock有以下优点
1. 可以让线程更公平
2. 可以使线程在等待一个锁定对象时响应中断
3. 可以去尝试获取锁定，但如果无法获取锁定时，则会立即返回或在超时后返回
4. 可以以不同的顺序获取或释放不同范围内的lock

### 谈谈Excutor
Excutor 是在jdk 1.5 引入的，通过java.util.concurrent.Executor接口
Excutor 主要是根据一组执行策略规范调用，调度，执行和控制异步任务
创建多个线程并且没有达到最大阈值的限制会导致应用程序耗尽堆内存，所以创建线程池是一个比较好的解决方案，应为有限的线程可以被集中和重用，而Excutor就是为了更好的创建线程池设计的

### 什么是BlockingQueue，怎么通过BlockingQueue实现一个生产者消费者模型
java.util.concurrent.BlockingQueue 是一个阻塞队列，阻塞必然有两种情况，
1. 当队列满了的时候，进行入列操作会被阻塞
2. 当队列空的时候，出列操作会被阻塞
阻塞队列是线程安全的，所有排队方法本质上都是原子性的，使用内部锁或其他形式的并发控制
阻塞队列主要也是用于生产者消费者问题，负责生产的线程不断的制造新对象并插入到阻塞队列中，直到达到这个队列的上限值。队列达到上限值之后生产线程将会被阻塞，直到消费的线程对这个队列进行消费。同理，负责消费的线程不断的从队列中消费对象，直到这个队列为空，当队列为空时，消费线程将会被阻塞，除非队列中有新的对象被插入。

### 谈谈Callable和Future
Callable相当于Runnable的一个扩展，不同于Runnable的是Callable是个泛型参数化接口，并能返回线程的执行结果，而且能在无法正常计算时抛出异常。
Callable并不像Runnable那样通过Thread的start方法就能启动实现类的run方法，通常是利用ExecutorService的submit方法去启动call方法自执行任务，而ExecutorService的submit又可以返回一个Future类型的结果，因此Callable通常也与Future一起使用，还有一种方式是使用FutureTask封装Callable再由Thread去启动。
所以Callable的好处是异步执行，还能返回结果，结合Future还能判断任务状态，取消任务

### 谈谈FutureTask
FutureTask是Future接口基类的实现类，可以和Executors一起用于异步处理，大多数情况下很少使用FutureTask类，但如果我们想要覆盖Future类的某些方法，并且保留基本实现，它就变得非常方便。我们可以扩展这个类，根据需求覆盖一些方法

### 谈谈Concurrent Collection类
通常Collection类是快速失败的，这意味着当一个线程在使用iterator便利时，去修改集合，这个iterator.next()操作将抛出ConcurrentModificationException异常。而Concurrent Collection则不会出现这个问题，因为它就是为多线程设计的
主要的类包括 ConcurrentHashMap, CopyOnWriteArrayList 和 CopyOnWriteArraySet

### 讲讲Executors类
Executors 提供了很多静态使用方法，包括Executor, ExecutorService, ScheduledExecutorService, ThreadFactory, 以及 Callable，所以可以使用executors类在java中轻松创建线程池，这也是唯一支持可调用实现执行的类。

### java8中并发改进了哪些？
重要的改进包括：
1. ConcurrentHashMap 的compute(), forEach(), forEachEntry(), forEachKey(), forEachValue(), merge(), reduce() 和 search() 等方法
2. 加入了CompletableFuture ，使异步编程更优美
3. Executors 新增了 newWorkStealingPool 线程池方法






