---
title: Java 阻塞队列
date: 2017-08-21
tags: Java
categories: Java
---

BlockingQueue是一个高性能的容器，它被用来线程之间共享数据，它是典型的生产者和消费者的实现。它支持两个附加的操作：读数据时等待队列变成非空；写数据时等待队列变成可写。JDK7提供了阻塞队列，它们分别是：

+ ArrayBlockingQueue 由数组组成的有界阻塞队列，必须指定队列的大小，遵循FIFO原则

+ LinkedBlockingQueue 由链表结构组成的有界阻塞队列，可改变大小的阻塞队列，遵循FIFO原则

+ PriorityBlockingQueue 支持优先级排序的无界阻塞阻塞队列，不是FIFO，而是有优先级的

+ DelayQueue 一个使用优先级队列实现的无界阻塞队列

+ SynchronousQueue：一个不存储元素的阻塞队列。每一插入必须等待另一个线程移除。

+LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

+LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

它定义了一种方法：

+ `add(o)`,向队列插入一个元素，如果该操作无法立即执行，则**抛出异常**

+ `offer(o)`,向队列插入一个元素，如果该操作无法立即执行，**返回一个特定的值**，ture/false

+ `put(o)`, 向队列插入一个元素，如果该操作无法立即执行，**线程阻塞直到执行完毕后返回**

+ `remove(o)`,删除队列一个元素，如果该操作无法立即执行，则**抛出一个异常**

+ `poll(o)`, 删除队列一个元素，如果该操作无法立即执行，则**返回一个特性的值**，true/false

+ `take(o)`, 删除队列一个元素，如果该操作无法立即执行，**线程阻塞直到执行完毕后返回**

注意，由于是队列操作，所以移除操作在非队首时，效率不高，建议不要这么做。

## 代码示例

下面使用`ArrayBlockingQueue`演示简单的生产者和消费者模型：
```
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;


class Producer implements Runnable{
	private BlockingQueue bq;
	
	Producer(BlockingQueue bq){
		this.bq = bq;
	}

	@Override
	public void run() {
		while(!Thread.currentThread().isInterrupted()){
			try {
				Integer t = new Random().nextInt(100);
				bq.put(t);
				System.out.println("put a " + t + " by " + Thread.currentThread().getName());
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
	}
	
}

class Customer implements Runnable{
	private BlockingQueue bq;
	
	Customer(BlockingQueue bq){
		this.bq = bq;
	}

	@Override
	public void run() {
		while(!Thread.currentThread().isInterrupted()){
			try {
				Integer t = (Integer)bq.take();
				System.out.println("get a " + t + " by " + Thread.currentThread().getName());
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
	}
	
}

public class Test {

	public static void main(String[] args) throws InterruptedException {
		ExecutorService es= Executors.newFixedThreadPool(10); 

		BlockingQueue bq = new ArrayBlockingQueue(4);
		for(int i = 0; i < 5;i++){
			es.submit(new Producer(bq));
			es.submit(new Customer(bq));
		}
		
		es.shutdown();
	}

}

```

## ArrayBlockingQueue原理分析

阻塞队列的原理很简单，就是使用了ReentrantLock+Condition来实现线程的同步互斥，每次只有一个线程能够操作目标数组/链表;其他线程则阻塞等待Conditionde singal，下面是其关键的三个函数：

### 构造函数
```
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {

    private static final long serialVersionUID = -817911632652898426L;
    /** The queued items */
    final Object[] items;
    /** items index for next take, poll, peek or remove */
    int takeIndex;
    /** items index for next put, offer, or add */
    int putIndex;
    /** Number of elements in the queue */
    int count;
    final ReentrantLock lock;
    /** Condition for waiting takes */
    private final Condition notEmpty;
    /** Condition for waiting puts */
    private final Condition notFull;
 ...省略
 }

```

### put函数

```
public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
```

### enqueue函数

```
/**
     * Inserts element at current put position, advances, and signals.
     * Call only when holding lock.
     */
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length) putIndex = 0;
        count++;
        notEmpty.signal();
    }
```
### take函数

```
public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

```

### dequeue函数

```
private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length) takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return x;
    }
```

由上可知，基于数组的阻塞队列的实现其实很简单，主要利用了可重入锁保证互斥的操作目标数组，当数组full的时候阻塞写线程await，并等待读进程的唤醒signal;同样的，当数组为空时，阻塞读线程，并等待写线程的唤醒。
