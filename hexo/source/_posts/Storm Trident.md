---
title: Storm Trient
date: 2017-09-19
tags: Storm
categories: Storm
---

**trident可以理解为storm批处理的高级抽象,提供了分组、分区、聚合、函数等操作,提供一致性和恰好一次处理的语义**

Trident 中含有对状态化（stateful）的数据源进行读取和写入操作的一级抽象封装工具。这个所谓的状态（state）既可以保存在拓扑内部（保存在内存中并通过 HDFS 来实现备份），也可以存入像 Memcached 或者 Cassandra 这样的外部数据库中。而对于 Trident API 而言，这两种机制并没有任何区别

Trident 使用一种容错性的方式实现对 state 的管理，这样，即使在发生操作失败或者重试的情况下状态的更新操作仍然是幂等的。基于这个机制，**每条消息都可以看作被恰好处理了一次**，然后你就可以很容易地推断出 Trident 拓扑的状态。

State 的更新过程支持多级容错性保证机制。在讨论这一点之前，我们先来看一个例子，这个例子展示了如何实现恰好一次的语义的技术。假如你正在对数据流进行一个计数聚合操作，并打算将计数结果存入数据库中。在这个例子里，你存入数据库的就是一个对应计数结果的值，每次处理新 tuple 的时候就会增加这个值。

考虑到可能存在的处理失败情况，tuple 有可能需要重新处理。这样就给 state 的更新操作带来了一个问题（或者其他的副作用）—— 你无法知道当前的这个 tuple 的更新操作是否已经处理过了。也许你之前没有处理过这个 tuple，那么你现在就需要增加计数结果；也许你之前已经处理过 tuple 了并且成功地增加了计数结果，但是在后续操作过程中 tuple 的处理失败了，并由此引发了 tuple 的重新处理操作，这时你就不能再增加计数结果了；还有可能你之前在使用这个 tuple 更新数据库的时候出错了，也就是说计数值的更新操作并未成功，此时在 tuple 的重新处理过程中你仍然需要更新数据库。

所以说，如果只是向数据库中简单地存入计数值，你确实无法知道 tuple 是否已经被处理过。因此，你需要一些更多的信息来做决定。Trident 提供了一种支持恰好一次处理的语义，如下所述：

通过小数据块（batch）的方式来处理 tuple（可以参考Trident 教程一文）
为每个 batch 提供一个唯一的 id，这个 id 称为 “事务 id”（transaction id，txid）。如果需要对 batch 重新处理，这个 batch 上仍然会赋上相同的 txid。
State 的更新操作是按照 batch 的顺序进行的。也就是说，在 batch 2 完成处理之前，batch 3 的状态更新操作不会进行。
基于这几个基本性质，你的 State 的实现就可以检测到 tuple 的 batch 是否已经被处理过，并根据检测结果选择合适的 state 更新操作。你具体采用的操作取决于你的输入 spout 提供的语义，这个语义对每个 batch 都是有效的。有三类支持容错性的 spout：“非事务型”（non-transactional）、“事务型”（transactional）以及“模糊事务型”（opaque transactional）。接下来我们来分析下每种 spout 类型的容错性语义。

## 事务型 spout（Transactional spouts）

记住一点，Trident 是通过小数据块（batch）的方式来处理 tuple 的，而且每个 batch 都会有一个唯一的 txid。spout 的特性是由他们所提供的容错性保证机制决定的，而且这种机制也会对每个 batch 发生作用。事务型 spout 包含以下特性：

1. 每个 batch 的 txid 永远不会改变。对于某个特定的 txid，batch 在执行重新处理操作时所处理的 tuple 集和它的第一次处理操作完全相同。

2. 不同 batch 中的 tuple 不会出现重复的情况（某个 tuple 只会出现在一个 batch 中，而不会同时出现在多个 batch 中）。

