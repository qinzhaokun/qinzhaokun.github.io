---
title: Java线程池原理
date: 2017-09-11
tags: Java
categories: Java
---

首先，我们来看看，Java线程池怎么写：
```
ExecutorService es = Executors.newFixedThreadPool(15);

es.submit(() -> {
  System.out.println("I am a new Thread!");
});

es.shutdown();

```
线程池的类型是`ExcutorService`, 通过`Excutors`类的静态方法`newFixedThreadPool`创建一个线程池，并使用`submit`方法提交一个Runable任务。

那么使用线程池，有什么好处呢？

1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗

2. 提高响应速度。当任务到达时，任务可以不需要等到线程创建就立即执行

3. 提高线程的可管理性

## ThreadPoolExecutor类

它是线程池中最核心的一个类，提供了四种*构造函数*：

```
public class ThreadPoolExecutor extends AbstractExecutorService {
......
    public ThreadPoolExecutor(int corePoolSize,
                                  int maximumPoolSize,
                                  long keepAliveTime,
                                  TimeUnit unit,
                                  BlockingQueue<Runnable> workQueue) {
            this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
                Executors.defaultThreadFactory(), defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize,
                                  int maximumPoolSize,
                                  long keepAliveTime,
                                  TimeUnit unit,
                                  BlockingQueue<Runnable> workQueue,
                                  ThreadFactory threadFactory) {
            this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
                 threadFactory, defaultHandler);
    }

    public ThreadPoolExecutor(int corePoolSize,
                                  int maximumPoolSize,
                                  long keepAliveTime,
                                  TimeUnit unit,
                                  BlockingQueue<Runnable> workQueue,
                                  RejectedExecutionHandler handler) {
            this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
                 Executors.defaultThreadFactory(), handler);
    }

    public ThreadPoolExecutor(int corePoolSize,
                                  int maximumPoolSize,
                                  long keepAliveTime,
                                  TimeUnit unit,
                                  BlockingQueue<Runnable> workQueue,
                                  ThreadFactory threadFactory,
                                  RejectedExecutionHandler handler) {
            if (corePoolSize < 0 ||
                maximumPoolSize <= 0 ||
                maximumPoolSize < corePoolSize ||
                keepAliveTime < 0)
                throw new IllegalArgumentException();
            if (workQueue == null || threadFactory == null || handler == null)
                throw new NullPointerException();
            this.corePoolSize = corePoolSize;
            this.maximumPoolSize = maximumPoolSize;
            this.workQueue = workQueue;
            this.keepAliveTime = unit.toNanos(keepAliveTime);
            this.threadFactory = threadFactory;
            this.handler = handler;
    }
```

以下是几个关键参数的定义：

+ `corePoolSize`: 核心池的大小。一开始时线程池里没有任何线程，线程池的大小为0，当不断有任务来时，会逐渐创建新的线程，当线程的数量到达`corePoolSize`时，新来的任务就会放入缓存队列当中。

+ `maximumPoolSize`: 表示线程池中最多的线程数。

+ `keepAliveTime`: 表示线程没有任何执行时最多能存活多久。一般的，只有当线程池中的线程数大于`corePoolSize`时，`keepAliveTime`才会起作用.即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。

+ `uint`: keepAliveTime的时间单位，有7种取值：

> TimeUnit.DAYS; // 天
> TimeUnit.HOURS; // 时
> TimeUnit.MINUTES; // 分
> TimeUnit.SECONDS; // 秒
> TimeUnit.MILLISECONDS; // 毫秒
> TimeUnit.MICROSECONDS; // 微妙
> TimeUnit.NANOSECONDS; // 纳秒

+ `workQueue` ： 一个阻塞队列，用来存储等待执行的任务,它线程池中运行过起着关键的作用，它有几种选择：

> ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序；

> LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列；

> SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于

> LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列；

> PriorityBlockingQueue：一个具有优先级的无限阻塞队列；

+ `threadFactory` ：线程工厂，主要用于创建线程

+ `handler` ：饱和策略，即当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是`AbortPolicy`，表示无法处理新任务时抛出异常

