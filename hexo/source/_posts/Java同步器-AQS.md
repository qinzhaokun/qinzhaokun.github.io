---
title: Java同步器-AQS.md
date: 2017-08-17
tags: Java
categories: Java
---

同步器AQS，全称是AbstractQueuedSynchronizer。框架是构建concurrent包下很多工具类的基础，其中包括，Lock,CountDownLatch, CycliBarrier等都需要依赖AQS。 同步中的ReentrantLock中都是依靠AQS实现同步的（它有Lock实例，而Lock实例持有Sync实现例，这个Sync就是继承自AbstractQueuedSynchronizer）。AQS只是一个抽象类，提供了接口，没有任何实现，具体保证同步的代码由其子类实现，如公平锁` FairSync` 和非公平锁`NonfairSyn`。

它为不同场景提供了实现锁及同步机制的基本框架，为同步状态的原子性管理、线程的阻塞、线程的解除阻塞及排队管理提供了一种通用的机制。

## 基本框架

### 数据成员

AQS的数据成员主要有两个：

1. state，同步状态，有`volatile`语义。32位的整型数据，对他的操作保证了**原子性**。

2. CHL Node的 FIFO的队列。它将线程封装到了Node里面，并封装了**阻塞线程和解除阻塞的操作**。这个队列的本质是双向链表。Node的插入和移除都是要保证原子性的，使用的是`CAS`来操作。

### 成员函数

```
protected boolean tryAcquire(int arg) {    
    throw new UnsupportedOperationException();    
}    
protected boolean tryRelease(int arg) {    
    throw new UnsupportedOperationException();    
}    
protected int tryAcquireShared(int arg) {    
    throw new UnsupportedOperationException();    
    
}    
protected boolean tryReleaseShared(int arg) {    
    throw new UnsupportedOperationException();    
}  

```
以上方法都是没有实现的，AQS框架留给子类去实现，它提供了类似aquire(),release()等方法，这些方法只是规定了基本流程，如`aquire()`规定了获取独占锁的步骤，方法里仍然依赖子类的实现：
```
    public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```
重点来了！！，把具体的实现留给子类，父类定义基本算法流程，这是设计模式中典型的**模板模式**，**模板模式**，**模板模式**。AQS中使用了大量的模板模式。

和一些私有方法：
```
final boolean acquireQueued(final Node node, int arg) ：申请队列
private Node enq(final Node node) : 入队
private Node addWaiter(Node mode) ：以mode创建创建节点，并加入到队列
private void unparkSuccessor(Node node) ： 唤醒节点的后续节点，如果存在的话。
private void doReleaseShared() ：释放共享锁
private void setHeadAndPropagate(Node node, int propagate)：设置头，并且如果是共享模式且propagate大于0，则唤醒后续节点。
private void cancelAcquire(Node node) ：取消正在获取的节点
private static void selfInterrupt() ：自我中断
private final boolean parkAndCheckInterrupt() ： park 并判断线程是否中断
```

### 独占和共享

AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。对于独占模式，AQS给出了基本的几种操作的流程：

+ `acpuire()` 获取锁，失败就阻塞

+ `acquireInterruptibly(int arg)` 获取锁，可中断

+ `tryAcquireNanos(int arg, long nanosTimeout)` 带有超时的阻塞模式获取锁

+ `release()` 释放锁

共享模式也定了几种基本操作：

+ `acquireShared(int arg)` 获取锁，失败就阻塞

+ `acquireSharedInterruptibly(int arg)` 可中断获取锁

+ `tryAcquireSharedNanos(int arg, long nanosTimeout)` 超时模式的获取锁

+ `releaseShared(int arg)` 释放共享锁

### 小结

由上可知，作为一个框架，AQS制定了一些基本流程，但具体的实现会不同，因此，它使用了模板模式把一些细节的实现留给子类，它基本包含三个功能：

1. 同步器基本范式、结构

2. 线程的阻塞、唤醒机制

3. 线程阻塞队列的维护

## 简单工作流程

### 获取同步状态

当线程1需要获取同步状态，首先检查state，如果等于0，即状态可用，可以顺利获取锁（`tryAcquire`），并将state置为1；否则，锁被占用，将线程封装金Node中，插入同步队列，使用`LockSupport.park()`阻塞线程。

### 释放同步状态

当持有同步状态的线程1释放锁时，将state置为0，唤醒头节点的后继节点，即调用`LockSupport.unpark()`方法，它重新执行获同步状态的步骤，如果成功就出列。

以上只是比较粗的流程，其中还有非常多的细节需要考虑。

## CAS操作

AQS框架中使用了CAS保证了操作的原子性，**LockSupport.park() 和 LockSupport.unpark() 实现线程的阻塞和唤醒就是通过CAS的**；**更新同步状态state也是通过CAS的**。那么，这个CAS底层是怎么实现的呢？

下面看一下更新state状态的源码：
```
     protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
     }
```
它是调用`unsafe`类来实现的。Unsafe是一个很强大的类，它可以分配内存、释放内存、可以定位对象某字段的位置、可以修改对象的字段值、可以使线程挂起、使线程恢复、可进行硬件级别原子的CAS操作等等。它里面的方法也是native方法，是C++代码实现，在`hotspot\src\share\vm\prims\unsafe.cpp`中。先线程相关的操作park()和unpark()的最终实现都和操作系统相关，比如windows下实现是在os_windows.cpp中。

由上可知，更新同步状态`state`需要使用`CAS`操作，插入节点到队列尾部也需要`CAS`操作，`unsafe.compareAndSwapObject`
