---
title: playframework 相关介绍
date: 2017-09-25
tags: play
categories: play
---

`playframework`这个web框架是我在腾讯实习的时候用的，之前开发web应用，都是使用spring MVC，但是项目组需要使用这个框架，所以也只好用了。但是对于这个框架也不是太熟悉，所以这里列举它的几个特点吧。

## 一站式开发，效率高

一般不用继承其他组件，所有的组件都有，它提供了诸如开发、测试、发布等一系列工具以及插件的支持。

## 动态编译

修改代码时，不用像其他的框架需要重启服务器。通过ClassLoader在源代码修改的时候动态加载类，解决了修改代码需要重启服务器的问题

## 抛弃Servlet技术栈

抛弃Servlet技术栈，基于Netty实现了自己的 请求响应接口（Request/Result），实现了自己的一套HTTP Server。

## 无状态（stateless）

Play! 框架抛弃了 Servlet/JSP 里 Session 等概念，在每次 HTTP Request 之间不会在 Server 端存储状态，所需的状态都需要在 HTTP Request 之间传递，这样做的好处就是使得服务器的水平扩展性增强了。比如，当系统流量过大时，我们只需要新增一个节点就可以立即增加系统的负载能力。

## 异步非阻塞

由于 Play! 的 HTTP Server 是基于 Netty 实现的， 而 Netty 具有异步高性能、高可靠性和高成熟度的优点，而且 Play! 的默认配置已经为 controller 做了优化，所以本质上， Play! 从里到外都是异步的。 Play! 能够以异步，非阻塞方式处理每一个请求。另外 Play! 的最新版本 Play 2.6.x ，其 HTTP Server 是基于 Akka HTTP 实现的，也能很好地支持异步非阻塞。 Play! 通过放弃传统 Java Web 框架中的 Servlet 而采用自己实现的 HTTP Server，使得它在开发高性能 Web 应用时具有很大的优势。

## 封装ORM

框架封装了JPA,底层实现采用hibernate

## 劣势

+ 要使用`Scala`,语言入门要求高

+ 中文资料少

+ 在国内用的人比较少
