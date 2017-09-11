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
