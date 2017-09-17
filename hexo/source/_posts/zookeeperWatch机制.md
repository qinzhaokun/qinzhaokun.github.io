---
title: Zookeeper Watcher机制 和 Session
date: 2017-07-12 18:54:25
tags: Zookeeper
categories: Zookeeper
---

Zookeeper可以用来监视集群的状态，同时提供了集群的高可用性，它可以使用瞬时节点设计一个集群机器状态监测机制。发起对于一个 ZNODE 的监听，当该 ZNODE 被改变后，我们会触发对应的方法进行处理，这类方式可以被用在数据监听、集群状态监听等用途。

## 回调函数

Watcher机制的原理涉及到回调函数，下面是一个简单的回调函数：

```
class Caller{
private Call c;

public void setCall(Call c){
	this.c = c;
}

public void call(){
	this.c.method();
}
}

 interface Call{
	public void methid();
}

class CallImpl implements Call{
public void mehtod(){
	System.out.println("call");
}
}

public class Test{
	public static void main(){
		Caller caller = new Caller();
		caller.setCall(new CallImpl());
		caller.call();
	}
}
```

总觉得这就是装饰器模式嘛。

好的那么来看看Zookeeper里发生了什么？

## Wathcer原理

ZooKeeper的Watcher机制主要包括客户端线程、客户端WatchManager和ZooKeeper服务器三部分。在具体工作流程上，简单地讲，客户端在向 ZooKeeper 服务器注册 Watcher 的同时，会将 Watcher 对象存储在客户端的 WatchManager 中。当 ZooKeeper 服务器端触发 Watcher 事件后，会向客户端发送通知，客户端线程从 WatchManager 中取出对应的 Watcher 对象来执行回调逻辑。

### Watcher 接口

```
public interface Watcher {
abstract public void process(WatchedEvent event);
}
```

任何实现了Watcher接口的类都是一个监听器

### Watcher的运行机制

1. Watch是轻量级的，其实就是本地JVM的Callback，服务器端只是存了是否有设置了Watcher的布尔类型。（源码见：org.apache.zookeeper.server.FinalRequestProcessor）
2. 在服务端，在FinalRequestProcessor处理对应的Znode操作时，会根据客户端传递的watcher变量，添加到对应的ZKDatabase（org.apache.zookeeper.server.ZKDatabase）中进行持久化存储，同时将自己NIOServerCnxn做为一个Watcher callback，监听服务端事件变化
3. Leader通过投票通过了某次Znode变化的请求后，然后通知对应的Follower，Follower根据自己内存中的zkDataBase信息，发送notification信息给zookeeper客户端。
4. Zookeeper客户端接收到notification信息后，找到对应变化path的watcher列表，挨个进行触发回调。

可以设置观察的操作：exists,getChildren,getData
可以触发观察的操作：create,delete,setData

1. 一次性触发器（可以通过定时调用方法 get() 或者 exists()获取触发事件，手动访问触发，）
client在一个节点上设置watch，随后节点内容改变，client将获取事件。当节点内容再次改变，client不会获取这个事件，除非它又执行了一次读操作并设置watch
2. 发送至client，watch事件延迟
watch事件异步发送至观察者。比如说client执行一次写操作，节点数据内容发生变化，操作返回后，而watch事件可能还在发往client的路上。这种情况下，zookeeper提供有序保证：client不会得知数据变化，直到它获取watch事件。网络延迟或其他因素可能导致不同client在不同时刻获取watch事件和操作返回值。

## Session连接

Zookeeper客户端和服务端维持一个长连接，每隔10s向服务端发送一个心跳，服务端返回客户端一个响应。这就是一个Session连接，拥有全局唯一的session id。Session连接通常是一直有效，如果因为网络原因断开了连接，客户端会使用相同的session id进行重连。由于服务端保留了session的各种状态，尤其是各种瞬时节点是否删除依赖于session是否失效。

## Session失效

通常客户端主动关闭连接认为是一次session失效。另外也有可能因为其它未知原因，例如网络超时导致的session失效问题。在服务端看来，无法区分session失效是何种情况，一次一旦发生session失效，一定时间后就会将session持有的所有watcher以及瞬时节点删除。

## Session状态

+ CONNECTING：连接中
+ CONNECTED:已连接
+ RECONNECTING:重新连接中
+ RECONNECTED:已经重连
+ CLOSE:关闭



















