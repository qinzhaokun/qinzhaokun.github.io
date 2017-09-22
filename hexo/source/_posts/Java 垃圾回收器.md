---
title: Java 垃圾回收器
date: 2017-09-22
tags: Java
categories: Java
---

本篇讲解垃圾回收的具体实现。由于Java虚拟机规范中对垃圾回收器应该如何实现没有明确的规定，因此，不同的厂商都会有自己的实现。

## 并发 && 并行

**并行**是Parallel, 是指多条垃圾回收线程并行的工作，这时用户的业务线程是被挂起的，造成了`Stop the world`

**并发**是Concurrent, 用户的线程和垃圾回收线程并发执行，共同抢占CPU时间。一般意义来说,不会造成`Stop the world`

## Serial收集器

串行垃圾回收收集器，最基本，最简单，历史最悠久的收集器

http://www.jianshu.com/p/50d5c88b272d
