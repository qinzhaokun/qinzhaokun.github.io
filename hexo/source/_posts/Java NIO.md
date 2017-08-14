---
title: Java NIO
date: 2017-08-14
tags: Java
categories: Java
---

Java的底层通信I/O系统，无论是文件I/O还是网络I/O。这里有两个最基本的概念，分别是BIO(阻塞IO)和NIO（非阻塞IO,又称为NEW IO）。*BIO*是指当某个线程进行I/O操作时，线程会被阻塞，直到数据被读取完毕或者数据被完全写入，在此期间，该线程无法执行任何操作。

## BIO模型
下面就来分析一下BIO的模型：
 ```
 import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class SocketIO implements Runnable{
	private Socket socket;
	SocketIO(Socket socket){
		this.socket = socket;
	}
	@Override
	public void run() {
		System.out.println("new message");
		//build dead loop
		while(!socket.isClosed() && !Thread.currentThread().isInterrupted()){
			//stream
			try {
				BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
				System.out.println(br.readLine());
				BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
		         bw.write("ok\n");
		         bw.flush();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
	}
	
}

public class Main {

	public static void main(String [] args){
		try {
			ExecutorService ex = Executors.newFixedThreadPool(15);
			//绑定8080端口
			ServerSocket serverSocket = new ServerSocket(8080);
			while(!Thread.currentThread().isInterrupted()){
				System.out.println("start");
				Socket socket = serverSocket.accept();
				ex.submit(new SocketIO(socket));
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

 ```
 分析：在主函数`main`中，建立一个死循环，当执行到`Socket socket = serverSocket.accept();`,主线程阻塞，等待新的连接，即等待新的网络I/O。当有新的数据发送到本机的8080端口时，主线程得到一个`socket`，为了不让改连接阻塞其他新的连接，采用线程池，创建新的线程处理该连接。主线程则继续去监听8080端口等待其他新的连接。如果不使用多线程，那么每次服务器只能和单一客户端通信。
 
 新的线程负责进行I/O数据的读取与写入，即`SocketIO`类中的`run()`方法。，此时，**该线程是阻塞**，它会等到所有I/O操作完成后才能继续执行。
 
 BIO之所以叫BIO，是因为它在读取或写入I/O数据时，是阻塞的，一定要等到所有的数据读完或写完，CPU才能去执行其他操作，而我们知道I/O操作，如读取本地文件，是内存和硬盘之间的数据传输，而网络上发来的数据，则是网卡和内存之间数据交互，和CPU无关。但此时CPU阻塞着等待内存和硬盘或网卡之间缓慢的I/O操作，**严重浪费了CPU资源**。
 
 ## NIO模型
 
 单个线程，当调用`read()`和`write()`时，线程是要阻塞的，改线程无法执行任何操作。而NIO，则是使得*一个线程从某个通道发送请求读取数据，直到数据变成可读之前，该线程可以继续做其他事，当数据可用时，才去读取数据*。即加入有100M的数据从网络中发过来，刚刚建立了tcp连接，数据还没发送，此时线程不用阻塞等待100M的数据全部到达内存，而是执行其他操作，当数据100M全部从网络到网卡，在去读取。在NIO的模型中，有3个重要的组件，分别是**缓冲区Buffer**，**通道Channel**和**选择器selector**。
 
 ### 缓冲区 Buffer
 
 Buffer是一个容器对象，它在内存中是连续的，其实实质上可以看成是一个数组。在执行I/O操作式时，读取数据时，程序直接从缓冲区中读；写入数据时，程序把数据写到缓冲区中。回忆之前的BIO。它是**面向流（stream）**，读取数据，一个字节一个字节的从流中读取，写入也是。而NIO可以说是**面向块（Block）**的，这里的块就是Buffer。最常用的缓冲区类型是`ByteBuffer`, 此外，每个基本类型都有一个缓冲区类型。
 
 ### 通道 Channel
 
 上面我们提到BIO是面向流的，而在NIO中，通道可以类比于数据流。缓冲区的数据，可以通过通道传输出去；而外面的数据，则是通过通道存储到缓冲区。Channel 用于在字节缓冲区和位于通道另一侧的实体(通常是一个文件或套接字)之间有效地传输数据。
 
 通道可以是单向的，也可以是双向的，而流只能是单向的（InputStream和OutputStream）。
 
 通道可以是阻塞的，也可以是非阻塞的。非阻塞模式的通道永远
 
 不会让调用的线程休眠。请求的操作要么立即完成,要么返回一个结果表明未进行任何操作。这也是NIO为什么能够实现非阻塞的I/O。
 
 ### 选择器 Selector

