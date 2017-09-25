---
title: Java Servlet
date: 2017-09-25
tags: Java
categories: Java
---

`Servlet`是一个Java对象，它运行在Web容器中，并且定义了一系列方法接收HTTP请求，处理，返回响应。

## 生命周期

Servlet 生命周期：Servlet加载--->实例化--->服务--->销毁。

1. `init（）`：在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。

> 可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init（）。
2. `service（）`：它是Servlet的核心，负责响应客户的请求。

> 每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。

3. `destroy（）`： 仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。

> 一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

## 工作原理

1. 首先简单解释一下Servlet接收和响应客户请求的过程，首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行了匹配，选择调用doGet,doPost等这些方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet（）、doPost（）等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。

2. 每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到init(),service(),destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。

3. Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse 强转为HttpRequest 和 HttpResponse。

## Tomcat和Servlet

1. Web Client 向Servlet容器（Tomcat）发出Http请求

2. Servlet容器接收Web Client的请求

3. Servlet容器创建一个`HttpRequest`对象，将Web Client请求的信息封装到这个对象中。

4. Servlet容器创建一个`HttpResponse`对象

5. Servlet容器调用`HttpServlet`对象的service方法，把`HttpRequest`对象与`HttpResponse`对象作为参数传给 `HttpServlet` 对象。

6. `HttpServlet`调用`HttpRequest`对象的有关方法，获取Http请求信息。

7. HttpServlet调用`HttpResponse`对象的有关方法，生成响应数据。

8. Servlet容器把`HttpServlet`的响应结果传给Web Client

## 线程安全

Servlet的init()方法是工作在单线程的环境下，开发者不必考虑任何线程安全的问题。

当服务器接收到来自客户端的多个请求时，服务器会在单独的Client Service Thread线程中执行Servlet实例的service()方法服务于每个客户端。此时会有多个线程同时执行同一个Servlet实例的service()方法，因此必须考虑线程安全的问题。