3. 每个 tuple 都会放入一个 batch 中（处理操作不会遗漏任何的 tuple）。
这是一种很容易理解的 spout，其中的数据流会被分解到固定的 batches 中

这是一种很容易理解的 spout，其中的数据流会被分解到固定的 batches 中。Storm-contrib 项目中提供了一种基于 Kafka 的事务型 spout 实现。

看到这里，你可能会有这样的疑问：为什么不在拓扑中完全使用事务型 spout 呢？这个原因很好理解。一方面，有些时候事务型 spout 并不能提供足够可靠的容错性保障，所以不需要使用事务型 spout。比如，TransactionalTridentKafkaSpout的工作方式就是使得带有某个 txid 的 batch 中包含有来自一个 Kafka topic 的所有 partition 的 tuple。一旦一个 batch 被发送出去，在将来无论重新发送这个 batch 多少次，batch 中都会包含有完全相同的 tuple 集，这是由事务型 spout 的语义决定的。现在假设 TransactionalTridentKafkaSpout 发送出的某个 batch 处理失败了，而与此同时，Kafka 的某个节点因为故障下线了。这时你就无法重新处理之前的 batch 了（因为 Kafka 的节点故障，Kafka topic 必然有一部分 partition 无法获取到），这个处理过程也会因此终止。

这就是要有“模糊事务型” spout 的原因了 —— 模糊事务型 spout 支持在数据源节点丢失的情况下仍然可以实现恰好一次的处理语义。我们会在下一节讨论这类 spout。

顺便提一点，如果 Kafka 支持数据复制，那么就可以放心地使用事务型 spout 提供的容错性机制了，因为这种情况下某个节点的故障不会导致数据丢失，不过 Kafka 暂时还不支持该特性。（本文的写作时间应该较早，Kakfa 早就已经可以支持复制的机制了 —— 译者注）。

在讨论“模糊事务型” spout 之前，让我们先来看看如何为事务型 spout 设计一种支持恰好一次语义的 State。这个 State 就称为 “事务型 state”，它支持对于特定的 txid 永远只与同一组 tuple 相关联的特性。

假如你的拓扑需要计算单词数，而且你准备将计数结果存入一个 K-V 型数据库中。这里的 key 就是单词，value 对应于单词数。从上面的讨论中你应该已经明白了仅仅存储计数结果是无法确定某个 batch 中的tuple 是否已经被处理过的。所以，现在你应该将 txid 作为一种原子化的值与计数值一起存入数据库。随后，在更新计数值的时候，你就可以将数据库中的 txid 与当前处理的 batch 的 txid 进行比对。如果两者相同，你就可以跳过更新操作 —— 由于 Trident 的强有序性处理机制，可以确定数据库中的值是对应于当前的 batch 的。如果两者不同，你就可以放心地增加计数值。由于一个 batch 的 txid 永远不会改变，而且 Trident 能够保证 state 的更新操作完全是按照 batch 的顺序进行的，所以，这样的处理逻辑是完全可行的。

下面来看一个例子。假如你正在处理 txid 3，其中包含有以下几个 tuple：
```
["man"]
["man"]
["dog"]
```
假如数据库中有以下几个 key-value 对：
```
man => [count=3, txid=1]
dog => [count=4, txid=3]
apple => [count=10, txid=2]
```
其中与 “man” 相关联的 txid 为 1。由于当前处理的 txid 为 3，你就可以确定当前处理的 batch 与数据库中存储的值无关，这样你就可以放心地将 “man” 的计数值加上 2 并更新 txid 为 3。另一方面，由于 “dog” 的 txid 与当前的 txid 相同，所以，“dog” 的计数是之前已经处理过的，现在不能再对数据库中的计数值进行更新操作。这样，在结束 txid3 的更新操作之后，数据库中的结果就会变成这样：
```
man => [count=5, txid=3]
dog => [count=4, txid=3]
apple => [count=10, txid=2]
```
现在我们再来讨论一下“模糊事务型” spout。