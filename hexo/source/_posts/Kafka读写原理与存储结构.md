---
title: Kafka读写原理与存储结构
date: 2017-09-10
tags: Kafka
categories: Kafka
---

之前我们提过Kafka的基本框架，知道Producer采用Push的方式发送消息给集群，Consumer采用Pull的方式从集群中拉取消息。那么，对于写入一条消息，它的基本流程是
