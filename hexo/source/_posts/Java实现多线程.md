---
title: Java实现多线程
date: 2017-08-03
tags: Java
categories: Java
---

多线程是Java重要的一块基础知识，各种面试中也是考察的重点，需要非常牢固的掌握。多线程的目的是**更好的利用CPU资源**

## 线程状态回顾

首先回顾一下线程的5种状态：

+ **NEW** 线程初始化，还没调用`run()`之前的状态

+ **RUNABLE** 可运行态，其中包括**RUNNING**和**READY**

+ **WAIT** 等待状态，一般都有线程Thread类的静态方法`sleep()`或`join()`

+ **TIMING WAIT** 可中断等待

+ **BLOKCED** 阻塞状态，线程尝试获取对象锁未成功，则进入阻塞，或者已经持有锁的情况下，调用了`Object`的`wait()`

+ **DEAD** 线程运行结束

## 线程实现的方法

### Thread类

这个是最基本的实现多线程的方式，**继承Thead类**，并且重写`run()`方法，如下：
```
/**
 * Created by q00416758 on 2017/6/8.
 */

class MyThread extends Thread{
	MyThread(ThreadGroup tp, String name){
		super(tp,name);
	}
    public void run(){
        for(int i = 0;i < 10;i++){
            try{
                System.out.println(this.getName()+":" + i);
                Thread.sleep(500);
            }catch (Exception e){
                System.out.println(e);
            }
        }
    }
}
public class Main {

    public static void main(String [] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
        t1.start();
        t2.start();

    }
}

```
注意，`Thread.sleep()`要使用`try catch`处理异常。

#### ThreadGroup线程组

ThreadGroup是为了方便对线程进行管理，例如设定线程的属性，setDaemon,异常处理方法，统一安全策略等，具体的使用如下：

```
ThreadGroup tp = new ThreadGroup("t");
MyThread t1 = new MyThread(tp,"t1");
MyThread t2 = new MyThread(tp,"t2");
```
线程一旦归入某个组，就无法更换组
补充，线程在逻辑上是一颗树，root是system线程组，线程组和包含另一个线程组。最简单的在main()函数生成一个线程，那么这个线程属于main线程组，查看一个线程所属的线程组，可使用如下方法：
```
Thread.currentThread().getThreadGroup();
```
此外，线程组还有一些实用的功能, `activeCount()`获取活动线程的数量和`enumerate()`传入线程数组，传出所有活动线程引用
```
Thread[] threads = new Thread[threadGroup1.activeCount()];
threadGroup1.enumerate(threads);
```

### Runable接口

继承一个Thread类来实现定义线程比较直观，但是如果某个线程有自己的父类，那么由于单一继承的原则，不能在继承Thread，这是，可以考虑使用Runable实现。实现Runable接口仅仅只是**定义任务**的过程，它本身不具有线程功能，`run()`也是不同方法，必须把任务附着在Thread上。

```
class MyTask implements Runnable{
	
	private int count = 0;
	private String name;
	MyTask(String name){
		this.name = name;
	}

	@Override
	public void run() {
		for(int i = 0;i < 10;i++){
			try{
				System.out.println("Thread"+name+":" + i);
				Random r = new Random();
				Thread.sleep(r.nextInt(1000));
			}catch (Exception e){
				System.out.println(e);
			}
		}
	}
  
  public class Hello {
	public static void main(String[] args) {
		  Thread t3 = new Thread(new MyTask("1"),"1");
		  Thread t4 = new Thread(new MyTask("2"),"2");
		  t3.start();
		  t4.start();
	}
}
	
```

#### Runnable对比Thread的优势

+ 适合共享变量，由于Runnable只是一个task，可以将一个task传入不同的Thread；不过我认Thread也不是不能共享变量。

+ 突破单继承的限制

+ 访问当前线程方法，使用Thread.currentThread()得到线程对象；而Thread直接使用this即可

### Callable/Future/FutureTask

#### Callable
Java.util.concurrent下的。 Runable线程的执行是没有返回值的，而Callable允许有返回值。首先来看一下Callable的定义：
```
public interface Callable<V> {
  V call() throws Exception;
}
```
实现Callable接口的类是有返回值的任务，和之前一样，它不具有多线程的功能。要让其实现多线程，**只能使用线程池**。其中，**ExcutorService**接口是线程池的接口，它提供了`submit`方法
```
<T> Future<T> submit(Callable<T> task);  
<T> Future<T> submit(Runnable task, T result);  
Future<?> submit(Runnable task);

```
#### Future

**Future<V>**接口可用于**获取异步计算结果**，当使用`submit`提交任务后， 会返回一个`Future<V>`。它主要提供了一下三个功能：

1. 能够中断执行中的任务, `cancel(true)`

2. 判断任务是否执行完成, `isDone()`

3. 获取任务执行完成后额结果, `V get()` 如果任务没没有完成，阻塞到完成； `V get(Long timeout , TimeUnit unit)`阻塞指定时间

代码示例：
```
class CallableTask implements Callable<Integer>{
	
	public int count;

	@Override
	public Integer call() throws Exception {
		for(int i = 0;i < 20;i++){
			count++;
			Thread.sleep(1000);
		}
		return count;
	}
}

public class Hello {
	public static void main(String[] args) {
		CallableTask c = new CallableTask();
		ExecutorService ex = Executors.newSingleThreadExecutor();
		Future<Integer> f =ex.submit(c); 
		while(!f.isDone()){
			System.out.println(c.count);
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
    ex.shutdown(); 
	}


}
```

#### FutureTask

上面实现多线程的方式虽然功能挺全的，但是比较繁琐，分成了Callable和Future两部分，FutureTask除了实现了Future接口外还实现了Runnable接口，因此FutureTask也可以直接提交给Executor执行。下面来总结一下FutureTask的特性：

1. 传入Runnable或者Callable给FutureTask

2. 相比于Funture+Callable+ThreadPool的组合，FutureTask具有线程功能，可直接调用`run()`方法，并且**多次run方法，它都只会执行一次Runnable或者Callable任务**

3. 也可提交线程池执行

4. 可通过`cancel()`取消执行

*适用场景*：提交异步任务，主线程执行其他任务，当需要执行结果时，异步获取子线程的执行结果

*应用*：FuntureTask的一个重要应用就是保证线程只会被执行一次，在高并发的情况下，能够有效的保证线程安全的同时，也能很好的保证效率。一个案例就是基于`ConcurrentHashMap`和`FutureTask`保证在高并发的情况下对象被创建一次。一个带key的连接池，当key存在时，即直接返回key对应的对象；当key不存在时，则创建连接。

```
private ConcurrentHashMap<String,FutureTask<Connection>>connectionPool = new ConcurrentHashMap<String, FutureTask<Connection>>();  
  
public Connection getConnection(String key) throws Exception{  
    FutureTask<Connection>connectionTask=connectionPool.get(key);  
    if(connectionTask!=null){  
        return connectionTask.get();  
    }  
    else{  
        Callable<Connection> callable = new Callable<Connection>(){  
            @Override  
            public Connection call() throws Exception {  
                // TODO Auto-generated method stub  
                return createConnection();  
            }  
        };  
        FutureTask<Connection>newTask = new FutureTask<Connection>(callable);  
        connectionTask = connectionPool.putIfAbsent(key, newTask);  
        if(connectionTask==null){  
            connectionTask = newTask;  
            connectionTask.run();  
        }  
        return connectionTask.get();  
    }  
}  
  
//创建Connection  
private Connection createConnection(){  
    return null;  
}  
```
