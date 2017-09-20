---
title: Kafka Spout
date: 2017-09-19
tags: Storm
categories: Storm
---

当Storm需要从Kafka读取数据时，这是Storm就是Kafka的一个消费者，Storm项目中已经集成了Kafka Spout组件，用于直接Kafka中读取某一个topic的数据。

Kafka Spout实现了一个`kafka.javaapi.consumer.SimpleConsumer`的客户端功能，既然是SimpleConsumer，那么表明zk的连接维护，partition的选取，offset的维护都要自己维护，而`KafkaSpout`则提供了这些功能。包括 partition的分配，消费状态的维护（offset）。同时KafkaSpout使用了storm的可靠API，并实现了spout的ack 和 fail机制。

KafkaSpout的大致处理流程如下：

1. 建立zookeeper客户端，在zookeeper zk_root + "/topics/" + _topic + "/partitions" 路径下获取到partition列表 

2．针对每个partition 到路径Zk_root + "/topics/" + _topic + "/partitions"+"/" + partition_id + "/state"下面获取到leader partition 所在的broker id

3．到/broker/ids/broker id 路径下获取broker的host 和 port 信息，并保存到Map中Partition_id –-> learder broker

4. 获取spout的任务个数和当前任务的index，然后再根据partition的个数来分配当前spout 所消费的partition列表

5. 针对所消费的每个broker建立一个SimpleConsumer对象用来从kafka上获取数据

6. 提交当前partition的消费信息到zookeeper上面保存
















