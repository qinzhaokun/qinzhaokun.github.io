---
title: Session机制
date: 2017-09-25
tags: Java
categories: Java
---

**Session代表服务器与浏览器的一次会话过程，这个过程是连续的，也可以时断时续的**。在Servlet中，session指的是HttpSession类的对象.

HTTP和HTTPS协议都是无状态的，一次请求/一次响应，服务器不知道客户端是谁，可想而知，对于需要权限的数据请求，岂不是每一次HTTP都需要携带登录信息？Session就解决了这个问题


