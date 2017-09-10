---
title: TCP三次握手和四次分手
date: 2017-09-03 18:54:25
tags: 网络
categories: 网络
---

## TCP三次握手协议

初始条件，Server处于监听端口状态

1. Client发送同步请求报文(SYN=1)给客户端，client进入同步已发送状态(SYN_SEND)

2. Server接收到请求报文，需要对这个报文进行确认，发送确认报文(SYN+ACK)给client，同时进入同步已接收状态(SYN_RECV)

3. client收到server的同步确认报文后，发送ACK确认报文给server，同时进入同步已建立状态ESTABLISHED，server收到后也会进入ESTABLISHED

为什么要三次握手？

> 为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。浪费服务器的资源

## 四次分手协议

1. 第一次分手：主机1（可以使客户端，也可以是服务器端），设置Sequence Number和Acknowledgment Number，向主机2发送一个FIN报文段；此时，主机1进入FIN_WAIT_1状态；这表示主机1没有数据要发送给主机2了；

2. 第二次分手：主机2收到了主机1发送的FIN报文段，向主机1回一个ACK报文段，Acknowledgment Number为Sequence Number加1；主机1进入FIN_WAIT_2状态；主机2告诉主机1，我“同意”你的关闭请求；

3. 第三次分手：主机2向主机1发送FIN报文段，请求关闭连接，同时主机2进入LAST_ACK状态；

4. 第四次分手：主机1收到主机2发送的FIN报文段，向主机2发送ACK报文段，然后主机1进入TIME_WAIT状态；主机2收到主机1的ACK报文段以后，就关闭连接；此时，主机1等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，主机1也可以关闭连接了。