> ThreadPoolExecutor.AbortPolicy： 丢弃任务并抛出RejectedExecutionException异常；

> ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常;

> ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）；

> ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务;

## 继承关系

`ThreadPoolExecutor`类 ---> `AbstractExectorService`类 ---> `ExecutorSerivce`接口 ---> `Executor`接口

1. `Executor`是一个顶层接口，在它里面只声明了一个方法`execute(Runnable)`，返回值为`void`，参数为`Runnable`类型，从字面意思可以理解，就是用来执行传进去的任务的；

然后`ExecutorService`接口继承了`Executor`接口，并声明了一些方法：`submit`、`invokeAll`、`invokeAny`以及`shutDown`等；

抽象类`AbstractExecutorService`实现了`ExecutorService`接口，基本实现了`ExecutorServic`e中声明的所有方法；

然后`ThreadPoolExecutor`继承了类`AbstractExecutorService`

`submit`和`execute`方法的区别：

`execute`是顶层接口`Executor`的唯一方法，它在`ThreadPoolExecutor`得到了具体实现，通过这个方法可以向线程池提交一个任务，交由线程池去执行。但是它的返回值是`void`

`submit`是`ExecutorService`申明的，在AbstractExecutorService就已经有了具体的实现，在`ThreadPoolExecutor`中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看`submit()`方法的实现，会发现它实际上还是调用的`execute()`方法，只不过它利用了`Future`来获取任务执行结果。

这里感觉由有点模板模式的味道了。

`shutdown()`和`shutdownNow()`是用来关闭线程池的。后者会尝试终止正在运行的task。

getQueue() 、getPoolSize() 、getActiveCount()、getCompletedTaskCount()等获取与线程池相关属性的方法

## 运行原理

重点来了，线程池是如何运行的呢？

### 线程池的状态

> static final int RUNNING = 0;
> static final int SHUTDOWN = 1;
> static final int STOP = 2;
> static final int TERMINATED = 3;

当创建线程池后，初始时，线程池处于`RUNNING`状态；

如果调用了`shutdown()`方法，则线程池处于`SHUTDOWN`状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；

如果调用了`shutdownNow()`方法，则线程池处于`STOP`状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；

当线程池处于`SHUTDOWN`或`STOP`状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为`TERMINATED`状态。

### 线程池中的成员变量
```
private final BlockingQueue<Runnable> workQueue; // 任务缓存队列，用来存放等待执行的任务

private final ReentrantLock mainLock = new ReentrantLock(); // 线程池的主要状态锁，对线程池的状态（比如线程池的大小，状态等）的改变都要使用这个锁

private final HashSet<Worker> workers = new HashSet<Worker>(); // 用来存放工作集

private volatile long keepAliveTime; //线程存活时间

private volatile boolean allowCoreThreadTimeOut; // 是否允许为核心线程设置存活时间

private volatile int corePoolSize; // 核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）

private volatile int maximumPoolSize; // 线程池最大能容忍的线程数

private volatile int poolSize; // 线程池中当前的线程数

private volatile RejectedExecutionHandler handler; // 任务拒绝策略

private volatile ThreadFactory threadFactory; // 线程工厂，用来创建线程

private int largestPoolSize; // 用来记录线程池中曾经出现过的最大线程数

private long completedTaskCount; //用来记录已经执行完毕的任务个数

```
基本上对应构造函数中的参数。

### 工作流程

一开始没有任何线程，当不断有新的任务来时，如果此时的线程数没有到达corePoolSize，则会创建新的线程；随着任务的不断到来，线程数到达了corePoolSize，新的任务会进入阻塞队列，等待被赋给某个线程执行。注意阻塞队列有两种取数据的方式，一种是`take()`,阻塞取；另一种是`poll()`有timeout的阻塞取，keepAlivetime可根据是够在有限时间内取到任务二判断应不应该删除某些线程。当阻塞队列满了之后，新来的任务又会让线程池生成新的线程，直到线程数到达maximum后，当还有任务进来时，则按照饱和策略对新的任务进行处理。
