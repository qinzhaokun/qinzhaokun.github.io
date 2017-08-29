---
title: Thrift基本介绍
date: 2017-08-29
tags: 分布式框架
categories: 分布式框架
---

## 基本概念
Thrift是一个RPC框架，能够帮助用户进行远程方法调用，它由Fackbook开源。它的一个比较大的特点就是跨语言，框架之上能够支持多种语言。它是如何支持多语言的呢？答案是Thrift的IDL（接口定义语言）文件，它是一个自定义的文件类型，主要用来描述**接口函数及数据类型**。但是该文件不能够直接在代码中调用，它会被不同语言的编译器，编译成不同语言的接口文件。

关于工作模式，RPC框架实际就是进程间的通信，它提供了多种不同的工作模式，如：线程池模型、非阻塞模型等

## 环境搭建

略

## IDL文件

### 命名空间 namespace

格式为：`namespace java com.xxx.service` namespace 语言 名字, 类似于java中的package

### service关键字

这个类似于class关键字一样，如：
include "thrift_datatype.thrift"  
```
service TestThriftService  
{  
  
    /**  
    *value 中存放两个字符串拼接之后的字符串  
    */  
    thrift_datatype.ResultStr getStr(1:string srcStr1, 2:string srcStr2),  
      
    thrift_datatype.ResultInt getInt(1:i32 val)  
      
}  
```

### 数据类型

+ string， 字符串类型，注意是全部小写形式；例如：string aString

+ i16, 16位整形类型，例如：i16 aI16Val;

+ i32，32位整形类型，对应C/C++/java中的int类型；例如：       I32  aIntVal

+ i64，64位整形，对应C/C++/java中的long类型；例如：I64 aLongVal

+ byte，8位的字符类型，对应C/C++中的char，java中的byte类型；例如：byte aByteVal

+ bool, 布尔类型，对应C/C++中的bool，java中的boolean类型； 例如：bool aBoolVal

+ double，双精度浮点类型，对应C/C++/java中的double类型；例如：double aDoubleVal

+ void，空类型，对应C/C++/java中的void类型；该类型主要用作函数的返回值，例如：void testVoid()

+ map，map类型，例如，定义一个map对象：map<i32, i32> newmap;

+ set，集合类型，例如，定义set<i32>对象：set<i32> aSet;

+ list，链表类型，例如，定义一个list<i32>对象：list<i32> aList

+ enum， 枚举类型

+ struct，自定义结构体类型

### 函数

函数这部分没有什么特别，但是参数类型前面需要加上序号，从1开始，如：

```
thrift_datatype.ResultStr getStr(1:string srcStr1, 2:string srcStr2),
```
## 生成特定语言的文件

thrift --gen <language> <Thrift filename> 

## 服务器端实现

### 提供服务的实现
```
import org.apache.thrift.TException;  
  
public class TestThriftServiceImpl implements TestThriftService.Iface  
{  
  
    @Override  
    public String getStr(String srcStr1, String srcStr2) throws TException {  
          
        long startTime = System.currentTimeMillis();  
        String res = srcStr1 + srcStr2;   
        long stopTime = System.currentTimeMillis();  
          
        System.out.println("[getStr]time interval: " + (stopTime-startTime));  
        return res;  
    }  
  
    @Override  
    public int getInt(int val) throws TException {  
        long startTime = System.currentTimeMillis();  
        int res = val * 10;   
        long stopTime = System.currentTimeMillis();  
          
        System.out.println("[getInt]time interval: " + (stopTime-startTime));  
        return res;  
    }  
  
}  

```
### 服务器启动

采用TNonblockingServer工作模式

```
public class testMain {  
    private static int m_thriftPort = 12356;  
    private static TestThriftServiceImpl m_myService = new TestThriftServiceImpl();  
    private static TServer m_server = null;  
    private static void createNonblockingServer() throws TTransportException  
    {  
        TProcessor tProcessor = new TestThriftService.Processor<TestThriftService.Iface>(m_myService);  
        TNonblockingServerSocket nioSocket = new TNonblockingServerSocket(m_thriftPort);  
        TNonblockingServer.Args tnbArgs = new TNonblockingServer.Args(nioSocket);  
        tnbArgs.processor(tProcessor);  
        tnbArgs.transportFactory(new TFramedTransport.Factory());  
        tnbArgs.protocolFactory(new TBinaryProtocol.Factory());  
        // 使用非阻塞式IO，服务端和客户端需要指定TFramedTransport数据传输的方式  
        m_server = new TNonblockingServer(tnbArgs);  
    }  
    public static boolean start()  
    {  
        try {  
            createNonblockingServer();  
        } catch (TTransportException e) {  
            System.out.println("start server error!" + e);  
            return false;  
        }  
        System.out.println("service at port: " + m_thriftPort);  
        m_server.serve();  
        return true;  
    }  
    public static void main(String[] args)  
    {  
        if(!start())  
        {  
            System.exit(0);  
        }  
    }  
      
}  
```

在TNonblockingServer模式下我们使用二进制协议：TBinaryProtocol,通信方式采用TFramedTransport，即以帧的方式对数据进行传输

### 客户端

```
 m_transport = new TSocket(THRIFT_HOST, THRIFT_PORT,2000);  
 TProtocol protocol = new TBinaryProtocol(m_transport);  
 TestThriftService.Client testClient = new TestThriftService.Client(protocol);  
  
 try {  
     m_transport.open();  
       
     String res = testClient.getStr("test1", "test2");  
     System.out.println("res = " + res);  
     m_transport.close();  
} catch (TException e){  
        // TODO Auto-generated catch block  
        e.printStackTrace();  
}  
```

客户端和服务器端的通信协议要一致，否则无法进行正常的RPC调用。

## protocol & transport

Thrift支持的传输方式

TSocket- 使用堵塞式I/O进行传输，也是最常见的模式。

TFramedTransport- 使用非阻塞方式，按块的大小，进行传输，类似于Java中的NIO。

TFileTransport- 顾名思义按照文件的方式进程传输，虽然这种方式不提供Java的实现，但是实现起来非常简单。

TMemoryTransport- 使用内存I/O，就好比Java中的ByteArrayOutputStream实现。

TZlibTransport- 使用执行zlib压缩，不提供Java的实现。

Thrift支持的服务模型

TSimpleServer – 单线程服务器端使用标准的堵塞式I/O。常用于测试。

TThreadPoolServer – 多线程服务器端使用标准的堵塞式I/O。

TNonblockingServer – 多线程服务器端使用非堵塞式I/O，并且实现了Java中的NIO通道。（需使用TFramedTransport数据传输方式）
