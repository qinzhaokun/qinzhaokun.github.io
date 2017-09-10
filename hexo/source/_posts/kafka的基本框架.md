---
title: Kafka基本框架
date: 2017-09-10 18:54:25
tags: Kafka
categories: Kafka
---

Kafka是由LinkedIn开发的一个分布式的消息系统，使用Scala编写，它以可水平扩展和高吞吐率而被广泛使用。目前越来越多的开源分布式处理系统如Cloudera、Apache Storm、Spark都支持与Kafka集成。

## 背景介绍

### 创建背景

Kafka是一个消息系统，原本开发自LinkedIn，用作LinkedIn的活动流（Activity Stream）和运营数据处理管道（Pipeline）的基础。现在它已被多家不同类型的公司作为多种类型的数据管道和消息系统使用。

活动流数据是几乎所有站点在对其网站使用情况做报表时都要用到的数据中最常规的部分。活动数据包括页面访问量（Page View）、被查看内容方面的信息以及搜索情况等内容。这种数据通常的处理方式是先把各种活动以日志的形式写入某种文件，然后周期性地对这些文件进行统计分析。运营数据指的是服务器的性能数据（CPU、IO使用率、请求时间、服务日志等等数据)。运营数据的统计方法种类繁多。

## 简介
Kafka是一种分布式的，基于发布/订阅的消息系统。作为一个分布式的消息系统，其主要设计目标如下：

+ 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间复杂度的访问性能。

+ 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条以上消息的传输。支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消息顺序传输。同时支持离线数据处理和实时数据处理。

+ Scale out：支持在线水平扩展。

为何使用消息系统?

### 解耦

在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

### 冗余

有些情况下，处理数据的过程会失败。除非数据被持久化，否则将造成丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

### 扩展性

因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。

### 灵活性 & 峰值处理能力

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

### 可恢复性

系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

### 顺序保证
在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。Kafka保证一个Partition内的消息的有序性。

### 缓冲

在任何重要的系统中，都会有需要不同的处理时间的元素。例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。

### 异步通信

很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

## 架构

+ Broker， Kafka集群包含一个或多个服务器，这种服务器被称为broker

+ Topic，每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

+ Partition，Parition是物理上的概念，每个Topic包含一个或多个Partition.

+ Producer，负责发布消息到Kafka broker

+ Consumer，消息消费者，向Kafka broker读取消息的客户端。

+ Consumer, Group，每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）

+ replica, partition 的副本，保障 partition 的高可用(HA)。

+ leader, replica 中的一个角色， producer 和 consumer 只跟 leader 交互。

+ follower, replica 中的一个角色，从 leader 中复制数据。

+ controller, kafka 集群中的其中一个服务器，用来进行 leader election 以及 各种 failover。

+ zookeeper, kafka 通过 zookeeper 来存储集群的 meta 信息。

### zookeeper

首先从zookeeper说起，kafka的集群只有broker，broker里只有partitions，里面就存储着各种消息(log数据)，没有存储**meta data**，如属于哪个topic，leader是谁等，而这些信息，都存放在zookeeper集群中，相当与hdfs的namenode一样。

![](/images/kafka_01.png)

在zookeeper集群中，kafka创建了许多znode，其中包括comsumer_group的offest，每个broker的ip和端口，每个partition信息等，因此，broker只是淡村的存储数据，而各种协调工作，以及meta data都存在zookeeper中。

### 拓扑结构

![](/images/kafka_02.jpg)

### Producer

一个典型的Kafka集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），他们负责发送消息给kafka集群。Producer采用**Push**的形式推送消息，至于和哪个broker通信，存储在那个partition里，则先要与zookeeper进行协调，由它来告知。

### Topic & Partition

Topic在逻辑上可以被认为是一个queue，每条消费都必须指定它的Topic，可以简单理解为必须指明把这条消息放进哪个queue里。为了使得Kafka的吞吐率可以线性提高，物理上把Topic分成一个或多个Partition，每个Partition在物理上对应一个文件夹，该文件夹下存储这个Partition的所有消息和索引文件.

### Broker

若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高）。一般一个Broker就是一台服务器，用于存储消息的，partition就在broker里。

### Consumer

Consumer是消息的消费者，多个不同的Consumer构成Consumer Group。consumer想要消费消息，采用主动**pull**的方式，同样的，它也要先于zookeeper进行协调，并且，zookeeper会维护consumer的消费对应的topic的offest，保证不会重复消费相同的消息。

### Zookeeper

首先从zookeeper说起，kafka的集群只有broker，broker里只有partitions，里面就存储着各种消息(log数据)，没有存储**meta data**，如属于哪个topic，leader是谁等，而这些信息，都存放在zookeeper集群中，相当与hdfs的namenode一样。

![](/images/kafka_01.png)

在zookeeper集群中，kafka创建了许多znode，其中包括comsumer_group的offest，每个broker的ip和端口，每个partition信息等，因此，broker只是淡村的存储数据，而各种协调工作，以及meta data都存在zookeeper中。



