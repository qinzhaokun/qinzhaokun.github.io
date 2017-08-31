---
title: ZooKeeper原理简介
date: 2017-08-30
tags: zookeeper
categories: zookeeper
---

ZooKeeper是一个分布式应用程序协调服务, 这个概念十分模糊，它主要是给分布式应用程序起协调作用的，例如帮助其他分布式服务实现同步服务，配置维护等。一些大型开源分布式系统会使用zookeeper作为赋值，例如hadoop，kafka，storm等

## 集群

### 角色

zookeeper集群中有三种角色，分别是领导者（leader），追随者（follower）和观察者（observer）。

+ leader，从名字就可以看出，它是负责管理整个集群，它不会接受client的请求，负责进行投票的发起和决议，最终更新状态

+ follower，可以理解为真正的工作者它负责与client交互，接受请求，反馈请求，参与领导者发起的投票

+ observer，它是拓展的角色，可以没有，它能够连接client，将请求发送给leader，但是它不参与投票，这就意味着在leader失效时，它无法成为新的leader，同时，它也会同步leader的状态。

### 状态

运行时的zookeeper总共有四种状态，leading，following，observing，looking：

+ looking， server不知道leader是谁，正在搜寻

+ following， 集群有leader，自己是follower，需要与server同步

+ observing， 与following一致，只是不参与投票竞选leader

+ leading，该集群的leader
## 数据模型

zookeeper的本质是一个高可用的文件系统，用来存储信息的。它的结构相当简单，由znode节点构成，它是数据的容器，它的数据全部存储在内存中。这种树形的znode结构和unix文件系统很像。znode共有四种类型：

1. PERSISTENT 持久化目录节点，创建后一直存在，client断开后也依然存在

2. PERSISTENT_SEQUENTIAL, 持久化顺序节点，client断开后依然存在，只是zook给其顺序编号

3. EPHEMERAL 临时节点，client断开后自动删除

4. EPHEMERAL_SEQUENTIAL， 临时顺序节点

## 工作原理

**原子广播**是zookeeper的工作核心，它保证了各个server之间的同步。使用的协议是**Zab**协议，它有两种模式：*恢复模式*和*广播模式*

### 恢复模式

在服务刚启动或者leader宕机时，进入恢复模式，就是一个选举leader的过程。基本原理是寻找系统当中最大的zxid，每次投票时都会投出zxid最大的那个。这一阶段采用UDP协议，基于paxos算法：

1 .每个Server启动以后都询问其它的Server它要投票给谁，**也会询问自己**。

2. 对于其他server的询问，server每次根据自己的状态都回复自己推荐的领导者（Leader）的id和上一次处理事务的zxid（系统启动时每个server都会推荐自己）

3. 收到所有Server回复以后，就计算出zxid最大的哪个Server，并将这个Server相关信息设置成下一次要投票的Server。

4. 计算这过程中获得票数最多的的sever为获胜者，如果获胜者的票数超过半数，则改server被选为领导者（Leader）。否则，继续这个过程，直到领导者（Leader）被选举出来

server的总数必须是奇数2n+1,存活下来的server的数目不少于n+1

这个阶段不对外提供服务

下面是在网上摘抄的一个例子：

> 1) 服务器1启动,此时只有它一台服务器启动了,它发出去的报没有任何响应,所以它的选举状态一直是LOOKING状态
> 2) 服务器2启动,它与最开始启动的服务器1进行通信,互相交换自己的选举结果,由于两者都没有历史数据,所以id值较大的服务器2胜出,但是由于没有达到超过半数以上的服务器都同意选举它(这个例子中的半数以上是3),所以服务器1,2还是继续保持LOOKING状态.> 
> 3) 服务器3启动,根据前面的理论分析,服务器3成为服务器1,2,3中的老大,而与上面不同的是,此时有三台服务器选举了它,所以它成为了这次选举的leader.
> 4) 服务器4启动,根据前面的分析,理论上服务器4应该是服务器1,2,3,4中最大的,但是由于前面已经有半数以上的服务器选举了服务器3,所以它只能接收当小弟的命了.
> 5) 服务器5启动,同4一样,当小弟.

## 广播模式

它采用经典的两阶段提交，leader提出一个决议，由followers进行投票通过这执行该事务，否则什么都不做



