---
title: Session机制
date: 2017-09-25
tags: Java
categories: Java
---

**Session代表服务器与浏览器的一次会话过程，这个过程是连续的，也可以时断时续的**。在Servlet中，session指的是HttpSession类的对象.

HTTP和HTTPS协议都是无状态的，一次请求/一次响应，服务器不知道客户端是谁，可想而知，对于需要权限的数据请求，岂不是每一次HTTP都需要携带登录信息？Session就解决了这个问题。

## 生成Session id

 Session ID 是通过一系列算法生成的一个唯一字符串，一般都不会重复。当有一个新的请求来的时候，**一般**都会为该请求创建session，并分配session id。
 
 ## Session 存储结构
 
这个新生成的 Session ID 用于标识这次发起请求的用户，并存储到服务器的某个区域中（默认情况下会在内存中）。 这个 Session ID 同样也是本地存储的一个 key，比如我们上面代码中设置了 name 属性，就相当于在这个 key 中设置了对应的属性。 这样说起来可能有点抽象，举个例子 Session 在服务端的存储结构大概相当于这样：

```
{
    "session-id-1": {
        "name" : "xxx"
    },
    "session-id-2": {
        "name" : "xxx"
    }
    //...
}
}
```
服务端会保存这样一个字典， 将每个用户的 Session ID 和对应的属性都记录下来。

## 返回session id给客户端

服务端会在返回给用户的 HTTP 响应消息中带上这个 Session ID，**在Response Headers**中的`set-cookie`里携带这个id。当客户端的cookie被禁用时，如果想下次再传输session id,那么久必须使用把其附加在url上传给服务器端。可以使用：

```
<a href=“<%=response.encodeURL(“maillogin.jsp“)%>“> 
```

## 客户端发送 Session ID

浏览器下一次再请求这个网站时，会把之前保存的 Session ID 再重新发给服务端。 这时候服务端就会用这个 Session ID 和它之前建立的存储结构中进行匹配，如果这个 Session ID 是之前合法创建的，那么就可以从服务端存储中得到用户之前的状态了，比如登录用户名之类的
