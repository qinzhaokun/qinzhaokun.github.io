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

这个类的作用是**允许一组线程相互等待，直到到达某个公共的屏障点，常用的情景是一组线程，它们不时的互相等待，而且，它是可以重用的**
