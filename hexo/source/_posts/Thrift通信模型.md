---
title: Thrift 通信模型
date: 2017-09-12 18:54:25
tags: 分布式服务框架
categories: 分布式服务框架
---

Thrift采用的是C/S的模式，IDL文件生成了服务器端的代码和客户端的代码，其中`Tprocessor`是服务器端用来处理请求的，`Client`是给客户端发送请求的。Tbase接口定义了数据如何方法和方法参数如何被序列化，而更底层的protocol协议层定义了数据的传输格式，如二进制格式，和transport传输层，定义了传输方式，如TCP/IP方式，共享内存等

![](/images/thrift_01.png)

这是thrift通信的协议栈。

+ 底层IO模块，负责实际的数据传输，包括Socket，文件，或者压缩数据流等。

+ TTransport负责以字节流方式发送和接收Message，是底层IO模块在Thrift框架中的实现，每一个底层IO模块都会有一个对应TTransport来负责Thrift的字节流(Byte Stream)数据在该IO模块上的传输。例如TSocket对应Socket传输，TFileTransport对应文件传输。

+ TProtocol主要负责结构化数据组装成Message，或者从Message结构中读出结构化数据。TProtocol将一个有类型的数据转化为字节流以交给TTransport进行传输，或者从TTransport中读取一定长度的字节数据转化为特定类型的数据。如int32会被TBinaryProtocol Encode为一个四字节的字节数据，或者TBinaryProtocol从TTransport中取出四个字节的数据Decode为int32。

+ TServer负责接收Client的请求，并将请求转发到Processor进行处理。TServer主要任务就是高效的接受Client的请求，特别是在高并发请求的情况下快速完成请求。

+ Processor(或者TProcessor)负责对Client的请求做出相应，包括RPC请求转发，调用参数解析和用户逻辑调用，返回值写回等处理步骤。Processor是服务器端从Thrift框架转入用户逻辑的关键流程。Processor同时也负责向Message结构中写入数据或者读出数据。

## TTransport

它描述了Thrift底层使用什么样的传输协议，一般都是TCP/IP协议，而在应用层上，它可以以使用`Socket`,`File`,`Zip`或`HTTP`.数据是按字节流(ByteStream)方式处理的，即传输层看到的是一个又一个的字节，并把这些字节按照顺序发送和接收。

它具有以下几种模式：
> TSocket：使用阻塞的TCP Socket进行数据传输，也是最常见的模式
> THttpTransport：采用Http传输协议进行数据传输
> TFileTransport：文件（日志）传输类，允许client将文件传给server，允许server将收到的数据写到文件中
> TZlibTransport：与其他的TTransport配合使用，压缩后对数据进行传输，或者将收到的数据解压

下面几个类主要是对上面几个类地装饰（采用了装饰模式），以提高传输效率。
> TBufferedTransport：对某个Transport对象操作的数据进行buffer，即从buffer中读取数据进行传输，或者将数据直接写入buffer
> TFramedTransport：同TBufferedTransport类似，也会对相关数据进行buffer，同时，它支持定长数据发送和接收（按块的大小，进行传输）。
> TMemoryBuffer：从一个缓冲区中读写数据

## TProtocol

TProtocol的主要任务是**把TTransport中的字节流转化为数据流(Data Stream)**,在TProtocol这一层就会出现具有数据类型的数据，如整型，浮点数，字符串，结构体等。TProtocol中数据虽然有了数据类型，但是TProtocol只会按照指定类型将数据读出和写入，而对于数据的真正用途，需要在Thrift自动生成的Server和Client中里处理。

Thrift 可以让用户选择客户端与服务端之间传输通信协议的类别，在传输协议上总体划分为**文本 (text)** 和**二进制 (binary)** 传输协议，为节约带宽，提高传输效率，一般情况下使用二进制类型的传输协议为多数。常用协议有以下几种:

> TBinaryProtocol: 二进制格式
> TCompactProtocol: 高效率的、密集的二进制编码格式
> TJSONProtocol: 使用 JSON 的数据编码协议进行数据传输
> TSimpleJSONProtocol: 提供JSON只写协议, 生成的文件很容易通过脚本语言解析。
> TDebugProtocol: 使用易懂的可读的文本格式，以便于debug

TCompactProtocol 高效的编码方式，使用了类似于ProtocolBuffer的Variable-Length Quantity (VLQ) 编码方式，主要思路是对整数采用可变长度，同时尽量利用没有使用Bit。对于一个int32并不保证一定是4个字节编码，实际中可能是1个字节，也可能是5个字节，但最多是五个字节。TCompactProtocol并不保证一定是最优的，但多数情况下都会比TBinaryProtocol性能要更好。

## TServer

TServer主要作用是接收Client的请求，并转到某个TProcessor上进行请求处理。针对不同的访问规模,Thrift提供了不同的TServer模型。Thrift目前支持的Server模型包括：

> TSimpleServer：使用阻塞IO的单线程服务器，主要用于调试
>TThreadedServer：使用阻塞IO的多线程服务器。每一个请求都在一个线程里处理，并发访问情况下会有很多线程同时在运行。
> TThreadPoolServer：使用阻塞IO的多线程服务器，使用线程池管理处理线程。
> TNonBlockingServer：使用非阻塞IO的多线程服务器，使用少量线程既可以完成大并发量的请求响应，必须使用TFramedTransport。

处理大量更新的操作，主要是在`TThreadedServer`和`TNonblockingServer`中进行选择。`TNonblockingServer`能够使用少量线程处理大量并发连接，但是延迟较高；`TThreadedServer`的延迟较低。实际中，`TThreadedServer`的吞吐量可能会比`TNonblockingServer`高，但是`TThreadedServer`的CPU占用要比`TNonblockingServer`高很多。

## Processor

Processor是由Thrift生成的TProcessor的子类，主要对TServer中一次请求的 InputProtocol和OutputTProtocol进行操作，也就是从InputProtocol中读出Client的请求数据，向OutputProtcol中写入用户逻辑的返回值。Processor是TServer从Thrift框架转到用户逻辑的关键流程。同时TProcessor.process是一个非常关键的处理函数，因为Client所有的RPC调用都会经过该函数处理并转发。

具体的调用流程有如下三步：

1. TServer接收到RPC请求之后，调用TProcessor.process进行处理

2. TProcessor.process首先调用TTransport.readMessageBegin接口，读出RPC调用的名称和RPC调用类型。如果RPC调用类型是RPC Call，则调用TProcessor.process_fn继续处理，对于未知的RPC调用类型，则抛出异常。

3. TProcessor.process_fn根据RPC调用名称到自己的`processMap`中查找对应的RPC处理函数。如果存在对应的RPC处理函数，则调用该处理函数继续进行请求响应。不存在则抛出异常。

http://blog.csdn.net/chen7253886/article/details/52779471



