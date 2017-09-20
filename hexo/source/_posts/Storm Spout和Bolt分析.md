---
title: Storm Spout和Bolt分析
date: 2017-09-19
tags: Storm
categories: Storm
---

Storm中，Spout和Bolt都是其Component，所以，Storm定义了一个名叫IComponent的总接口。无论是下面的ISpout还是IBolt，都实现了Serializable，表示他们能够被序列化，代码会被发送到`Supervisior`机器上。

![](images/storm_06.png)

以上是总图谱，绿色部分是我们最常用、比较简单的部分。红色部分是与事务相关的

## Spout

![](images/storm_07.png)

这是Spout的全家图谱，由于可知，Spout的最顶层抽象是`ISpout`接口,它定义了若干个接口方法：

+ open方法是初始化动作。允许你在该spout初始化时做一些动作，传入了上下文，方便取上下文的一些数据。

+ close方法在该spout关闭前执行，但是并不能得到保证其一定被执行。spout是作为task运行在worker内，在cluster模式下，supervisor会直接kill -9 woker的进程，这样它就无法执行了。而在本地模式下，只要不是kill -9, 如果是发送停止命令，是可以保证close的执行的。

+ activate和deactivate ：一个spout可以被暂时激活和关闭，这两个方法分别在对应的时刻被调用。

+ nextTuple 用来发射数据。**nextTuple**的调用是非阻塞的。

+ ack(Object)传入的Object其实是一个id，唯一表示一个tuple。该方法是这个id所对应的tuple被成功处理后执行。

+fail(Object)同ack，只不过是tuple处理失败时执行。

`BaseRichSpout`做了一些封装，继承自它的类不用实现close、activate、deactivate、ack、fail和getComponentConfiguration方法，只关心最基本核心的部分(nextTuple)。

`IRichSpout`则需用实现更多的接口方法。同时他的自己有更大，例如决定何时指定ack，或ack函数中执行那些额外的操作。

总的来说，`IRichSpout`是集合了顶层接口`ISpout`和`IComponent`的接口，由于是接口它定义的所有方法都没有实现，需要实现了该接口的类去实现；而`BaseRichSpout`实现了`IRichSpout`，并且实现了部分方法，爸核心的方法留出来给继承它的类。凡是有`Base`的组件，都是已经自动实现了ack,fail等函数的组件。

## Bolt

![](images/storm_08.png)

IBolt是顶层接口，定义了三个方法：

+ prepare(Map, TopologyContext, OutputCollector)

+ execute(Tuple)

+ cleanup()

IBolt继承了java.io.Serializable，我们在nimbus上提交了topology以后，创建出来的bolt会序列化后发送到具体执行的worker上去。worker在执行该Bolt时，会先调用prepare方法传入当前执行的上下文。

execute接受一个tuple进行处理，并用prepare方法传入的OutputCollector的ack方法（表示成功）或fail（表示失败）来反馈处理结果。

cleanup 同ISpout的close方法，在关闭前调用。同样不保证其一定执行。

在execute方法中，如果有需要，需要使用OutputCollector的ack/fail来表示处理的结果，不然，可能导致超时，从而Spout从发数据。那么，如果想自动反馈结果呢？Storm提供了IBasicBolt接口，其目的就是实现该接口的Bolt不用在代码中提供反馈结果了，Storm内部会自动反馈成功。如果确实需要反馈失败，可以抛出FailedException。

值得注意的是，`IBasicBolt`并没有实现`IBolt`

由于可以看出，写一个Bolt，可以有两条路线：

+ 实现`IRichBolt`接口或继承`BaseRichBolt`，这种形式，你需要手动在execute来调用ack/fail

+ 实现IBasicBolt接口或继承BaseBasicBolt，它实际上相当于自动做掉了prepare方法和collector.emit.ack(inputTuple)

两种Bolt的使用场景：

使用RichBolt？
 Bolt不是在每次execute()时立刻产生新消息，需要异步的发送新消息(比如聚合一段时间的数据再发送)时，又或者想异步的ack/fail原消息时就需要。

BasicBolt的prepare()里并没有collector参数，只在每次execute()时传入collector。而RichBolt刚好相反，你可以在初始化时就把collector保存起来，用它在任意时候发送消息。

另外，如果用RichBolt的collector，还要考虑在发送消息时是否带上传入的Tuple，如果不带，则下游的处理节点出错也不会回溯到Spout重发。用BasicBolt则已默认带上。
















