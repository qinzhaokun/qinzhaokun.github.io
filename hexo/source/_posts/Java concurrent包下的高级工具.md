---
title: Java concurrent包下的高级工具
date: 2017-08-16
tags: Java
categories: Java
---

Java1.5中的concurrent包下提供了很多的工具类，本次来讲述这些实用的类。

## CountDownLatch

它主要用来管理一组相关的线程，使得某个线程，要等待其他所有线程执行完后在执行。常用的有两个方法：`countdown()`和`await()`.它的用法很简单，可分为以下三步：

1. `new CountDownLatch(10)`，设置需要等待的线程数量，这是设置为10。

2. `CountDownLatch.await()`, 调用该方法的线程立刻阻塞，等待需要等待的线程数量为0后重新变成可运行态。

3. `CountDownLath.countdown()`,使得该CountDownLatch需要等待的线程数量减1。

### 代码示例：

```
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class Task implements Runnable{
	
	private CountDownLatch cdl;
	
	private CountDownLatch start;
	
	Task(CountDownLatch cdl, CountDownLatch start){
		this.cdl = cdl;
		this.start = start;
	}

	@Override
	public void run() {
		try {
			start.await();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName());
		cdl.countDown();
	}
	
}

public class Test {

	public static void main(String[] args) throws InterruptedException {
		ExecutorService es= Executors.newFixedThreadPool(10); 
		
		CountDownLatch cld = new CountDownLatch(100);
		CountDownLatch start = new CountDownLatch(1);
		
		for(int i = 0; i < 100; i++){
			es.submit(new Task(cld,start));
		}
		start.countDown();
		cld.await();
		System.out.println("over");
		es.shutdown();
	}

}
```
### 实现原理

它是自定义的**AQS**组件,之前对AQS同步框架已有详细介绍，这里就不在虚岁。有了AQS框架的实现子类`sync`的实例，`CountDownLatch`就变得异常简单，它就是一个简单的共享锁的应用。下面是它三个主要方法的源码：
```
//设置最多多少个线程共享一把锁
public CountDownLatch(int count){
	if(count < 0) throw new IllegalArgumentExceptoin("count < 0");
	this.sync = new Sync(count);
}

//释放共享锁
public void countDown() {
	sync.releaseShared(1);
}

//获取共享锁
public void await() throws InterruptedException {
	sync.accquireSharedInterruptibly(1);
}


```

由上可知，`CountDownLatch`整个过程就是设置共享锁的数量，然后不断释放锁的过程。

## CyclicBarrier

这个类的作用是**允许一组线程相互等待，直到到达某个公共的屏障点，常用的情景是一组线程，它们不时的互相等待，而且，它是可以重用的**，就像它的名字一样，它叫栅栏，要等到所有线程都执行完了，在一起继续做其他事。

那么，一定觉得`CountDownLatch`和`CyclicBarrier`很像吧。它们之间的区别有：

1. 后者可以重复使用，而前者不行

2. 后者是**N个线程相互等待**，然后一起继续执行；而前者在于**一个线程等待N个线程**，N个线程执行完后可以继续执行，而等待的线程需要在N个线程执行完`countDown`后再执行。

### 代码示例

下面来看看它的具体使用：
```
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class CyclicBarrierTask implements Runnable{
	
	private CyclicBarrier barrier;
	
	CyclicBarrierTask(CyclicBarrier barrier){
		this.barrier = barrier;
	}
	@Override
	public void run() {
		try {
			Thread.sleep(new Random().nextInt(10000));
			System.out.println(Thread.currentThread().getName() + " arrives");
			barrier.await();
			System.out.println(Thread.currentThread().getName() + " go on");
			
		} catch (InterruptedException | BrokenBarrierException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
	
}

public class Test {

	public static void main(String[] args) throws InterruptedException {
		ExecutorService es= Executors.newFixedThreadPool(10); 
		
		CyclicBarrier cb = new CyclicBarrier(4);
		for(int i = 0;i < 4;i++)
			es.submit(new CyclicBarrierTask(cb));
		//System.out.println("over");
		es.shutdown();
	}

}

```
以上，四个线程分别随机睡眠一段时间，等到睡醒后再等到其他所有线程都醒来后再一起执行。

### 原理分析

`CyclicBarrier`的实现不是直接基于AQS的，而是借助了`ReentrantLock`,源码在这里就不贴，调用`await`方法，会转而执行`dowait()`，在函数里，使用了lock,Condition,和一个count计数变量，主要实现的功能如下：

1. 使用lock，保证多线程对CyclicBarrier的内部数据操作都是互斥的

2. 当该线程是最后一个到达的，唤醒其他所有线程，并继续执行下去

3. 当该线程不是最后一个线程到达时，进入Condition自旋等待。