选择器充当一个监听者的角色，它提供一个注册的功能，通道可以注册到选择器上，这样，选择器就可以管理这些通道，到检测到通道有数据时，执行后续的操作。它是整个NIO的核心。

#### 选择键（SelectionKey）

选择键封装了特定通道和特定选择器的关系，它用来表示选择器关心某个通道的特定操作。

当有任何读写事件发生在通道时，Selector可以感知到，并且我们能从其中得到SelectionKey，近儿找到事件对应的SelectableChannel，从而得到客户端发送的数据。

1. 向Selector对象注册感兴趣的事件
```
//创建Selector对象
Selector sel = Selector.open();

//创建可选择通道，配置为非阻塞模式
ServerSocketChannel server = ServerSocketChannel.open();
server.configureBlocking(false);

//通道监听某个端口
server.socket().bind(new IntetSocketAddress(8080));

// 向Selector中注册感兴趣的事件,监听ACCEPT事件
server.register(sel,SelectorKey.OP_ACCEPT);
```

2. Sekector开始监听，进入死循环
```
try {   
        while(true) {   
            // 该调用会阻塞，直到至少有一个事件发生  
            selector.select();   
            Set<SelectionKey> keys = selector.selectedKeys();  
            Iterator<SelectionKey> iter = keys.iterator();  
            while (iter.hasNext()) {   
                SelectionKey key = (SelectionKey) iter.next();   
                iter.remove();   
                process(key);   
            }   
        }   
    } catch (IOException e) {   
        e.printStackTrace();  
    }
```

或者用以下方式实现：
```
   //IO线程主循环:
   class IoThread extends Thread{
   public void run(){
   Channel channel;
   while(channel=Selector.select()){//选择就绪的事件和对应的连接
      if(channel.event==accept){
         registerNewChannelHandler(channel);//如果是新连接，则注册一个新的读写处理器
      }
      if(channel.event==write){
         getChannelHandler(channel).channelWritable(channel);//如果可以写，则执行写事件
      }
      if(channel.event==read){
          getChannelHandler(channel).channelReadable(channel);//如果可以读，则执行读事件
      }
    }
   }
```

3. 事件发生，去处理对应事件
```
/* 
 * 根据不同的事件做处理 
 * */  
protected void process(SelectionKey key) throws IOException{  
    // 接收请求  
    if (key.isAcceptable()) {  
        ServerSocketChannel server = (ServerSocketChannel) key.channel();  
        SocketChannel channel = server.accept();  
        channel.configureBlocking(false);  
        channel.register(selector, SelectionKey.OP_READ);  
    }  
    // 读信息  
    else if (key.isReadable()) {  
        SocketChannel channel = (SocketChannel) key.channel();   
        int count = channel.read(buffer);   
        if (count > 0) {   
            buffer.flip();   
            CharBuffer charBuffer = decoder.decode(buffer);   
            name = charBuffer.toString();   
            SelectionKey sKey = channel.register(selector, SelectionKey.OP_WRITE);   
            sKey.attach(name);   
        } else {   
            channel.close();   
        }   
        buffer.clear();   
    }  
    // 写事件  
    else if (key.isWritable()) {  
        SocketChannel channel = (SocketChannel) key.channel();   
        String name = (String) key.attachment();   

        ByteBuffer block = encoder.encode(CharBuffer.wrap("Hello " + name));   
        if(block != null)  
        {  
            channel.write(block);  
        }  
        else  
        {  
            channel.close();  
        }  

     }  
}

```

总结：这是最简单的Reactor模式：注册所有感兴趣的事件处理器，单线程轮询选择就绪事件，执行事件处理器。

