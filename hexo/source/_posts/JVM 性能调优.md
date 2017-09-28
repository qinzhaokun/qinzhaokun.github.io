---
title: JVM 性能调优
date: 2017-09-28
tags: Java
categories: Java
---

## Full GC

GC 调优对高并发大数据量交互的应用还是很有必要的，尤其是默认 JVM 参数通常不满足业务需求，需要进行专门调优。由于系统堆设置较大，Full GC 一次暂停应用时间会较长，这对线上实时服务影响较大

