---
title: Kafka读写原理与存储结构
date: 2017-09-10
tags: Kafka
categories: Kafka
---

之前我们提过Kafka的基本框架，知道Producer采用Push的方式发送消息给集群，Consumer采用Pull的方式从集群中拉取消息。那么，对于写入一条消息，它的基本流程是怎样的呢？下面，我们讲沿着：Producer发布消息 -> 消息存储格式 -> Consumer消费消息这三个步骤展开。

## Producer发布消息

我们来回顾一下，要发送一条消息，首先一定要指定一个topic，表明他是那一种类的消息，然后一条消息要发送到一个broker上的某一个partition中，由于需要支持HA，所以对这条消息进行持久化的时候，肯定要同时写入多个partition中，成功之后在回ack给client。

每条消息都被 append 到 patition 中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障 kafka 吞吐率）。

整体的流程图如下：

![](/images/kafka_03.png)

写入消息分为下面几个步骤：

### 确定partition

这是第一步，由于一个topic对应这多个partitions，首先要确定和哪个partition进行通信并存储，发送到broker时，**分区算法**选择将其存储到哪一个 partition. 他遵循以下规则：

1. 指定了 patition，则直接使用；
2. 未指定 patition 但指定 key，通过对 key 的 value 进行hash 选出一个 patition
3. patition 和 key 都未指定，使用轮询选出一个 patition。

### 找到partition的通信地址

由于系统实现了HA，因此，一个partition有多个replica，Producer 先从 `Zookeeper` 的 "/brokers/.../state" 节点找到该 partition 的 **leader**, 之后就只与该leader通信。

### 数据传输

1. producer 将消息发送给该 leader
2. leader 将消息写入本地 log
3. followers 从 leader pull 消息，写入本地 log 后 leader 发送 ACK
4. leader 收到所有 ISR 中的 replica 的 ACK 后，增加 HW（high watermark，最后 commit 的 offset） 并向 producer 发送 ACK

这里我们一定想到了各种意外的情况，比如说网络中断，某个partition宕机了等，因此，**保证消息的传输**有几种方式：

1. At most once 消息可能会丢，但绝不会重复传输
2. At least one 消息绝不会丢，但可能会重复传输
3. Exactly once 每条消息肯定会被传输一次且仅传输一次

## 消息存储格式

一个topic有一个partition？和broker数一样？ 这配置文件里会进行设置： server.properties 中的 num.partitions=3 
一条消息的格式如下：
```
消息长度:     4 bytes (value: 1+4+n)   
版本号:       1 byte  
CRC校验码:    4 bytes  
具体的消息:   n bytes  
```
一个topic的一个partition，物理上对应着一个文件夹，里面存储这所有该partition的消息。没条消息都可以对应一个64位的offest，offset标注了这条消息在发送到这个分区的消息流中的起始位置，每个日志文件的名称都是这个文件第一条日志的offset.所以第一个日志文件的名字就是00000000000.kafka.所以每相邻的两个文件名字的差就是一个数字S,S差不多就是配置文件中指定的日志文件的最大容量。

![](/images/kafka_04.png)

文件夹下有一个索引文件，和若干个.kafka文件。

## Consumer订阅消息

消费者要比生产者复杂得多。由于kafka不会为消费者维护offset，因此，要么自己（Consumer）维护offset，要么交给Zookeeper维护。此外还有Consumer Group的概念，一个Consumer Group有多个Consumer，一条消息只会被其中的一个Consumer消费。一般的，Consumer的数量不要大于Partition的数量，这是因为一个Partition只能被一个Consumer消费，Consumer多了就会有的轮空

对于Consumer，kafka提供了两种API对其进行消费，分别是High Level API 和 Low Level API。前者是Kafka消费数据的高层抽象，消费者客户端代码不需要管理offset的提交，并且采用了消费组的自动负载均衡功能，确保消费者的增减不会影响消息的消费；而后者通常针对特殊的消费逻辑（比如客消费者只想要消费某些特定的Partition），低级API的客户端代码需要自己实现一些和Kafka服务端相关的底层逻辑，比如选择Partition的Leader，处理Leader的故障转移等。
