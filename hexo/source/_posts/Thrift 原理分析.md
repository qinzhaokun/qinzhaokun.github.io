---
title: Thrift 原理分析
date: 2017-08-29
tags: 分布式服务框架
categories: 分布式服务框架
---

由浅入深，我们先定义一个基本的接口文件，最最基本的：userService.thrift

```
namespace java com.xxx.userservice
struct User{
  1: i64 id,
  2: string name,
  3: i32 age
}

service UserService{
  User getUserById(1:i64 id)
}
```

## `Tbase`接口

`struct`只支持简单的数据类型，不支持复杂的Java类，这也是为了兼容更多的语言而做出的妥协。由struct生成的对象`User`实现了"Serializable"接口和"TBase"接口。这两个接口都是和序列化，反序列化相关的。`Serializable`是支持Java原生的序列化；而`Tbase`则是支持thrift本身的序列化和反序列化。它的两个核心方法是：`write`和 `read`. 在struct中的定义的时候发现有个序列号，这个就是按顺序来序列化字段，保证顺序序号**不能重复,但是可以不需要从"1"开始**

`Tbase`实现序列化的方式比Java原生序列化方式更适合在网络传输，它不会携带class类型，性能更高，这也是为什么要用序号严格保证顺序

## Client静态类

UserService.Client类是编译器自动生成的，它被看做一个代理类，它实现了被代理类UserService的所有方法，并且，它是负责与server交互的，主要是**在socket连接中发送请求和相应请求**。 以下是Client关键源码：
```
//参见:TServiceClient  
//API方法调用时,发送请求数据流  
protected void sendBase(String methodName, TBase args) throws TException {  
    oprot_.writeMessageBegin(new TMessage(methodName, TMessageType.CALL, ++seqid_));//首先写入"方法名称"和"seqid_"  
    args.write(oprot_);//序列化参数  
    oprot_.writeMessageEnd();  
    oprot_.getTransport().flush();  
}  
  
protected void receiveBase(TBase result, String methodName) throws TException {  
    TMessage msg = iprot_.readMessageBegin();//如果执行有异常  
    if (msg.type == TMessageType.EXCEPTION) {  
      TApplicationException x = TApplicationException.read(iprot_);  
      iprot_.readMessageEnd();  
      throw x;  
    }//检测seqid是否一致  
    if (msg.seqid != seqid_) {  
      throw new TApplicationException(TApplicationException.BAD_SEQUENCE_ID, methodName + " failed: out of sequence response");  
    }  
    result.read(iprot_);//反序列化  
    iprot_.readMessageEnd();  
}  
```
代码比较简单，值得注意的是其维护了seqid来标记一次请求，回复时也会携带该请求。

## Processor类

该类也是UserService的内部类，主要是给服务器端使用的，它是服务器端的代理类. 附上源码：
```
//参考: TBaseProcessor.java  
@Override  
public boolean process(TProtocol in, TProtocol out) throws TException {  
    TMessage msg = in.readMessageBegin();  
    ProcessFunction fn = processMap.get(msg.name);//根据方法名,查找"内部类"  
    if (fn == null) {  
      TProtocolUtil.skip(in, TType.STRUCT);  
      in.readMessageEnd();  
      TApplicationException x = new TApplicationException(TApplicationException.UNKNOWN_METHOD, "Invalid method name: '"+msg.name+"'");  
      out.writeMessageBegin(new TMessage(msg.name, TMessageType.EXCEPTION, msg.seqid));  
      x.write(out);//序列化响应结果,直接输出  
      out.writeMessageEnd();  
      out.getTransport().flush();  
      return true;  
    }  
    fn.process(msg.seqid, in, out, iface);  
    return true;  
} 
```

可以看出，processor读入方法名后，通过保存在一个processMap中的信息查找到真正的方法，可以看出，以方法名作为key值，也表明发thrift不支持重载。

thrift没有使用反射机制去调用对应方法，而是为每个方法生成一个内部类来处理，这样增大了代码量，却减小了性能方面的开销，另一方面用户也可自行修改生成的方法类代码。但是个人认为，这样做仍然是为了实现不同语言的统一，毕竟只有java有反射机制
```
public static class getById<I extends Iface> extends org.apache.thrift.ProcessFunction<I, getById_args> {  
  public getById() {  
    super("getById");//其中getById为标识符  
  }  
  
  public getById_args getEmptyArgsInstance() {  
    return new getById_args();  
  }  
  
  protected boolean isOneway() {  
    return false;  
  }  
  //实际处理方法  
  public getById_result getResult(I iface, getById_args args) throws org.apache.thrift.TException {  
    getById_result result = new getById_result();  
    result.success = iface.getById(args.id);//直接调用实例的具体方法，硬编码  
    return result;  
  }  
}  
```
这段代码的关键就是I iface，这个是实现类的实例，那么它是怎么被传进来的呢？
http://www.cnblogs.com/wxd0108/p/7026522.html
